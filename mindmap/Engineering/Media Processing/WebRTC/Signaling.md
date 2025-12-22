[[Session Description Protocol]]

- Its the process of exchanging connection information between two peers before the actual media connection can start.
- Interestingly, webRTC doesnt define how signaling happens, it's left to the devs to implement it.
	- Apps were already using their own custom protocols before (SIP, XMPP, custom APIs)
- It's used to exchange - 
	1. SDP offer
	2. SDP answer
	3. ICE candidates
- This helps the peers know,
	- what codecs they support
	- how to encrypt media (DTLS/SRTP keys)
	- what IP/ports they can reach on

- It's mostly implemented using websockets or even TCP. 