# TCP Three-Way Handshake

The connection-establishment procedure TCP uses to synchronize sequence numbers between client and server before data transfer.

## What it does

Three steps:
1. **SYN** — client picks an initial sequence number (ISN) and sends a SYN segment
2. **SYN-ACK** — server picks its own ISN, sends SYN, and acknowledges client's ISN by returning `client_ISN + 1`
3. **ACK** — client acknowledges server's ISN by returning `server_ISN + 1`

After this exchange both sides know each other's sequence numbers and the connection is ESTABLISHED.

Connection teardown uses FIN (graceful) or RST (abrupt).

## Sources

- https://www.rfc-editor.org/rfc/rfc793

## Related

- [[TCP]]
- [[packets]]
- [[ACK]]
- [[SSL]]

## Process

- What sequence number state is established after step 3 that makes reliable delivery possible?
- Where does TLS fit relative to this handshake?
- What does RST indicate compared to FIN, and when does each appear?
- How does SYN flooding exploit this process?
