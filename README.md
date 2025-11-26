# ReconGuard – Penetration Workflow Helper

ReconGuard stitches together four battle-tested workflows—Dirb content discovery, Nmap scanning, Burp-style crawling/exploitation, and Wireshark-style packet capture—into a single CLI you can run on Linux targets. All module outputs are aggregated into a `pen_result` report in the working directory, alongside the raw artifacts (`dirb_<target>.txt`, `nmap_<target>.xml`, `capture_<ip>.pcap`).

## Requirements

- Python 3.8+
- `nmap` binary installed and available in `PATH`
- `dirb` binary installed and available in `PATH` (use `--skip-dirb` to bypass)
- Linux-compatible network interface for packet capture (root or sudo recommended)
- Python dependencies in `requirements.txt`
- (Optional) SecLists installed for large SQLi/XSS wordlists: `sudo apt install seclists`
- (Optional) SecLists password lists (e.g., `Passwords/Common-Credentials/common.txt`) for form bruteforce
- (Optional) Dirb wordlists (default path: `/usr/share/wordlists/dirb/common.txt`)

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
- `--skip-dirb`: Skip the Dirb discovery pre-phase
- `--nmap-args "<args>"`: Pass raw arguments directly to Nmap (enables any advanced scan profile)
- `--sqli-wordlist <path>`: Override SQL injection payload list (default: `/usr/share/seclists/Fuzzing/SQL Injection/Generic-SQLi.txt`)
- `--xss-wordlist <path>`: Override reflected XSS payload list (default: `/usr/share/seclists/Fuzzing/XSS/XSS-Common.txt`)
- `--bruteforce-wordlist <path>`: Credentials dictionary for login cracking (default: `/usr/share/seclists/Passwords/Common-Credentials/common.txt`)
- `--dirb-wordlist <path>`: Dirb content wordlist (default: `/usr/share/wordlists/dirb/common.txt`)
- `--dirb-args "<args>"`: Additional CLI flags passed straight to Dirb

During runtime ReconGuard:

1. Runs Dirb (unless skipped) and tees discovered URLs back into the crawler.
2. Runs Nmap.
3. Launches the crawler to map forms, parameters, and login fields, including every Dirb hit.
4. After discovery, walks through exploitation stages in order:
   - Ask to run SQL injection tests (only if forms/parameters exist), prompting for how many payloads to pull from SecLists (first _N_ entries). Payloads are submitted to every detected input or parameter, and SQL error signatures trigger findings.
   - Ask to run reflected XSS (only if input fields exist). Payloads are echoed back and tracked when reflections occur.
   - Ask to run credential brute-force (only if login fields were detected). The tool keeps trying username/password combinations until it detects success indicators.
5. Consolidates every finding and successful payload/credential into the `BURP` section of `pen_result`.

## Output

- `pen_result`: Consolidated text report including DIRB/NMAP/BURP/WIRESHARK sections
- `dirb_<target>.txt`: Raw Dirb output (URLs discovered, status codes)
- `nmap_<target>.xml`: Raw Nmap XML
- `capture_<ip>.pcap`: Packet capture ready for Wireshark

Review these artifacts to continue deeper manual testing or documentation.

