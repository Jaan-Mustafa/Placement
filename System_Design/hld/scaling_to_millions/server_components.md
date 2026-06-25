# What a Server Has

A server is just a computer — optimized for handling many requests.

## Hardware Components

### CPU (Processor)
- Executes all logic, calculations, request handling
- More cores = handles more requests **in parallel**
- Example: Intel Xeon 32-core processor

### RAM (Memory)
- Stores data **currently being used** (active sessions, cache, query results)
- Fast but temporary — cleared on restart
- Example: 128GB RAM — holds millions of user sessions in memory

### Storage (Disk)
- Stores data **permanently** — databases, files, logs, code
- Two types:
  - **HDD** — cheap, slow (used for backups)
  - **SSD** — fast, expensive (used for databases, active files)
- Example: 2TB SSD for the database

### Network Interface Card (NIC)
- Physical hardware that connects the server to the internet/network
- Measured in bandwidth: how much data it can send/receive per second
- Example: 10 Gbps NIC — handles massive traffic

### IP Address
- The server's **address on the network** so clients can reach it
- Example: `142.250.80.46`

## How They Work Together — Real Example

User requests `youtube.com/watch?v=abc`:

1. **NIC** receives the request over the network
2. **CPU** processes it — finds the video metadata
3. **RAM** checks if the video info is cached (fast path)
4. **Storage (SSD)** is hit if not in RAM — reads from DB
5. **NIC** sends the response back to the user

## Quick Reference

| Component | Role | Example |
|-----------|------|---------|
| CPU | Processes logic | 32-core Xeon |
| RAM | Fast temporary storage | 128 GB |
| SSD/HDD | Permanent storage | 2 TB SSD |
| NIC | Network communication | 10 Gbps |
| IP Address | Network identity | `142.x.x.x` |

**TL;DR:** CPU thinks, RAM remembers temporarily, Disk stores permanently, NIC talks to the network.
