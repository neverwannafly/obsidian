[[NAT]] [[ICE]]
- It stands for Session Traversal for NAT.
- It is helpful for discovering the publicly reachable IP address of a client.
- Since, it's just a simple UDP acknowledgement, there are many free ICE servers available on the internet.

# Request Lifecycle of STUN
## Initial Conditions
Initially the client is behind a NAT:
```
Internal client address:  10.0.0.5:52528
NAT public address     :  203.0.113.77 (unknown to client)
STUN server address    :  
```
The client doesnt know its public IP:Port

## Client Creates an ICE Candidate and sennds STUN binding request
Thr browser's ICE agent sends a STUN binding request over UDP to the STUN server.
Outgoing packet looks like this
```
Source IP:Port = 10.0.0.5:52528
Dest   IP:Port = 34.120.10.2:3478 (ice candidate)

Payload = STUN Binding Request
  - Message Type: 0x0001 (Binding Request)
  - Transaction ID: random 96-bit value
  - Attributes:
      SOFTWARE
      ICE-CONTROLLING/ICE-CONTROLLED (optional)
      PRIORITY (ICE candidate priority)
      USERNAME (ICE ufrag pair)
      MESSAGE-INTEGRITY (if short/long-term creds apply)
      FINGERPRINT

```

## NAT rewrites source IP/Port
The NAT creates a new UCP mapping entry:
```
Internal -> External:
10.0.0.5:52528  → 203.0.113.77:49123
```

## STUN server receives the request
The STUN server doesnt rtust anything inside the packet payload, it extracts client's public IP and port

## STUN server sends the binding response
The Stun server will send a binding success response back to the source.
```
Message Type: 0x0101 (Binding Success Response)
Transaction ID: <same as request>

Attributes:
  XOR-MAPPED-ADDRESS:
      IP   = 203.0.113.77
      port = 49123
  SOFTWARE
  FINGERPRINT

```
## NAT receives the response and forwards it to internal client (browser)
Since NAT already has the entry of this, it is able to forward the return response from STUN server back to the client that initiated the request. 
The browser ICE agent is able to extract the `XOR-mapped-address` and it creates a server reflexive candidate (srflx)
This candidate is then communicated to the remote peer via the signaling channel
