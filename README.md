# TCP SYN Port Scan Detection with Wireshark

Catching a port scan at the packet level, reading TCP flags to separate open ports from closed, and correlating the traffic back to the tool that produced it.

## At a Glance

| Field | Detail |
| --- | --- |
| Alert Type | Network reconnaissance, TCP SYN scan |
| Severity | Medium |
| Detection Method | Wireshark packet capture and flag analysis |
| Tools Used | Wireshark, Nmap, Linux terminal |
| Target | 192.168.1.x, lab host |
| Outcome | Recon confirmed, no exploitation observed |

## What Happened

A host was probed across multiple TCP ports. The source sent SYN packets and never completed the three way handshake, which is the signature of a SYN scan: learn what is listening, leave no session behind.

Reconnaissance is not an incident on its own. It is the step that comes before one. A Tier 1 analyst who can name the scan, list what it found, and confirm nothing followed has closed the ticket properly.

## Traffic Capture

![Wireshark Capture](./screenshots/wireshark_capture.png)

Live traffic was captured on the monitored interface, confirming packet flow between source and target before any filtering was applied.

## SYN Scan Identification

```bash
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

![SYN Scan](./screenshots/syn_scan.png)

This filter isolates connection attempts, SYN set and ACK clear, which is the opening packet of a handshake and nothing else.

The capture showed a high volume of these across different destination ports, probed in sequence, with no handshake ever completing. One SYN is a connection. Hundreds of SYNs and zero sessions is a scan.

## Open Port Response Analysis

![Open Ports](./screenshots/open_ports.png)

The target answered some probes with SYN ACK. A SYN ACK means a service is listening.

Confirmed open on the target:

Port 22, SSH.

Port 80, HTTP.

Port 443, HTTPS.

This is the attacker's shopping list, and reading it from the victim side means the analyst knows exactly what the attacker now knows.

## Closed Port Detection

```bash
tcp.flags.reset == 1
```

![RST Packets](./screenshots/rst_packets.png)

RST responses mark ports that refused the connection. Filtering on the reset flag separates the closed ports from the open ones and confirms the scan was answered honestly by the stack, which is what makes the SYN ACK results trustworthy.

## Packet Level Inspection

![Packet Details](./screenshots/packet_details.png)

TCP header fields were inspected directly rather than trusting the summary view. SYN, ACK, and RST flags were verified per packet, along with source and destination behaviour, confirming the incomplete handshake pattern across the whole capture.

## Attack Correlation

```bash
nmap -sS 192.168.1.x
```

![Nmap Scan](./screenshots/nmap_scan.png)

The scan was run with Nmap in SYN mode and matched against the capture. The tool output and the packet evidence line up, which closes the loop: the traffic in Wireshark is explained, not guessed at.

## Indicators Observed

High volume of SYN packets across multiple destination ports.

No completed TCP three way handshake.

Sequential port probing pattern.

RST responses returned from closed ports.

Traffic pattern consistent with an automated scanning tool.

## MITRE ATT&CK Mapping

| Tactic | Technique ID | Description |
| --- | --- | --- |
| Discovery | T1046 | Network service discovery |

## Analyst Conclusion

Activity confirmed as a TCP SYN port scan.

The source enumerated listening services on the target and identified SSH, HTTP, and HTTPS as open.

No exploitation attempt followed inside the capture window. This is pre attack intelligence gathering, not a breach.

## Recommended Response

Monitor the source IP range for repeat scan activity.

Block the source if the behaviour persists.

Enable IDS alerting on SYN scan patterns so this does not depend on someone watching a packet capture.

Correlate scan activity across network sensors to see whether this host was the only target.

## What This Lab Demonstrates

Capturing and filtering live traffic in Wireshark with purpose, not just recording it.

Reading TCP flags to tell an open port from a closed one at the packet level.

Recognising a scan by handshake behaviour rather than by volume alone.

Correlating attacker tooling with the exact traffic it generates.

Triaging recon activity to a conclusion and mapping it to MITRE ATT&CK.

## Repository Structure

```
.
├── README.md
├── screenshots/
│   ├── wireshark_capture.png
│   ├── syn_scan.png
│   ├── open_ports.png
│   ├── rst_packets.png
│   ├── packet_details.png
│   └── nmap_scan.png
```

---

[![LinkedIn](https://img.shields.io/badge/LinkedIn-WilliamInCyber-blue?style=flat&logo=linkedin)](https://linkedin.com/in/WilliamInCyber)
[![X](https://img.shields.io/badge/X-WilliamInCyber-black?style=flat&logo=x)](https://x.com/WilliamInCyber)
