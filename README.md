# ReconGuard – Penetration Workflow Helper

ReconGuard stitches together three battle-tested workflows—Nmap scanning, Burp-style crawling, and Wireshark-style packet capture—into a single CLI you can run on Linux targets. All module outputs are aggregated into a `pen_result` report in the working directory, alongside the raw artifacts (`nmap_<target>.xml`, `capture_<ip>.pcap`).

## Requirements

- Python 3.8+
- `nmap` binary installed and available in `PATH`
- Linux-compatible network interface for packet capture (root or sudo recommended)
- Python dependencies in `requirements.txt`
- (Optional) SecLists installed for large SQLi/XSS wordlists: `sudo apt install seclists`
- (Optional) SecLists password lists (e.g., `Passwords/Common-Credentials/common.txt`) for form bruteforce

Install the Python packages:

```bash
python3 -m pip install -r requirements.txt
```

## Usage

```bash
python3 "recon_guard (3).py" -t example.com
```

Optional flags:

- `--interface <iface>`: Interface for packet capture (`eth0` default)
- `--skip-nmap`: Skip the Nmap phase
- `--skip-spider`: Skip the web spider/vulnerability checks
- `--skip-sniff`: Skip the packet capture
- `--nmap-args "<args>"`: Pass raw arguments directly to Nmap (enables any advanced scan profile)
- `--sqli-wordlist <path>`: Override SQL injection payload list (default: `/usr/share/seclists/Fuzzing/SQL Injection/Generic-SQLi.txt`)
- `--xss-wordlist <path>`: Override reflected XSS payload list (default: `/usr/share/seclists/Fuzzing/XSS/XSS-Common.txt`)
- `--bruteforce-wordlist <path>`: Credentials dictionary for login cracking (default: `/usr/share/seclists/Passwords/Common-Credentials/common.txt`)

During runtime ReconGuard:

1. Runs Nmap.
2. Immediately asks whether you want to enable brute-force, XSS, or SQLi testing and how many payloads/guesses to take from the SecLists dictionaries (first _N_ entries only).
3. Launches the crawler/Burp phase, where the requested attacks are applied only to relevant forms/parameters.

All confirmed payloads/credentials are appended to the `BURP` section of `pen_result` for quick copy/paste into your final report.

## Output

- `pen_result`: Consolidated text report
- `nmap_<target>.xml`: Raw Nmap XML
- `capture_<ip>.pcap`: Packet capture ready for Wireshark

Review these artifacts to continue deeper manual testing or documentation.

