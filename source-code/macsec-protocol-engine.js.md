/\*\*  
 \* MACsecForge Pro \- IEEE 802.1AE Protocol & Frame Engine  
 \* Standards: IEEE 802.1AE-2006, IEEE 802.1AEbn-2011, IEEE 802.1AEbw-2013 (XPN), IEEE 802.1X-2010 (MKA)  
 \*/

(function (root, factory) {  
  if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.MACsecProtocolEngine \= factory();  
  }  
})(typeof self \!== 'undefined' ? self : this, function () {

  const CIPHER\_SUITES \= {  
    'GCM-AES-XPN-256': {  
      id: '00-80-C2-00-01-04',  
      name: 'GCM-AES-XPN-256',  
      keyBits: 256,  
      pnBits: 64,  
      icvBytes: 16,  
      xpn: true,  
      description: 'Galois/Counter Mode AES-256 with 64-bit Extended Packet Numbers (IEEE 802.1AEbw)'  
    },  
    'GCM-AES-256': {  
      id: '00-80-C2-00-01-02',  
      name: 'GCM-AES-256',  
      keyBits: 256,  
      pnBits: 32,  
      icvBytes: 16,  
      xpn: false,  
      description: 'Galois/Counter Mode AES-256 with 32-bit Packet Numbers (IEEE 802.1AEbn)'  
    },  
    'GCM-AES-XPN-128': {  
      id: '00-80-C2-00-01-03',  
      name: 'GCM-AES-XPN-128',  
      keyBits: 128,  
      pnBits: 64,  
      icvBytes: 16,  
      xpn: true,  
      description: 'Galois/Counter Mode AES-128 with 64-bit Extended Packet Numbers (IEEE 802.1AEbw)'  
    },  
    'GCM-AES-128': {  
      id: '00-80-C2-00-01-01',  
      name: 'GCM-AES-128',  
      keyBits: 128,  
      pnBits: 32,  
      icvBytes: 16,  
      xpn: false,  
      description: 'Galois/Counter Mode AES-128 with 32-bit Packet Numbers (IEEE 802.1AE base)'  
    }  
  };

  /\*\*  
   \* Encodes an IEEE 802.1AE Security Tag (SecTAG)  
   \* Header format:  
   \*  \- MACsec EtherType: 2 octets (0x88E5)  
   \*  \- TCI/AN: 1 octet (V\[7\], ES\[6\], SC\[5\], SCB\[4\], E\[3\], C\[2\], AN\[1:0\])  
   \*  \- SL (Short Length): 1 octet (Reserved\[7:6\]=0, SL\[5:0\])  
   \*  \- PN (Packet Number): 4 octets  
   \*  \- SCI (Secure Channel Identifier): 8 octets (Present only if SC \== 1\)  
   \*/  
  function encodeSecTAG(options) {  
    const opts \= options || {};  
    const v \= opts.v ? 1 : 0;  
    const es \= opts.es ? 1 : 0;  
    const sc \= opts.sc \!== undefined ? (opts.sc ? 1 : 0\) : 1; // Default: explicit SCI  
    const scb \= opts.scb ? 1 : 0;  
    const e \= opts.e \!== undefined ? (opts.e ? 1 : 0\) : 1; // Default: encrypted  
    const c \= opts.c \!== undefined ? (opts.c ? 1 : 0\) : 1; // Default: changed text  
    const an \= (parseInt(opts.an, 10\) || 0\) & 0x03;  
    const sl \= (parseInt(opts.sl, 10\) || 0\) & 0x3F;  
    const pn \= (parseInt(opts.pn, 10\) || 1\) \>\>\> 0;  
    const sciHex \= (opts.sci || '0011223344550001').replace(/\[^0-9a-fA-F\]/g, '');

    const headerLen \= sc \=== 1 ? 16 : 8;  
    const buf \= new Uint8Array(headerLen);

    // Byte 0-1: EtherType 0x88E5  
    buf\[0\] \= 0x88;  
    buf\[1\] \= 0xE5;

    // Byte 2: TCI / AN  
    buf\[2\] \= ((v & 1\) \<\< 7\) |  
             ((es & 1\) \<\< 6\) |  
             ((sc & 1\) \<\< 5\) |  
             ((scb & 1\) \<\< 4\) |  
             ((e & 1\) \<\< 3\) |  
             ((c & 1\) \<\< 2\) |  
             (an & 3);

    // Byte 3: Short Length  
    buf\[3\] \= sl & 0x3F;

    // Bytes 4-7: Packet Number (32-bit big endian)  
    buf\[4\] \= (pn \>\>\> 24\) & 0xFF;  
    buf\[5\] \= (pn \>\>\> 16\) & 0xFF;  
    buf\[6\] \= (pn \>\>\> 8\) & 0xFF;  
    buf\[7\] \= pn & 0xFF;

    // Bytes 8-15: SCI (if SC bit set)  
    if (sc \=== 1\) {  
      const paddedSci \= sciHex.padEnd(16, '0').slice(0, 16);  
      for (let i \= 0; i \< 8; i++) {  
        buf\[8 \+ i\] \= parseInt(paddedSci.substr(i \* 2, 2), 16\) || 0;  
      }  
    }

    return buf;  
  }

  /\*\*  
   \* Decodes an IEEE 802.1AE Security Tag (SecTAG)  
   \*/  
  function decodeSecTAG(buf) {  
    if (\!buf || buf.length \< 8\) {  
      throw new Error('SecTAG buffer must be at least 8 bytes');  
    }  
    const etherType \= (buf\[0\] \<\< 8\) | buf\[1\];  
    if (etherType \!== 0x88E5) {  
      throw new Error('Invalid MACsec EtherType: 0x' \+ etherType.toString(16) \+ ' (expected 0x88E5)');  
    }  
    const tciAn \= buf\[2\];  
    const v \= (tciAn \>\> 7\) & 1;  
    const es \= (tciAn \>\> 6\) & 1;  
    const sc \= (tciAn \>\> 5\) & 1;  
    const scb \= (tciAn \>\> 4\) & 1;  
    const e \= (tciAn \>\> 3\) & 1;  
    const c \= (tciAn \>\> 2\) & 1;  
    const an \= tciAn & 3;

    const sl \= buf\[3\] & 0x3F;  
    const pn \= ((buf\[4\] \<\< 24\) \>\>\> 0\) | (buf\[5\] \<\< 16\) | (buf\[6\] \<\< 8\) | buf\[7\];

    let sci \= null;  
    let sciMac \= null;  
    let sciPort \= null;  
    if (sc \=== 1\) {  
      if (buf.length \< 16\) {  
        throw new Error('SecTAG with SC=1 requires 16 bytes');  
      }  
      const rawHex \= Array.from(buf.slice(8, 16)).map(b \=\> b.toString(16).padStart(2, '0')).join('');  
      const macParts \= \[\];  
      for (let i \= 0; i \< 6; i++) {  
        macParts.push(rawHex.substr(i \* 2, 2));  
      }  
      sciMac \= macParts.join(':');  
      sciPort \= (buf\[14\] \<\< 8\) | buf\[15\];  
      sci \= \`\${sciMac} / Port \${sciPort} (0x\${rawHex})\`;  
    }

    return {  
      etherType: '0x88E5',  
      version: v,  
      endStation: Boolean(es),  
      secureChannel: Boolean(sc),  
      singleCopyBroadcast: Boolean(scb),  
      encryption: Boolean(e),  
      changedText: Boolean(c),  
      associationNumber: an,  
      shortLength: sl,  
      packetNumber: pn,  
      sci: sci,  
      sciMac: sciMac,  
      sciPort: sciPort,  
      headerLength: sc \=== 1 ? 16 : 8  
    };  
  }

  /\*\*  
   \* Calculates Packet Number (PN) & XPN Rollover Exhaustion at Line Rate  
   \* Computes exact packets-per-second, time until rollover, and risk level.  
   \*/  
  function calculatePNDuration(options) {  
    const opts \= options || {};  
    const lineRateGbps \= parseFloat(opts.lineRateGbps) || 100;  
    const avgFrameSizeBytes \= parseInt(opts.avgFrameSizeBytes, 10\) || 64; // Minimum 64 bytes for worst-case  
    const cipherSuite \= opts.cipherSuite || 'GCM-AES-XPN-256';  
    const isXpn \= cipherSuite.includes('XPN') || Boolean(opts.isXpn);

    // Ethernet physical framing: Preamble (7B) \+ SFD (1B) \+ Frame (L bytes) \+ IFG (12B) \= L \+ 20 bytes on wire  
    const onWireBytes \= Math.max(64, avgFrameSizeBytes) \+ 20;  
    const onWireBits \= onWireBytes \* 8;  
    const bitsPerSec \= lineRateGbps \* 1e9;  
    const maxPacketsPerSec \= bitsPerSec / onWireBits;

    // Standard 32-bit PN capacity vs 64-bit XPN capacity  
    const pnLimit32 \= 4294967295; // 2^32 \- 1  
    // 64-bit limit is 2^64 \- 1  
    const pnLimit64 \= 18446744073709551615;

    const secondsToExhaust32 \= pnLimit32 / maxPacketsPerSec;  
    const secondsToExhaustXpn \= pnLimit64 / maxPacketsPerSec;

    const actualSeconds \= isXpn ? secondsToExhaustXpn : secondsToExhaust32;

    let timeFormatted \= '';  
    let riskLevel \= 'LOW';  
    let recommendation \= '';

    if (actualSeconds \< 60\) {  
      timeFormatted \= \`\${actualSeconds.toFixed(1)} seconds\`;  
      riskLevel \= 'CRITICAL';  
      recommendation \= 'CRITICAL ROLLOVER EXHAUSTION: 32-bit PN will exhaust in seconds at this line rate\! Upgrade to GCM-AES-XPN-256 immediately.';  
    } else if (actualSeconds \< 3600\) {  
      const mins \= actualSeconds / 60;  
      timeFormatted \= \`\${mins.toFixed(1)} minutes\`;  
      riskLevel \= 'HIGH';  
      recommendation \= 'HIGH FREQUENT REKEYING: SAK rekeying required every few minutes, burning CPU and risking transit packet drops.';  
    } else if (actualSeconds \< 86400 \* 30\) {  
      const days \= actualSeconds / 86400;  
      timeFormatted \= \`\${days.toFixed(1)} days\`;  
      riskLevel \= 'MODERATE';  
      recommendation \= 'Acceptable for lower speed links, but monitor MKA rekey timers and ensure Rekey-on-PN threshold is set.';  
    } else {  
      const years \= actualSeconds / (86400 \* 365.25);  
      if (years \> 1000\) {  
        timeFormatted \= '\> 1,000 years';  
      } else {  
        timeFormatted \= \`\${years.toFixed(1)} years\`;  
      }  
      riskLevel \= 'SAFE';  
      recommendation \= 'OPTIMAL ZERO-TRUST RUNTIME: 64-bit Extended Packet Numbering eliminates premature rekeying fatigue and session flap risk.';  
    }

    return {  
      lineRateGbps: lineRateGbps,  
      avgFrameSizeBytes: avgFrameSizeBytes,  
      cipherSuite: cipherSuite,  
      isXpn: isXpn,  
      maxPacketsPerSec: Math.round(maxPacketsPerSec),  
      timeFormatted: timeFormatted,  
      secondsToExhaust: actualSeconds,  
      secondsToExhaust32: secondsToExhaust32,  
      secondsToExhaust64: secondsToExhaustXpn,  
      riskLevel: riskLevel,  
      recommendation: recommendation  
    };  
  }

  /\*\*  
   \* Calculates MTU Headroom, Wire Overhead & TCP MSS Clamping  
   \*/  
  function calculateMTUHeadroom(options) {  
    const opts \= options || {};  
    const basePayloadMtu \= parseInt(opts.basePayloadMtu, 10\) || 1500;  
    const includeSci \= opts.includeSci \!== undefined ? Boolean(opts.includeSci) : true;  
    const vlanCount \= parseInt(opts.vlanCount, 10\) || 0; // 0=None, 1=802.1Q (4B), 2=QinQ (8B)  
    const confidentialityOffset \= parseInt(opts.confidentialityOffset, 10\) || 0; // 0, 30, 50

    const secTagBytes \= includeSci ? 16 : 8;  
    const icvBytes \= 16;  
    const vlanBytes \= vlanCount \* 4;  
    const macsecOverhead \= secTagBytes \+ icvBytes;  
    const totalL2Overhead \= 14 \+ vlanBytes \+ macsecOverhead; // 14B Ethernet header (DMAC, SMAC, EtherType)  
    const requiredInterfaceL2Mtu \= basePayloadMtu \+ macsecOverhead \+ vlanBytes;

    // TCP MSS Clamping calculation (IPv4 standard: MTU \- 20B IP \- 20B TCP)  
    const standardIpv4Mss \= basePayloadMtu \- 40;  
    const clampedMssIfFixedMtu \= standardIpv4Mss \- macsecOverhead;

    return {  
      basePayloadMtu: basePayloadMtu,  
      secTagBytes: secTagBytes,  
      icvBytes: icvBytes,  
      vlanBytes: vlanBytes,  
      macsecOverhead: macsecOverhead,  
      requiredInterfaceL2Mtu: requiredInterfaceL2Mtu,  
      confidentialityOffset: confidentialityOffset,  
      recommendedTcpMss: clampedMssIfFixedMtu,  
      jumboMtuRecommendation: basePayloadMtu \>= 9000 ? (9000 \+ macsecOverhead \+ vlanBytes) : 1532  
    };  
  }

  /\*\*  
   \* Formats binary buffer into Wireshark / hexdump format  
   \*/  
  function formatHexDump(buf) {  
    const lines \= \[\];  
    for (let i \= 0; i \< buf.length; i \+= 16\) {  
      const chunk \= buf.slice(i, i \+ 16);  
      const hexParts \= \[\];  
      const asciiParts \= \[\];  
      for (let j \= 0; j \< 16; j++) {  
        if (j \< chunk.length) {  
          const b \= chunk\[j\];  
          hexParts.push(b.toString(16).padStart(2, '0'));  
          asciiParts.push((b \>= 32 && b \<= 126\) ? String.fromCharCode(b) : '.');  
        } else {  
          hexParts.push('  ');  
          asciiParts.push(' ');  
        }  
      }  
      const left \= hexParts.slice(0, 8).join(' ');  
      const right \= hexParts.slice(8, 16).join(' ');  
      const offset \= i.toString(16).padStart(4, '0');  
      lines.push(\`\${offset}  \${left}  \${right}  |\${asciiParts.join('')}|\`);  
    }  
    return lines.join('\\n');  
  }

  return {  
    CIPHER\_SUITES: CIPHER\_SUITES,  
    encodeSecTAG: encodeSecTAG,  
    decodeSecTAG: decodeSecTAG,  
    calculatePNDuration: calculatePNDuration,  
    calculateMTUHeadroom: calculateMTUHeadroom,  
    formatHexDump: formatHexDump  
  };  
});  
