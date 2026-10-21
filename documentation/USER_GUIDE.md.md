# MACsecForge Pro \- User Guide & DevSecOps Manual

## 1\. Quickstart: Interactive Web Studio

1. Open `source-code/index.html` in any modern web browser (zero external internet connectivity or server installation required).  
2. Select an industry preset from the **Industry Architecture Presets** dropdown:  
   - **400G Cloud Data Center Interconnect**: 400 Gbps AWS/Azure DirectConnect link with GCM-AES-XPN-256, 256-bit CAK, strict fail-closed, and Replay Window 2048 (Grade A+).  
   - **10G Low-Latency Financial Metro WAN**: 10 Gbps financial interconnect with Confidentiality Offset 30 preserving 802.1Q tags for carrier transit (Grade A).  
   - **Vulnerable Legacy 100G Deployment**: Demonstrates fatal PN rollover within 28 seconds, missing replay protection, and cleartext fallback (Grade F).  
3. Review the **High-Speed Line Rate & Packet Number Exhaustion Metrics** to verify line-rate stability.  
4. Inspect the **10-Point Zero-Trust MACsec Security Audit** to evaluate design vulnerabilities and review step-by-step remediation advice.  
5. Review the **IEEE 802.1AE SecTAG Binary Dissection** and live **Wireshark Hex Dump** with 1-click clipboard copy.  
6. Switch across vendor tabs under **Multi-Vendor MACsec Synthesizer** (Cisco IOS-XR, Arista EOS, Juniper Junos, Linux Kernel, Prometheus Alerts) to copy verified configurations.

---

## 2\. DevSecOps CLI Tool (`macsecforge-cli.py`)

MACsecForge Pro includes a standalone, zero-dependency Python 3 CLI tool located at `build/macsecforge-cli.py`.

### 2.1 Encoding SecTAG Headers into Wireshark Hex Format

python3 build/macsecforge-cli.py encode-sectag \-p source-code/presets/dci-400g-xpn-cloud.json

Output:

\=================================================================

IEEE 802.1AE SecTAG Frame (16 Bytes):

\=================================================================

0000  88 e5 2c 00 00 00 30 a2  00 11 22 33 44 55 00 01  |..,...0..."3DU..|

\=================================================================

EtherType: 0x88E5 | Length: 16 Bytes

AN: 0 | SL: 0 | PN: 12450

SCI: 00:11:22:33:44:55 / Port 1 (0x0011223344550001)

Flags: SC=True, E=True, C=True

### 2.2 Calculating Packet Number (PN) & XPN Rollover Exhaustion

\# Calculate 400G line rate with worst-case 64-byte frames

python3 build/macsecforge-cli.py calc-xpn \--line-rate 400 \--frame-size 64 \--cipher GCM-AES-XPN-256

\# Verify 32-bit PN exhaustion on 100G link

python3 build/macsecforge-cli.py calc-xpn \--line-rate 100 \--frame-size 64 \--cipher GCM-AES-128

### 2.3 Calculating Interface MTU Headroom & Clamped TCP MSS

\# Standard 1500 byte payload with explicit SCI

python3 build/macsecforge-cli.py calc-mtu \--base-mtu 1500

\# Jumbo 9000 byte payload

python3 build/macsecforge-cli.py calc-mtu \--base-mtu 9000

### 2.4 Running 10-Point Zero-Trust Audits in CI/CD Pipelines

\# Regular audit

python3 build/macsecforge-cli.py audit \-p source-code/presets/dci-400g-xpn-cloud.json

\# Strict CI/CD gate: exits with code 1 if score \< 90 or grade is not A/A+

python3 build/macsecforge-cli.py audit \-p source-code/presets/dci-400g-xpn-cloud.json \--strict

### 2.5 Synthesizing Multi-Vendor Router Configurations

\# Cisco IOS-XR

python3 build/macsecforge-cli.py synthesize \-p source-code/presets/dci-400g-xpn-cloud.json \-f cisco

\# Arista EOS

python3 build/macsecforge-cli.py synthesize \-p source-code/presets/dci-400g-xpn-cloud.json \-f arista

\# Juniper Junos

python3 build/macsecforge-cli.py synthesize \-p source-code/presets/dci-400g-xpn-cloud.json \-f juniper

\# Linux Native Kernel MACsec (iproute2)

python3 build/macsecforge-cli.py synthesize \-p source-code/presets/dci-400g-xpn-cloud.json \-f linux

\# Prometheus Alerting Rules

python3 build/macsecforge-cli.py synthesize \-p source-code/presets/dci-400g-xpn-cloud.json \-f prometheus

---

## 3\. GitHub Actions CI/CD Integration

Embed MACsec configuration auditing directly into NetDevOps infrastructure repositories:

name: MACsec Zero-Trust Security Audit

on: \[push, pull\_request\]

jobs:

  audit-macsec-configs:

    runs-on: ubuntu-latest

    steps:

      \- uses: actions/checkout@v4

      \- name: Set up Python

        uses: actions/setup-python@v5

        with:

          python-version: '3.11'

      \- name: Audit Production DCI Presets

        run: |

          python3 build/macsecforge-cli.py audit \-p source-code/presets/dci-400g-xpn-cloud.json \--strict

          python3 build/macsecforge-cli.py audit \-p source-code/presets/metro-wan-financial-vlan.json \--strict