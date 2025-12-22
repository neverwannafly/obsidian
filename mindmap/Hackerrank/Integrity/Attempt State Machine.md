# AASM
- Attempt state machine uses the aasm gem under the hood. You can define states, events, and assign callbacks on what should happen before a tranmsition event is fired also after the state has been updated in the database. 
- Under the hood, following things happen when you fire an event -
	1. Check if transition is allowed (from correct state)
	2. Run any before callbacks
	3. Change the state
	4. Save to database (if configured)
	5. Run any after callbacks
	6. Return true/false for success
- `Hackerrank` has defined a `enq` method that essentially calls the event that is passed to it in a Resqueue worker. It can be called with some delay or immediately as well. 
# DFA Diagram

![[Attempt State Machine 2025-12-02 12.06.15.excalidraw]]

## Retry State Machine - Detailed Mechanics

### Overview

The retry mechanism is a **sophisticated checkpoint-based system** that allows failed attempts to resume processing from specific recovery points rather than restarting completely.

### Key Constants

```ruby
# Maximum attempts to retry before giving up
MAX_FAILED_ATTEMPT_RETRY = 3

# Maps "in-progress" states to their "checkpoint" recovery states
RETRY_CHECKPOINT_STATE = {
  indexing_for_plagiarism: :enqueued_for_plagiarism_index,
  enqueued_for_plagiarism_calculation: :score_calculated,
  solves_processing: :attempt_cleanup
}

# Maps specific states to specific retry events
RETRY_EVENT = {
  created: :"force_complete!",
  indexed_for_plagiarism: :"retry_verify_proctor_results!"
}

# States that shouldn't trigger retry events (handled by scheduler)
SCHEDULER_PROCESSED_STATES = [:enqueued_for_plagiarism_index, :score_calculated]

# Events that should NOT trigger retries
SKIP_RETRY_EVENTS = [:restart, :fail, :discard]
```

### Retry Flow Diagram

```
Attempt Processing
        ↓
   [Fails]
        ↓
    :failed state
        ↓
  retry_failed_attempt() called
        ↓
   Count failed transitions
        ↓
   if failed_count < MAX_FAILED_ATTEMPT_RETRY (3)
        ↓
   schedule Recruit::ReprocessFailedAttempt worker (10 mins later)
        ↓
   Worker calls process_failed_attempt()
        ↓
   Determine retry checkpoint state
        ↓
   Trigger appropriate retry event
        ↓
   Resume processing from checkpoint
```

### Step-by-Step Retry Process

#### 1. **Error Detection & Transition to Failed State**

When any state transition fails:
```ruby
# handle_error callback triggered (line 625)
def handle_error(e)
  opsgenie_alert(e.message) if raise_alert?(exception: e)
  raise e unless e.is_a?(Recruit::Attempt::AasmError)
end

# If alert conditions met, sends ops alert before attempting retry
def raise_alert?(exception: nil)
  return false if exception.is_a?(::Recruit::Attempt::AsyncRetryError)
  return false if retry_pending?  # Don't alert if retries remaining
  return false if processed?
  true
end
```

The attempt automatically transitions to `:failed` state via the `fail` event.

#### 2. **Retry Eligibility Check**

```ruby
def retry_pending?
  failed_state_count = self.state_logs.where(to_state: ATTEMPT_STATES[:failed]).count
  failed_state_count < MAX_FAILED_ATTEMPT_RETRY - 1
end
```

**Logic:**
- Counts how many times attempt has transitioned to `:failed`
- Returns `true` if count < 2 (allowing up to 3 attempts total: initial + 2 retries)
- Used to determine whether to raise alerts (only alert on final failure)

#### 3. **Retry Scheduling**

When `fail` event occurs, the `retry_failed_attempt` callback executes:

```ruby
def retry_failed_attempt
  failed_state_count = self.state_logs.where(to_state: ATTEMPT_STATES[:failed]).count
  
  if failed_state_count < MAX_FAILED_ATTEMPT_RETRY  # still has retries
    Resque.enqueue_at(10.minutes.from_now, Recruit::ReprocessFailedAttempt, id)
  else  # max retries exhausted
    processing_details = get_report_processing_details
    processing_details.merge!({error: 'Max retries exceeded for failed attempt processing'})
    CustomLogger.log('recruit-attempt-state-machine-errors', processing_details)
  end
end
```

**Key Points:**
- **Retry Delay**: 10 minutes after failure
- **Retry Queue**: Uses Resque for async scheduling
- **Final Failure**: Logs detailed error metrics but doesn't raise alert (prevents alert spam)

#### 4. **Checkpoint Recovery Logic**

When the `Recruit::ReprocessFailedAttempt` worker executes, it calls:

```ruby
def process_failed_attempt
  return if self.aasm_state.to_sym != :failed
  
  last_transition = self.state_logs.last
  return if last_transition.to_state != ATTEMPT_STATES[:failed]
  
  # Get the state where failure occurred (from_state of last failed transition)
  last_active_state = ATTEMPT_STATES_ID_NAME_MAP[last_transition.from_state]
  
  # Determine recovery checkpoint
  retry_state = RETRY_CHECKPOINT_STATE[last_active_state] || last_active_state
  
  # Special case: if failed in indexed_for_plagiarism, keep it failed
  # (this prevents looping on proctoring retry)
  if retry_state == :indexed_for_plagiarism
    self.aasm_state = ATTEMPT_STATES[:failed]
  else
    self.aasm_state = ATTEMPT_STATES[retry_state]
  end
  
  # For scheduler-processed states, just save without triggering events
  if SCHEDULER_PROCESSED_STATES.include?(retry_state)
    return self.save!
  end
  
  # Trigger appropriate retry event
  trigger_retry_event(retry_state)
end
```

**Checkpoint Mapping:**
- **In-Progress State** → **Recovery Checkpoint**
- `indexing_for_plagiarism` → `enqueued_for_plagiarism_index` (restart plagiarism indexing)
- `enqueued_for_plagiarism_calculation` → `score_calculated` (restart plagiarism calc)
- `solves_processing` → `attempt_cleanup` (restart solve processing)
- Any other state → itself (no rollback, just retrigger same state)

**Example Retry Scenarios:**

| Failed in State | Recovery Checkpoint | Behavior |
|---|---|---|
| `indexing_for_plagiarism` | `enqueued_for_plagiarism_index` | Re-enqueue plagiarism indexing worker |
| `solves_processing` | `attempt_cleanup` | Go back to cleanup, then retry solve processing |
| `score_calculated` | `score_calculated` | **No rollback** - scheduler will handle (just save) |
| `proctoring_results_generated` | `proctoring_results_generated` | Retrigger proctoring verification |
| `completed` | `completed` | Retrigger attempt cleanup |
| `created` | `created` | Force complete if candidate is done |

#### 5. **Event Selection & Triggering**

```ruby
def trigger_retry_event(retry_state)
  # Get all permitted events, excluding restart/fail/discard
  permitted_events = self.aasm.events(
    permitted: true,
    reject: SKIP_RETRY_EVENTS
  ).map(&:name)
  
  # Reset solve metadata if going back before solves_processed
  reset_solve_metadata_and_process if ATTEMPT_STATES[retry_state] < ATTEMPT_STATES[:solves_processed]
  
  retry_event = nil
  
  # Check if there's a specifically mapped retry event for this state
  if RETRY_EVENT.key?(retry_state)
    retry_event = RETRY_EVENT[retry_state]
  elsif permitted_events.count == 1
    # If only one permitted event, use it
    retry_event = "#{permitted_events.first}!"
  else
    # Multiple permitted events: default to reprocess_solves for safety
    self.aasm_state = ATTEMPT_STATES[:failed]
    retry_event = 'reprocess_solves!'
  end
  
  # Execute the retry event
  self.send(retry_event)
end
```

**Event Selection Priority:**
1. **Explicit Mapping**: Check `RETRY_EVENT` constant for state-specific events
   - `created` → `force_complete!`
   - `indexed_for_plagiarism` → `retry_verify_proctor_results!`
2. **Single Permitted Event**: If only one event is allowed from current state, use it
3. **Fallback**: Multiple options or no clear path → revert to `:failed` and trigger `reprocess_solves!`

**Solve Metadata Reset:**

For failures before `solves_processed`, the system resets solve processing metadata:

```ruby
def reset_solve_metadata_and_process
  solves.where.not(status: 2).find_each do |solve|
    retry_count = solve.metadata_get('cc_count')
    
    # If retry count >= max allowed reruns, reset it
    if retry_count.to_i >= Recruit::Runstatus::MAX_RERUNS
      solve.metadata_set('cc_count', 0)
      solve.save
    end
    
    # Re-submit to codechecker
    solve.run_status&.submit_to_codechecker
  end
end
```

This ensures code execution limits are reset and solves can be reprocessed.

### Retry State Examples

#### Example 1: Plagiarism Indexing Failure
```
State: indexing_for_plagiarism (indexing in progress)
Error: Service timeout
        ↓
Transition to: failed
        ↓
Retry scheduled (10 mins later)
        ↓
process_failed_attempt() runs
        ↓
last_active_state = :indexing_for_plagiarism
retry_state = RETRY_CHECKPOINT_STATE[:indexing_for_plagiarism] = :enqueued_for_plagiarism_index
        ↓
aasm_state set to :enqueued_for_plagiarism_index
        ↓
trigger_retry_event(:enqueued_for_plagiarism_index)
        ↓
Permitted events from :enqueued_for_plagiarism_index: [:start_index_for_plagiarism]
        ↓
trigger_event: start_index_for_plagiarism!
        ↓
Transition: enqueued_for_plagiarism_index → indexing_for_plagiarism
        ↓
Resume plagiarism indexing from beginning
```

#### Example 2: Solve Processing Failure
```
State: solves_processing (processing candidate code)
Error: Worker crash / timeout
        ↓
Transition to: failed
        ↓
Retry scheduled (10 mins later)
        ↓
process_failed_attempt() runs
        ↓
last_active_state = :solves_processing
retry_state = RETRY_CHECKPOINT_STATE[:solves_processing] = :attempt_cleanup
        ↓
aasm_state set to :attempt_cleanup
        ↓
reset_solve_metadata_and_process() called
  - Resets code execution retry counts
  - Re-submits unprocessed solves to codechecker
        ↓
trigger_retry_event(:attempt_cleanup)
        ↓
Permitted events from :attempt_cleanup: [:start_solve_processing]
        ↓
trigger_event: start_solve_processing!
        ↓
Transition: attempt_cleanup → solves_processing
        ↓
Resume solve processing with fresh metadata
```

#### Example 3: Maximum Retries Exhausted
```
Attempt fails 3 times in same state
        ↓
failed_state_count reaches MAX_FAILED_ATTEMPT_RETRY
        ↓
raise_alert? returns true (no longer retry_pending)
        ↓
Opsgenie alert sent to ops team
        ↓
retry_failed_attempt() checks: failed_state_count < MAX_FAILED_ATTEMPT_RETRY = FALSE
        ↓
No new worker enqueued
        ↓
detailed processing_details logged with error message
        ↓
Attempt stuck in :failed state (requires manual intervention)
```

### Retry Constraints & Limitations

| Constraint | Value | Reason |
|---|---|---|
| Maximum retries | 3 total attempts | Prevent infinite retry loops |
| Retry delay | 10 minutes | Allow services to recover |
| Alerts on | 3rd failure only | Prevent alert spam during retries |
| Solve reset | Before `solves_processed` | Avoid poisoned execution state |
| Max state rollback | One checkpoint back | Prevent cascading failures |

### Error Types & Handling

```ruby
# AsyncRetryError: Transient error, trigger retry without alert
# Typically used for timeout/service errors
raise ::Recruit::Attempt::AsyncRetryError

# AasmError: State machine configuration error, always raise
# Logic errors in transitions

# Other exceptions: Logged and may trigger alert if not retrying
```

---

### Special Events

#### `fail` Event
- **Trigger**: Error in any state transition
- **Transition**: Any state → `failed`
- **After callback**: `retry_failed_attempt` (initiates retry logic)
- **Impact**: Logs error, sends alerts (unless retrying), enqueues reprocessing

#### `restart` Event
- **Transition**: Any state (except `created`, `discarded`) → `created`
- **Purpose**: Restart attempt processing from beginning
- **After callback**: Re-enqueues completion worker

#### `discard` Event
- **Transition**: Any state (except `discarded`) → `discarded`
- **Purpose**: Permanently discard attempt (cannot recover)
- **After callback**: `do_discard` cleanup

#### `reprocess_solves` Event
- **Trigger**: Retry from `failed` state
- **Transition**: `failed` → `solves_processing`
- **Purpose**: Restart solve reprocessing after failure

#### `retry_verify_proctor_results` Event
- **Trigger**: Special retry for proctoring-related failures
- **Transition**: `failed` → `additional_attempt_info_stored`
- **Purpose**: Re-enqueue proctoring analysis after failure

