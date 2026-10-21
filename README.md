# MACsecForge Pro (v1.0.0)

> **Zero-Trust IEEE 802.1AE MACsec, 802.1AEbw XPN & 802.1X MKA Wire-Speed Encryption Studio, SecTAG Binary Wireframe Dissection, High-Speed Line Rate (10G–800G) PN Rollover Calculator, MTU/MSS Overhead Planner, 10-Point Security Auditor & Multi-Vendor Synthesizer.**

[![Standards](https://img.shields.io/badge/Standards-IEEE%20802.1AE%20%7C%20802.1AEbw%20XPN%20%7C%20802.1X--2010%20MKA-emerald)](#) [![Telemetry](https://img.shields.io/badge/Security-Wire--Speed%20L2%20%7C%20Zero--Trust%20Fail--Closed-cyan)](#) [![Tests](https://img.shields.io/badge/Tests-9%2F9%20Passed-green)](#) [![License](https://img.shields.io/badge/License-Commercial%20%2F%20Developer-blue)](#)

---

## Overview

Modern cloud architectures, high-frequency trading (HFT) financial institutions, and global enterprises rely on high-capacity (10G, 40G, 100G, 400G, 800G) point-to-point optical links to interconnect data centers (DCI) and public cloud dedicated circuits (AWS DirectConnect, Azure ExpressRoute, Google Cloud Dedicated Interconnect).

At these line rates, software IPsec is fundamentally unviable—it introduces severe CPU exhaustion, microsecond-level latency spikes, and packet fragmentation. **IEEE 802.1AE Media Access Control Security (MACsec)** solves this by encrypting and authenticating Ethernet frames directly in hardware PHY/ASIC at wire-speed with sub-microsecond latency.

However, real-world MACsec deployments suffer from catastrophic operational hazards:

1. **The 32-bit Packet Number (PN) Rollover Flap Hazard**: Under high-speed line rates, a standard 32-bit Packet Number counter (\$2^{32}-1 \= 4,294,967,295\$ packets) exhausts in seconds under small frame sizes\! At 100 Gbps, 32-bit PN rolls over in **28.8 seconds**; at 400 Gbps, it exhausts in **7.2 seconds**; and at 800 Gbps, it exhausts in **3.6 seconds**\! If standard MACsec is deployed without **IEEE 802.1AEbw Extended Packet Numbering (XPN)**, the MACsec Key Agreement (MKA) protocol must continuously negotiate new Security Association Keys (SAKs) every few seconds, triggering catastrophic control plane CPU spikes, packet drop bursts, or complete link flapping.  
2. **The "Should-Secure" Cleartext Data Leakage Hazard**: Permissive default configurations (e.g. Cisco `should-secure` or Arista `traffic-unprotected allow`) fall back to transmitting cleartext, unencrypted frames across the optical link if MKA authentication drops. In regulated environments (PCI-DSS v4.0, DoD, HIPAA), this represents an immediate critical breach.  
3. **Replay Protection vs QoS Queue Dropping**: MACsec anti-replay protection with `window-size 0` drops valid packets that arrive slightly out of sequence due to traffic prioritization across multi-queue QoS or link aggregation (LAG) hashing. Conversely, disabling replay protection exposes the fiber link to optical tap packet re-injection.  
4. **MTU Blackholes & TCP MSS Fragmentation**: MACsec adds 32 octets (16-byte SecTAG \+ 16-byte ICV) of Layer 2 overhead. If physical interface MTUs are not adjusted to at least 1532 bytes (or 9032 bytes for jumbo frames) or TCP MSS is not clamped, packets are silently dropped by intermediate carrier equipment without ICMP Fragmentation Needed notifications.

**MACsecForge Pro** provides a complete, 100% client-side Web Studio and standalone Python 3 DevSecOps CLI tool to encode valid IEEE 802.1AE frames, calculate exact PN rollover timelines, compute MTU headroom, audit network configurations against a 10-point Zero-Trust rubric (0–100%, Grades A+ to F), and synthesize production configurations for **Cisco IOS-XR**, **Arista EOS**, **Juniper Junos**, **Linux Native MACsec (iproute2)**, and **Prometheus Alerting Rules**.

---

## Key Capabilities

1. **Interactive SecTAG Binary Wireframe & Dissection Studio**:  
   - Encodes complete 8-byte (implicit SCI) and 16-byte (explicit SCI) IEEE 802.1AE SecTAG headers: EtherType `0x88E5`, TCI/AN (V, ES, SC, SCB, E, C, AN), Short Length (SL), Packet Number (PN), and 64-bit SCI.  
   - Generates formatted Wireshark-style hex dumps with live bit-level breakdown badges.  
2. **High-Speed Line Rate & 64-bit XPN Rollover Calculator**:  
   - Calculates exact packets-per-second, time until 32-bit PN exhaustion vs 64-bit XPN lifetime across 10G, 40G, 100G, 400G, and 800G line rates and frame sizes (64B to 9000B).  
   - Demonstrates how IEEE 802.1AEbw XPN extends SAK lifetime from 7.2 seconds to over **982 years** on 400G links.  
3. **MTU Headroom & TCP MSS Clamping Planner**:  
   - Computes precise Layer 2 overhead (32 bytes with SCI, 24 bytes without SCI, plus 4–8 bytes for 802.1Q/QinQ VLAN tags).  
   - Determines exact required interface physical MTU (e.g. 1532 or 9032\) and recommended clamped TCP MSS (e.g. 1428\) to prevent silent path MTU blackholes.  
4. **10-Point Zero-Trust MACsec & MKA Security Auditor (0–100%, Grades A+ to F)**:  
   - Evaluates: Cipher Suite Strength (GCM-AES-XPN-256), High-Speed PN Rollover Resilience, MKA Key Entropy (256-bit CAK & CKN), Replay Protection Window Sizing, Fail-Closed Policy Enforcement, Confidentiality Offset Privacy, MTU Headroom, MKA Rekey Timers, Explicit SCI Inclusion, and Telemetry/Prometheus Alerting.  
5. **Multi-Vendor Configuration Synthesizer**:  
   - **Cisco IOS-XR**: Key chains, MKA policy, MACsec policy, interface bindings.  
   - **Arista EOS**: MACsec profiles, MKA key configuration, interface security.  
   - **Juniper Junos**: Connectivity association, static CAK, interface security.  
   - **Linux Native Kernel**: `ip link add type macsec`, TX/RX SA configuration via `iproute2`.  
   - **Prometheus Alerting**: Real-time alerts for MKA session drops, ICV failures, replay drops, and PN exhaustion.  
6. **Standalone DevSecOps Python 3 CLI Tool (`macsecforge-cli.py`)**:  
   - Subcommands: `encode-sectag`, `calc-xpn`, `calc-mtu`, `audit`, `synthesize`.  
   - Supports `--strict` CI/CD gate returning exit code 1 if security grade is below A/90%.

---

## Commercial Licensing & Pricing

- **MACsec Engineer Solo (\$49)**: Single-seat license for network consultants and CCIE/JNCIE candidates.  
- **Carrier Team & Enterprise (\$99)**: Unlimited commercial use, white-label configuration rights, and CI/CD gate templates.  
- **Hosted Telemetry Micro-SaaS (\$29/mo)**: Automated continuous MKA drift scanning and Prometheus anomaly alerting.