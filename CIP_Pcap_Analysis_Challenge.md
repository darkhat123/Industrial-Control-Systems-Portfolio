
# Service codes Cheat Sheet
| Hex Code | Service Name                   | Typical Use                                        |
| -------- | ------------------------------ | -------------------------------------------------- |
| 0x0A     | Multiple Service Request (MSR) | Bundle multiple CIP commands in one packet         |
| 0x0E     | Get Attribute Single           | Read a single attribute from an object             |
| 0x10     | Set Attribute Single           | Write a single attribute to an object              |
| 0x4C     | Read Data                      | Read multiple tags or I/O data                     |
| 0x4D     | Write Data                     | Write multiple tags or I/O data                    |
| 0x4E     | Forward Close                  | Close an established real-time connection          |
| 0x54     | Forward Open                   | Establish a real-time I/O or messaging connection  |
| 0x65     | Register Session               | Start an EtherNet/IP session (encapsulation layer) |
| 0x66     | Unregister Session             | End an EtherNet/IP session                         |
| 0x70     | Send Unit Data                 | Real-time I/O data (UDP encapsulation)             |
| 0x71     | Send RR Data                   | Request/response for CIP commands (TCP)            |

⚡ Notes:
- Read vs Write / Control
  - Read-only: 0x0E, 0x4C
  - Write / control: 0x10, 0x4D, 0x4E, 0x54
  - Bundled operations: 0x0A (MSR)
- Session Management
  - 0x65 / 0x66 → required for every TCP session; not actual I/O commands
- Real-time I/O
  - 0x70 / 0x71 → used for cyclic or event-driven I/O; often UDP
 
# Investigating CIP traffic

## Focus on “high-impact” services (writes, Forward Open/Close, MSR)

Service codes above 0x4C are usually write or connection-related operations. We filter for operations above read requests to:

- Reduce noise: Most traffic in PCAPs is repetitive “read tag” requests (0x4C).
- Focus on high-impact actions:
- Writes (0x4D) → someone changing values
- Forward Open/Close (0x54,0x4E)→ someone establishing/ending connections
- Multiple Service Requests (0x0A) may still be caught separately, but most other reads below 0x4C are background chatter.
Investigations / Security: If you’re hunting for unauthorized changes or malicious commands, reads aren’t interesting — writes and session control are.

Command: `tcp.port == 44818 && cip.service > 0x4C`

