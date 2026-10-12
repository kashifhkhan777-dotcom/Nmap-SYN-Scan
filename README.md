# Nmap SYN Scan (TCP Half-Open) Lab

## Overview
Demonstrated a TCP SYN scan (half-open scan) with Nmap. Unlike a full TCP Connect scan, a SYN scan never completes the three-way handshake - it sends a SYN, analyzes the response, and immediately tears down without sending ACK. This makes it faster and stealthier.

## TCP SYN Scan Logic
| Target Response | Port State |
|-----------------|-----------|
| SYN/ACK         | Open      |
| RST             | Closed    |
| No response / ICMP error | Filtered |

Because the connection is never completed, the target's application layer never sees an established session - only the kernel's TCP stack processes the SYN. This is why it's called a "half-open" scan.

## Lab Environment
- Scanner: Kali Linux (10.0.2.15)
- Target: 10.0.2.2 (VirtualBox NAT gateway)
- Tool: Nmap 7.99
- Requires root/sudo (raw socket access)

## Command Used

sudo nmap -sS 10.0.2.2 -oN syn_scan.txt

## Results

## Analysis

### Open Ports
- **135/tcp (MSRPC)** - Windows RPC endpoint mapper
- **445/tcp (SMB)** - Windows file sharing

Both are Windows-specific services proxied through the VirtualBox NAT gateway to the underlying Windows host.

### Filtered Ports (998)
Unlike a typical closed port (which returns RST), these 998 ports returned no response. This means VirtualBox's Slirp NAT stack is silently dropping the SYN packets rather than rejecting them. This is a form of firewall behavior at the network gateway.

### MAC Address
`52:55:0A:00:02:02` - a locally administered MAC address typical of VirtualBox virtual interfaces, which is why no vendor is identified.

## Why SYN Scans Are Preferred
1. **Faster** - no full handshake, no waiting for timeout after ACK
2. **Stealthier** - never appears as a completed connection in application logs
3. **Less bandwidth** - fewer packets per port
4. **Reliable** - works against firewalls that block ICMP but allow TCP

## Comparison of Nmap Scan Types

| Scan Type | Flag | Root Required | Detectability | Speed |
|-----------|------|---------------|---------------|-------|
| SYN (half-open) | -sS | Yes | Low | Fast |
| TCP Connect | -sT | No | High (full logon) | Slower |
| UDP | -sU | Yes | Variable | Very slow |
| FIN/Xmas/Null | -sF/-sX/-sN | Yes | Low (bypasses some FW) | Fast |

## Key Takeaways
1. SYN scan is the default when running Nmap as root on Kali - no need for `-sS` explicitly.
2. Port states tell you as much as open ports do: "filtered" implies a firewall, "closed" implies a reachable but unused port.
3. MAC addresses in the output reveal whether targets are local or routed.
4. A low latency (0.0015s) confirms the target is on the same local segment.
5. In a real engagement, open ports become the priority for follow-up service enumeration.

## Evidence
- syn_scan.txt - Raw Nmap -sS output

## Tools Used
- Nmap 7.99 (-sS flag)
