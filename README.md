# 🔄 Week 06 — Lateral Movement & Network Pivoting

**Intern:** Ali Ahsan | **Roll No:** CSI-B1-427
**Program:** Cyberstar Cybersecurity Red Teaming Internship
**Instructor:** Umar Niaz
**Date:** 12 April 2026
**Targets:** Metasploitable 2 (Machine 1) · Internal Network (Machine 2)

---

## 📌 Overview

This week demonstrated how an attacker moves from a single compromised host to deeper targets inside an internal network — covering internal discovery, tunnel setup, credential dumping, and lateral movement using harvested credentials.

---

## 🧪 Tasks Covered

### Task 01 — Internal Network Discovery
After gaining Meterpreter access to Metasploitable 2 via the vsftpd backdoor, internal network mapping was performed using native OS tools and a Bash loop with Netcat:

```bash
# Bash subnet scanner (1-second timeout per host)
for i in $(seq 1 254); do
  nc -w 1 192.168.x.$i <port> && echo "Host up: 192.168.x.$i"
done
```

### Task 02 — Pivoting & Tunneling
Established a **SOCKS5 proxy tunnel** through Machine 1 to reach Machine 2 (not directly accessible from Kali):

```bash
# SSH Dynamic Port Forwarding
ssh -D 1080 user@machine1-ip

# Configure /etc/proxychains.conf
socks5 127.0.0.1 1080

# Route Nmap through proxy (must use -sT for TCP connect over SOCKS)
proxychains nmap -sT -p 22,80,445 <machine2-ip>
```

### Task 03 — Credential Dumping & Harvesting

**Dump password hashes from /etc/shadow:**
```bash
cat /etc/shadow
```

**Offline hash cracking with John the Ripper:**
```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

**Cracked Credentials:**

| Username | Cracked Password |
|----------|-----------------|
| sys | batman |
| klog | 123456789 |
| root | pending |
| msfadmin | pending |

### Task 04 — Lateral Movement Techniques
- **SSH with harvested credentials** — logged into Metasploitable 2 as different users
- **Pass The Hash** — reused NTLM hashes to authenticate without plaintext passwords
- Simulated how attackers pivot between machines on the same subnet

---

## 🔑 Key Lesson

> Initial access is only the beginning. The real impact comes from lateral movement — reaching domain controllers, databases, and high-value targets. Defenders must **assume breach** and monitor internal traffic, not just the perimeter.

---

## 🛡️ Security Recommendations

| Control | Purpose |
|---------|---------|
| Network Segmentation (VLANs) | Limits free pivoting after initial compromise |
| Credential Hygiene | Prevents password reuse across machines |
| Disable LLMNR / NBT-NS | Removes credential capture vectors |
| MFA on SSH / RDP / WinRM | Blocks access even with stolen credentials |
| SIEM/EDR Monitoring | Detects unusual internal auth and scanning |

---

## 🛠️ Tools Used

`Metasploit` · `SSH` · `Netcat` · `Proxychains` · `John the Ripper` · `Nmap`

---

## ⚠️ Disclaimer

> Performed in an **authorized lab environment** using Metasploitable 2. For educational purposes only.
