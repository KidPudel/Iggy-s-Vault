File is a collection of data stored on a persistent medium like a hard drive or SSD.
However in the context of [[OS]] it is more broadly used like "everything is a file".
[[OS]] tries to provide unified interface for accessing various types of resources using the smae basic operations (open, read, write, close)A and represeting thm with [[file descriptor]]
- **Regular Files:** Data on disk.
- **Directories:** Special files that list other files.
- **Device Files:** Representations of hardware devices (like /dev/tty for your terminal, /dev/sda for a disk drive, /dev/null for a data sink). Interacting with these "files" causes the OS to interact with the [[device driver]].
	- Block device read /writes data in fixed-size blocks (like disk sectors), Usually seekable (random access). Think: Hard drives, SSDs, USB drivers
	- Character device: reades/writes data as a stream of characters/bytes. Usually no seakable (sequential). Think: keyboard, mouse, serial port, terminal (TTY)
- **Pipes:** In-memory buffers used to send the output (stdout) of one process to the input (stdin) of another. ([[named pipe]])
- **Sockets:** Endpoints for network communication