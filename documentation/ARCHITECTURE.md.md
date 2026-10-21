# MACsecForge Pro \- Technical Architecture & Protocol Engineering Manual

## 1\. Standards Foundation & Scope

MACsecForge Pro implements the core IEEE 802 LAN/MAN Standards Committee specifications governing Media Access Control (MAC) Security and Key Agreement:

- **IEEE 802.1AE-2006**: Media Access Control (MAC) Security (MACsec base standard)  
- **IEEE 802.1AEbn-2011**: MACsec Amendment 1: Galois/Counter Mode-Advanced Encryption Standard-256 (GCM-AES-256) Cipher Suite  
- **IEEE 802.1AEbw-2013**: MACsec Amendment 2: Extended Packet Numbering (XPN) with GCM-AES-XPN-128 and GCM-AES-XPN-256 (64-bit Packet Numbers)  
- **IEEE 802.1AEcg-2017**: MACsec Amendment 3: Ethernet Data Encryption Devices (EDE-M, EDE-CS, EDE-SS)  
- **IEEE 802.1X-2010**: Port-Based Network Access Control (incorporating MKA \- MACsec Key Agreement Protocol)  
- **RFC 7634 / RFC 7635**: ChaCha20-Poly1305 Cipher Suites for MACsec and IETF Interoperability Guidelines

---

## 2\. IEEE 802.1AE Frame Format & SecTAG Binary Wireframe

MACsec operates directly above the Ethernet Physical (PHY) layer and below the Logical Link Control (LLC) / Network (IP) layer. It encrypts and authenticates everything following the Security Tag (SecTAG), terminating with an Integrity Check Value (ICV).

### 2.1 Complete Encapsulated Frame Layout

\+-----------------------------------------------------------------------------------------+

| DMAC (6B) | SMAC (6B) | \[802.1Q (4B)\]\* | SecTAG (8 or 16B) | User Data (Payload) | ICV (16B) | FCS (4B) |

\+-----------------------------------------------------------------------------------------+

                                          |\<--------- Authenticated Data \--------\>|

                                                              |\<--- Encrypted \---\>|

*\* Note: If Confidentiality Offset (CO) \> 0, the first CO bytes (e.g., 30 bytes for 802.1Q or 50 bytes for QinQ/MPLS) remain in cleartext but are cryptographically authenticated by the ICV.*

### 2.2 SecTAG Structure (8 or 16 Octets)

 0                   1                   2                   3

 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1

\+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

|       EtherType (0x88E5)      |    TCI/AN     |  SL (Short)   |

\+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

|                      Packet Number (PN)                       |

\+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

|            Secure Channel Identifier (SCI) \- Octets 0-3       | (If SC=1)

\+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

|            Secure Channel Identifier (SCI) \- Octets 4-7       | (If SC=1)

\+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

### 2.3 Bit-Field Specifications:

1. **EtherType (16 bits)**: Fixed value `0x88E5` identifying IEEE 802.1AE MAC Security frames.  
2. **TCI / AN (Tag Control Information & Association Number, 8 bits)**:  
   - `V (Version, bit 7)`: Version bit. Set to `0` for IEEE 802.1AE.  
   - `ES (End Station, bit 6)`: Set if the transmitting system is an end station.  
   - `SC (Secure Channel, bit 5)`: If `1`, the 8-byte SCI is explicitly included in the SecTAG. If `0`, the SCI is omitted (implicit SCI derived from the receiving port and SMAC).  
   - `SCB (Single Copy Broadcast, bit 4)`: Used in EPON or shared media.  
   - `E (Encryption, bit 3)`: If `1`, user data is encrypted. If `0`, integrity-only protection is applied.  
   - `C (Changed Text, bit 2)`: Indicates user data has changed (always `1` when E=`1`).  
   - `AN (Association Number, bits 1:0)`: 2-bit identifier (values 0–3) specifying which Security Association (SA) within the Secure Channel (SC) generated this frame.  
3. **SL (Short Length, 8 bits)**:  
   - Bits 7–6 are reserved (`00`).  
   - Bits 5–0 specify the user data length if shorter than 48 octets (to prevent padding oracle issues with runt frames). If user data is 48 octets or longer, SL is set to `0`.  
4. **Packet Number (PN, 32 bits)**:  
   - Monotonically increasing counter transmitted in big-endian network byte order.  
   - Used in standard AES-GCM as the Initialization Vector (IV) component.  
5. **Secure Channel Identifier (SCI, 64 bits)**:  
   - Present when SC bit \= 1\.  
   - Formatted as: 48-bit System MAC Address \+ 16-bit Port Identifier.

---

## 3\. Extended Packet Numbering (XPN) & Rollover Dynamics (IEEE 802.1AEbw)

On modern multi-hundred gigabit links (100G, 400G, 800G), a 32-bit counter (\$2^{32} \- 1 \= 4,294,967,295\$ packets) exhausts in mere seconds under minimum frame sizes:

\$\$\\text{Packets Per Second (PPS)} \= \\frac{\\text{Line Rate (bps)}}{(\\text{Frame Size (Bytes)} \+ 20\\text{ B on-wire overhead}) \\times 8}\$\$

\$\$\\text{Time to Exhaustion} \= \\frac{4,294,967,295}{\\text{PPS}}\$\$

### 3.1 Exhaustion Timeline Across Line Rates (Worst-Case 64B Frames)

| Line Rate | Wire PPS (64B Frames) | 32-bit PN Exhaustion | 64-bit XPN Exhaustion | Risk Level |
| :---- | :---- | :---- | :---- | :---- |
| **10 Gbps** | 14.88 Mpps | \~288.6 seconds (4.8 min) | \~39,300 years | Moderate (32-bit) / Safe (XPN) |
| **40 Gbps** | 59.52 Mpps | \~72.1 seconds (1.2 min) | \~9,825 years | High (32-bit) / Safe (XPN) |
| **100 Gbps** | 148.81 Mpps | **\~28.8 seconds** | \~3,930 years | **CRITICAL (32-bit)** / Safe (XPN) |
| **400 Gbps** | 595.24 Mpps | **\~7.2 seconds** | \~982 years | **FATAL FLAP (32-bit)** / Safe (XPN) |
| **800 Gbps** | 1,190.48 Mpps | **\~3.6 seconds** | \~491 years | **FATAL FLAP (32-bit)** / Safe (XPN) |

### 3.2 XPN Mechanics:

In IEEE 802.1AEbw, the frame still carries only the lower 32 bits of the PN in the on-wire SecTAG (preserving header efficiency). The sender and receiver independently maintain the upper 32 bits of the 64-bit counter in local SA state, incrementing it every time the lower 32 bits roll over from `0xFFFFFFFF` to `0x00000001`. A 12-byte Salt is derived via MKA and XORed with the 64-bit XPN to construct the 96-bit AES-GCM IV.

---

## 4\. Replay Protection & QoS Window Sizing

MACsec includes hardware anti-replay protection. When enabled, the receiver tracks the highest accepted Packet Number (\$PN\_{\\text{max}}\$). Any incoming packet with \$PN \< (PN\_{\\text{max}} \- \\text{Replay Window})\$ is dropped as a replay attack and increments the `InPktsLate` counter.

- **Window \= 0 (Strict in-order)**: Any packet arriving out of order is dropped. This breaks networks utilizing multi-queue QoS (where Voice/Video packets bypass Bulk data packets) or LAG hashing across member links.  
- **Tuned Window (e.g. 1024 for 100G, 2048 for 400G)**: Allows packets to arrive up to \$N\$ packets out of sequence, absorbing transmission jitter across QoS queues while strictly blocking re-injected traffic.

---

## 5\. Fail-Closed vs Cleartext Fallback Policy

- **`must-secure` (Fail-Closed)**: If MKA authentication fails, the pre-shared key is mismatched, or physical line noise disrupts SAK installation, the interface blocks all transit data frames. **Zero cleartext data is leaked.**  
- **`should-secure` (Permissive / Cleartext Fallback)**: If MKA drops, the interface falls back to transmitting unencrypted Layer 2 frames. In zero-trust and compliance environments (PCI-DSS v4.0, DoD, HIPAA), this represents a **critical vulnerability**.