[[ICE]] [[NAT]]
- It stands for traversal using relays around NAT
- Most NATs allow inbound requests to pass through to the same internal client for the same inbound connection ip and port. 
- However, symmetric NAT will assign a new port for each destination.
- This means the mapping seen by STUN server cant be seen by other peers.
- TURN is essentially just a server to which you exchange medias to and from. 

# Who uses symmetric NATs?
- Mobile carriers often NAT thoursand of users behind one IP
- Corporate networks