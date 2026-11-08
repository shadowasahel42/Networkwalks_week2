# Wireshark SOC Analyst Labs

## Overview
This repository documents my completion of a 6-part Wireshark SOC Analyst project. The objective was to step into the role of a SOC analyst, capture live network traffic, and investigate various security incidents using Wireshark. All labs were completed with a 100% score.

## Tools Used
*   **Wireshark:** Primary tool for packet capture and analysis.
*   **Virtual Lab Environment:** Provided by Networkwalks.

## Skills Demonstrated
*   **Display Filters:** (`tcp.port == 5000`, `tcp.flags.syn == 1`, `ftp.request.command == "USER"`)
*   **Stream Analysis:** Reconstructing TCP streams to view application-layer payloads.
*   **Protocol Analysis:** HTTP, FTP, DNS, TCP, SIP, RTP.
*   **Threat Hunting:** Identifying brute force attacks, port scans, and data leaks.

---

## Lab Breakdown & Findings

### 1. Identify the Attacker's IP (Port Scan)
*   **Scenario:** Unusual traffic was reported to the web server.
*   **Analysis:** Filtered for SYN packets (`tcp.flags.syn == 1`). Found a single IP (`192.168.10.66`) sending a rapid block of SYN packets to multiple ports on `192.168.10.20`.
*   **Findings:** The attacker scanned 40 ports. Open ports identified: 22, 80, 443. Closed ports replied with `RST, ACK`.
*   **Conclusion:** This was a **Port Scan**.

### 2. Follow TCP Stream (Part 1 & 2)
*   **Scenario:** Suspicious communications via a legacy chat tool and a hidden note on a web page.
*   **Analysis:** Used `tcp.port == 5000` to find the chat on stream 5. Followed the TCP stream to read the cleartext conversation. For Part 2, analyzed HTTP GET requests and found a hidden note in an HTML comment (`<!-- -->`) on `/staff.html`.
*   **Findings:** Exposed a temporary password (`Tiger#4821`), guest Wi-Fi password (`BlueHarbor77`), and a hidden access code (`Falcon-9921`) pointing to a backup server (`192.168.10.77`).
*   **Conclusion:** Unencrypted chat tools and HTML comments are prime targets for reconnaissance.

### 3. HTTP Plain Text Credentials
*   **Scenario:** A staff member signed into an old portal.
*   **Analysis:** Filtered `http.request.method == "POST"`. Followed the HTTP stream and examined the "HTML Form URL Encoded" data.
*   **Findings:** Found two login attempts. The second was successful with credentials: `username=farhan.k` and `password=Mango#2026`.
*   **Conclusion:** The use of HTTP (instead of HTTPS) exposed sensitive credentials to any network sniffer.

### 4. Catch the Brute Force Attacker
*   **Scenario:** Numerous failed FTP logins reported by the server owner.
*   **Analysis:** Filtered for `ftp`. Identified the server (`192.168.10.20`) and the attacker (`192.168.10.66`). Filtered for `ftp.request.command == "USER"` and `ftp.request.command == "PASS"`.
*   **Findings:** The attacker made 24 failed attempts (Code 530) before succeeding (Code 230). Found the successful password: `Sunshine99`.
*   **Conclusion:** A classic **Brute Force** attack exploiting a weak password.

### 5. VoIP Call Replay
*   **Scenario:** A phone call was intercepted on the network.
*   **Analysis:** Used `Telephony > VoIP Calls` to reconstruct the SIP (Session Initiation Protocol) and RTP (Real-time Transport Protocol) streams. Played back the audio.
*   **Findings:** Identified the caller ("Ali Raza"), the receiver ("Sara"), and extracted sensitive data spoken during the call (Credit Card ending in `2394`, Expiry `09/28`).
*   **Conclusion:** VoIP traffic (SIP/RTP) must be encrypted using SRTP and TLS to prevent eavesdropping.

---

## Key Takeaways
1.  **Encryption is Mandatory:** Never use plain-text protocols (HTTP, FTP, Telnet) for sensitive data.
2.  **Traffic Analysis is Powerful:** Even encrypted traffic can reveal metadata (IPs, ports, packet sizes) that indicate malicious activity.
3.  **Wireshark Proficiency:** Mastering display filters and stream following is essential for fast incident response.
