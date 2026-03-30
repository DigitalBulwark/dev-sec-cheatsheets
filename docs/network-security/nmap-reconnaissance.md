# Advanced Nmap Reconnaissance Techniques

Network mapping can be noisy, slow, and caught by IDS/IPS quickly. Below are practical techniques used in professional pentesting to bypass firewalls, optimize speed, and evade rudimentary detection.

## 1. Stealth and Evasion Techniques

> [!WARNING]
> Use these flags carefully on sensitive legacy environments, as heavy evasion tools like fragmentation can sometimes crash older stateful firewalls.

### Packet Fragmentation
Bypass simple packet-filtering firewalls by fragmenting IP packets.
```bash
nmap -f -sS <target>
```

### Decoy Scanning
Mask your IP by sending scans from multiple random or specific decoy IPs. This forces the SOC to waste time analyzing logs.
```bash
nmap -D RND:10 <target>
nmap -D 192.168.1.10,192.168.1.11,ME <target>
```

### Source Port Manipulation
Bypass rules that blindly trust traffic originating from specific ports (e.g., DNS `53`, HTTP `80`).
```bash
nmap --source-port 53 -sS <target>
```

## 2. Speed Optimization for /16 and /8 Networks

When scanning large subnets, default Nmap parameters are incredibly slow. Use these parameters to force rapid discovery.

> [!TIP]
> Min-rate forces Nmap to send packets quickly, bypassing dynamic delays. Adjust `2000` to `5000` depending on network bandwidth.

```bash
nmap -p- --min-rate=2000 -T4 -n -Pn <target>
```
- `-n`: Never do DNS resolution (saves time).
- `-Pn`: Treat all hosts as online (skips host discovery ping).
- `-T4`: Aggressive timing template.

## 3. Advanced Nmap Scripting Engine (NSE)

Nmap is not just a port scanner; it’s a vulnerability assessment engine. Instead of a blanket `-sC` (which runs default simple scripts), target your NSE scripts manually.

### Vulnerability Scanning
Run all `vuln` category scripts against open services. This can find unpatched CVEs in SMB, HTTP, or SSL.
```bash
nmap --script vuln <target>
```

### SMB Enumeration
Specifically target Windows environments to pull null sessions, shares, and OS versions without triggering noisy active directory alarms.
```bash
nmap --script smb-enum-shares.nse,smb-os-discovery.nse -p445 <target>
```
