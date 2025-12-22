- It stands for transmission control protocol. 
- It's a reliable, stateful connection between two devices. 
- It's key properties essentially includes it being reliable and ordered.
- Its extensiively used for HTTP/s, ssh, email, databases and anything else that needs accuracy.

## TCP Handshake
![[Screenshot 2025-12-08 at 15.40.56.png]]

### **Three-Way Handshake**
1. **Client → Server:** `SYN, seq = x`
2. **Server → Client:** `SYN + ACK, seq = y, ack = x+1`
3. **Client → Server:** `ACK, ack = y+1`

**Purpose of the handshake:**
- Negotiate initial **sequence numbers** (ISN)
- Confirm both parties are reachable
- Set up initial window sizes

## TCP Reliability
TCP guarantees your data arrives correctly and in order.  
It does this by:
- **Sequence numbers**  
    → Every byte is numbered.
- **ACKs**  
    → Receiver says “I got up to byte X”.
- **Retransmission**  
    → If data is not ACKed in time, TCP resends it.
- **Reordering**  
    → If packets arrive out of order, TCP fixes it.
- **Flow control**  
    → Don’t overwhelm the receiver.
- **Congestion control**  
    → Don’t overwhelm the network.

****