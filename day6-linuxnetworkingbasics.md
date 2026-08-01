# Day 6: Linux Networking Basics

## Concepts Covered
- IP addressing, subnetting (CIDR math), gateways
- DNS resolution chain (/etc/hosts → resolver → root → TLD → authoritative)
- Ports and the LISTEN state
- The 4-phase debug model: DNS → TCP → TLS → HTTP

## My Environment (WSL2 Ubuntu 24.04)
- IP: 192.168.39.228
- Subnet: 192.168.32.0/20 (broadcast 192.168.47.255)
- Gateway: 192.168.32.1

## Subnetting Method (host bits → block size → find block)
Example: 172.16.4.50/28
- Host bits = 32 - 28 = 4
- Block size = 2^4 = 16
- Blocks: 0-15, 16-31, 32-47, 48-63...
- 50 falls in 48-63
- Network: 172.16.4.48 | Broadcast: 172.16.4.63 | Usable: .49-.62

## ss -tulpn findings
| Port | Process | Scope |
|---|---|---|
| 22 | sshd | 0.0.0.0 (external) |
| 53 | systemd-resolved | 127.0.0.x (local only) |
| 323 | chrony | 127.0.0.1 (local only) |

Key takeaway: 0.0.0.0 = reachable from anywhere, 127.0.0.x = local only.
This is the first thing to check when "why can't I connect to my app" comes up.

## DNS Break/Fix Exercise
Added `127.0.0.1 google.com` to /etc/hosts:
- curl -v skipped DNS entirely, went straight to 127.0.0.1
- Got "Connection refused" — confirmed via ss -tulpn that nothing listens on port 80
- Removed the line → DNS resolution restored, real Google IPs returned

Key takeaway: /etc/hosts is checked BEFORE DNS. curl's "was resolved" message
doesn't tell you the SOURCE of resolution — a dangerous blind spot in real debugging.
