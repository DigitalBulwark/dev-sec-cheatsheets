# Python Automation Scripts for Reconnaissance

Offensive operators leverage Python to orchestrate repetitive security tasks. Below are common modular chunks to scrape subdomains, evaluate certificates, and identify open ports rapidly.

## 1. Quick Socket Port Scanner

Instead of spawning `nmap` binaries through Python's `subprocess`, you can implement a native threaded socket scanner for extreme agility on restrictive hosts.

```python
import socket
from concurrent.futures import ThreadPoolExecutor

def scan_port(ip, port):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(0.5)
    result = sock.connect_ex((ip, port))
    if result == 0:
        print(f"[+] Port {port} is OPEN on {ip}")
    sock.close()

def multithreaded_scan(target, ports=[80, 443, 22, 3389, 445]):
    with ThreadPoolExecutor(max_workers=50) as executor:
        for port in ports:
            executor.submit(scan_port, target, int(port))

# multithreaded_scan('192.168.1.1', range(1, 1024))
```

## 2. Certificate Transparency (CT) Log Subdomain Enumeration

Brute-forcing subdomains is noisy. Subdomain discovery via CT Logs (crt.sh) is 100% passive, instant, and leaves no logs on the target server.

> [!TIP]
> Use this function to silently map a company's internal dev/staging environments that were accidentally provisioned with an SSL certificate.

```python
import requests
import json
import re

def crtsh_subdomains(domain):
    url = f"https://crt.sh/?q=%25.{domain}&output=json"
    try:
        req = requests.get(url, timeout=10)
        req.raise_for_status()
        
        data = req.json()
        subdomains = set()
        
        for item in data:
            name_value = item['name_value'].lower()
            # Clean up wildcard *. certs and split multi-domain certs
            clean_names = re.split(r'\n', name_value.replace('*.', ''))
            subdomains.update(clean_names)
            
        print(f"[*] Discovered {len(subdomains)} unique passive subdomains:")
        for sub in sorted(subdomains):
            print(f"    - {sub}")
            
    except Exception as e:
        print(f"[-] CT Log error: {str(e)}")

# crtsh_subdomains("example.com")
```

## 3. Asynchronous HTTP Banner Grabber

Grab server headers rapidly using `asyncio` and `aiohttp` to identify WAF signatures, unpatched IIS servers, or debug modes.

```python
import aiohttp
import asyncio

async def fetch_headers(session, url):
    try:
        async with session.get(url, timeout=3, ssl=False) as response:
            server = response.headers.get("Server", "Unknown")
            powered_by = response.headers.get("X-Powered-By", "Unknown")
            print(f"[+] {url} => {server} | {powered_by}")
    except Exception:
        print(f"[-] {url} => Timeout/Error")

async def bulk_scan(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_headers(session, url) for url in urls]
        await asyncio.gather(*tasks)

# asyncio.run(bulk_scan(["http://target1.com", "https://target2.com"]))
```
