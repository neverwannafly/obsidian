- It stands for Network Address Translation
- NAT allows multiple devices within a private network to share a single public IP address. 
- When data leaves one's network, the NAT device (usually the router) changes the source address from a device's private address to network's public address, making a note of this translation. 
- When response come back, it uses these notes to route the data to the correct internal device. 
- NAT can map internalIP:port -> publicIp:port, and will have a different port for every outbound request.

When your device sends a packet to the internet, the NAT:
1. **Rewrites** your IP and port
2. **Creates a mapping entry** in its NAT table
3. **Forward the packet to the remote server**
    

For example:
NAT stores a table entry like:
```
Internal: 10.0.0.5:55555 
Public:   203.0.113.77:49123
Remote:   34.120.10.2:3478 
```

## NAT Type Comparison Table (Technical)

| NAT Type                 | Inbound Rules                                                                                                                             | Port Allocation Behavior                                                                                                 | STUN Behavior                                                             | Hole Punching Success                                                  | Security Characteristics                                      | Typical Environments                                                |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------- |
| Full Cone NAT            | Once an internal host creates a mapping, inbound packets from _any_ external IP and _any_ port are accepted on the mapped (public) port   | NAT assigns a single external port per internal port and keeps it stable                                                 | STUN returns a usable server-reflexive address; mapping is predictable    | Very high; mapping is globally reachable                               | Weak isolation; behaves almost like 1:1 NAT                   | Older home routers; uncommon today                                  |
| Restricted Cone NAT      | Inbound allowed only from external IPs that the internal host has previously sent packets to                                              | External port remains stable; one mapping per internal IP:port                                                           | STUN works reliably; returned public address is typically correct for P2P | High; peers usually succeed once both send packets                     | Medium isolation; inbound limited to previously contacted IPs | Home routers; small office networks                                 |
| Port-Restricted Cone NAT | Inbound allowed only from the exact external IP _and_ port that the internal host contacted                                               | External port stable; mapping is per internal IP:port pair                                                               | STUN works; but ICE needs correct remote port knowledge                   | Moderate; requires both endpoints to send to the exact predicted tuple | High isolation; inbound limited to exact source tuples        | Many modern consumer routers                                        |
| Symmetric NAT            | Inbound allowed only from the specific external IP:port pair for which the mapping was created; multiple mappings created per destination | NAT allocates a different external port for each distinct (internal IP:port, remote IP:port) tuple; highly unpredictable | STUN result is not reusable toward peers; mapping differs per target      | Poor; hole punching usually fails without relay                        | Very high isolation; designed for enterprise-grade security   | Corporate networks, mobile carriers, CGNAT deployments, public WiFi |