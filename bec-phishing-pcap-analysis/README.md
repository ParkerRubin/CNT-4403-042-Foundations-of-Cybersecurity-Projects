# Project 1: Catch Me If You Can — BEC Phishing Investigation

Network forensics project simulating a Business Email Compromise (BEC) scenario: a malicious actor sent phishing emails attempting to trick employees into fraudulent payments. The goal was to inspect `.pcap` captures in Wireshark, isolate the malicious traffic from legitimate mail, and trace the phishing campaign back to its source.

## Objective

- Inspect `.pcap` files and extract raw email content from SMTP traffic
- Distinguish legitimate emails from fraudulent ones across multiple captures
- Identify the malicious actor's source IP
- Understand why SMTP header metadata — not just email body content — is the actual forensic anchor for tracing a phishing campaign's origin

## Findings

**Malicious actor's IP address:** `10.6.1.104`

**Phishing subject lines identified** (all from `C.pcap`):
1. "Read carefully! - dayrit"
2. "Pay! - 12345"
3. "Don't wait too long! - fatima"

## Methodology

1. Applied the `smtp` display filter across all four provided `.pcap` files to surface mail traffic.
2. Triaged each capture for suspicious language — payment demands, extortion phrasing, ransomware indicators — and identified `C.pcap` as the capture containing malicious content.
3. Used **Follow → TCP Stream** on flagged packets to read full email exchanges in sequence.
4. Traced the origin of the malicious mail to `10.6.1.104` sending to the mail server.
5. Narrowed the view with `ip.addr == 10.6.1.104` to isolate every packet tied to that host.
6. Confirmed the pattern repeated across multiple emails from the same IP, all following a RAT ("Remote Administration Tool") extortion script demanding payment.

**Note:** `10.6.1.104` is a private (RFC 1918) address. Seeing a private IP as the apparent source of external phishing traffic is a notable detail — it suggests either an internal host was compromised and used to relay the phishing emails, or the capture reflects internal network traffic/IP spoofing rather than a direct external actor.

## Stretch Goal — Extracted Phishing Emails

Used **File → Export Objects → IMF** in Wireshark to pull every email object out of `C.pcap`, then exported and opened three as `.eml` files in a mail client to inspect their full content.

![IMF object export list from C.pcap](images/imf-export-list.png)
*Full list of extracted email objects from `C.pcap`, showing the recurring extortion-style subject line pattern across dozens of messages.*

![Phishing email content - password disclosure and RAT claim](images/eml-peace.png)
*Extracted email opening with a claimed password and RAT (Remote Administration Tool) compromise claim — a common BEC/sextortion scare tactic.*

![Phishing email with Bitcoin payment demand](images/eml-tomcat.png)
*Same extortion template, this time including a specific Bitcoin payment demand and instructions for purchasing BTC.*

![Third phishing email variant](images/eml-incretible.png)
*A third variant of the same template sent to a different recipient, confirming this was a mass, scripted phishing campaign rather than a targeted one-off.*

## Tools Used

- **Wireshark** — packet capture inspection, SMTP filtering, TCP Stream follow, IMF object export
- Display filters: `smtp`, `ip.addr == 10.6.1.104`

## Key Takeaway

BEC phishing works because the emails are designed to look legitimate — the real signal isn't in the email body, it's in the metadata: sending IP, SMTP headers, and routing path. Packet-level inspection exposes exactly what an email client normally hides from the end user, which is why raw SMTP traffic analysis is a foundational skill for tracing phishing infrastructure back to its source.

---
**Parker Rubin**
