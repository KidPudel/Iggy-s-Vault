Both sides synchronize (SYN) and acknowledges (ACK) each other
Steps are: SYN, SYN-ACK, and ACK

- The client chooses an initial sequence number and set it in the first SYN packet ([[packets]])
- The server chooses its own, and sends SYN with ACK acknowledgment of client's sequence, by incrementing it
- The client sends acknowledgment to the server
> This allows to detect missing or out-of-order segments.

ACK is then typically follow for each segment.

The connection will eventually end with RST (reset or tear down the connection) or FIN (gracefully end the connection)