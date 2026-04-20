[[go]] [[synchronization]] [[mutex]]  

Lets consider the following codeblock

```go
func (c *Client) GetToken(ctx context.Context) (string, error) {
	c.tokenMutex.RLock()
	token := c.token
	c.tokenMutex.RUnlock()

	if token != "" {
		return token, nil
	}

	c.fetchMutex.Lock()
	defer c.fetchMutex.Unlock()

	c.tokenMutex.RLock()
	token = c.token
	c.tokenMutex.RUnlock()

	if token != "" {
		return token, nil
	}

	// Fetch the token
	if err := c.fetchToken(ctx); err != nil {
		return "", err
	}

	c.tokenMutex.RLock()
	token = c.token
	c.tokenMutex.RUnlock()

	return token, nil
}

```

The two read locks are intentional and serve different purposes.

- The first read lock is the fast path. It allows callers to return a cached token without acquiring `fetchMutex`, avoiding unnecessary contention when the token is already available.
    
- The second read lock occurs after acquiring `fetchMutex` and is critical for correctness under concurrency. It prevents duplicate fetches when multiple goroutines race to fetch the token.
    

Consider this scenario:

1. The token is empty. Goroutines A and B call `GetToken` concurrently.
    
2. Both perform the first read and observe that the token is empty.
    
3. Goroutine A acquires `fetchMutex` and begins fetching the token.
    
4. Goroutine B blocks on `fetchMutex`.
    
5. Goroutine A finishes fetching and sets `c.token` (under `tokenMutex`).
    
6. Goroutine B acquires `fetchMutex`. Without rechecking the token, it would fetch again unnecessarily. With the second read, it observes that the token is now set and returns it.
    



This pattern is not redundant. While it introduces an extra read, that second read is what enables single-flight behavior and avoids duplicate network calls under contention.

The correctness of this pattern relies on all reads and writes to `c.token` being synchronized via `tokenMutex`.