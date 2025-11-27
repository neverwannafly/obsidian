[[NAT]]

## Limitations of NAT
Applications that need direct P2P access can't do it with NATs. Reason being, NAT was only desigend for supporting outbound connections, so a client while may connect to a router, but a router wont be able to transfer that request to the correct device in the private address.

## ICE (Pathfinder Protocol)
- It stands for Interactice Connectivity Establishment
- it utilizes two protocols, [[STUN]] and [[TURN]], to overcome different types of NAT configurations.

Lets try to understand in depth how ICE works

### Step 1: Candidate Gathering
When a WebRTC application wants to establish a connection, ICE begins by collecting potential connection points called “candidates.” These come in three varieties:

1. **Host Candidates**: These are your device’s own IP addresses and ports. If your device has multiple network interfaces (like both Wi-Fi and Ethernet), each will generate host candidates.
2. **Server Reflexive Candidates**: These are obtained by sending STUN requests to a STUN server. The responses reveal your public IP address and port as seen from the internet.
3. **Relay Candidates**: These are addresses and ports on TURN servers that will relay media if direct connections fail.

This is like gathering a list of all possible ways someone could contact you: your office extension, your public phone number, and the number of a receptionist who can transfer calls to you.

### Step 2: Candidate Exchange
Once candidates are gathered, they need to be shared with the peer. This happens through the signaling mechanism (which is not specified by WebRTC and is implemented separately, often using web sockets or other real-time communication channels).

The candidates can be included in the initial SDP offer/answer exchange, or they can be sent incrementally as they’re discovered - a process known as “trickle ICE.” Trickle ICE is like starting to share contact methods as soon as you have them, rather than waiting until you’ve collected all possibilities.

### Step 3: Connectivity Checks (Verification)

With candidates exchanged, each peer now has a list of its own candidates (local) and the other peer’s candidates (remote). ICE creates pairs from these candidates and begins methodically testing each pair to see if a connection can be established.

These tests involve sending STUN binding requests from each local candidate to each remote candidate. If a response is received, the connection is viable. This process is like trying to call someone back on each number they provided to see which ones actually work.

Interestingly, during these checks, new candidates might be discovered called “peer reflexive candidates.” These emerge when a STUN check creates a new NAT binding that wasn’t discovered during the initial STUN server phase. This often happens with symmetric NATs.

### Step 4: Candidate Selection (Coordination)

As connectivity checks proceed, successful candidate pairs are identified. ICE then needs to select which pair to use for the actual media transmission. One peer (typically the one that initiated the connection) acts as the “controlling” agent and makes this decision.

The selection typically prioritizes direct connections (host to host) over reflexive candidates, which are in turn preferred over relay candidates. This prioritizes the most efficient path with the lowest latency.

### Step 5: Media Flow (Success)

With a candidate pair selected, media can begin flowing between the peers. Depending on the selected pair, the connection might be:

- **Direct**: A peer-to-peer connection where media flows directly between devices (using host or reflexive candidates)
- **Relayed**: Media flows through a TURN server

The type of connection established depends on the network configuration of both peers and represents the best path that could be found through the NAT maze.