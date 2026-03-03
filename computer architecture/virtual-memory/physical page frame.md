contiguous 4KB by default block of physical [[RAM]] space that [[OS]] uses to back a [[virtual memory page]]

> NOTE: in physical it is page frame, in virtual it is page

| **Virtual Page** | **Physical Page Frame** |
| ---------------- | ----------------------- |
| 0x10000000       | 0xCAFEB000              |
| 0x10001000       | 0x81234000              |
| 0x10002000       | 0x44556000              |

| **Term**       | **Address Space**      | **Meaning**                          | **Who Sees It** | **Typical Size** |
| -------------- | ---------------------- | ------------------------------------ | --------------- | ---------------- |
| **Page**       | Virtual address space  | A fixed-size block of virtual memory | Seen by process | 4 KB             |
| **Page Frame** | Physical address space | A fixed-size block of physical RAM   | Managed by OS   | 4 KB             |