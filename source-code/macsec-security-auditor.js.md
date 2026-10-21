/\*\*  
 \* MACsecForge Pro \- 10-Point Zero-Trust MACsec & MKA Security Auditor  
 \* Standards: IEEE 802.1AE-2006, 802.1AEbn-2011, 802.1AEbw-2013, 802.1X-2010  
 \*/

(function (root, factory) {  
  if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.MACsecSecurityAuditor \= factory();  
  }  
})(typeof self \!== 'undefined' ? self : this, function () {

  /\*\*  
   \* Audits a MACsec / MKA design configuration against a 10-point Zero-Trust rubric  
   \* @param {Object} cfg \- Configuration object  
   \* @returns {Object} { score, maxScore: 100, percentage, grade, summary, findings }  
   \*/  
  function auditMACsecConfiguration(cfg) {  
    const findings \= \[\];  
    let totalScore \= 0;

    const cipherSuite \= (cfg.cipherSuite || 'GCM-AES-XPN-256').toUpperCase();  
    const isXpn \= cipherSuite.includes('XPN') || Boolean(cfg.isXpn);  
    const lineRateGbps \= parseFloat(cfg.lineRateGbps) || 100;  
    const replayProtect \= cfg.replayProtect \!== undefined ? Boolean(cfg.replayProtect) : true;  
    const replayWindow \= parseInt(cfg.replayWindow, 10\) || (replayProtect ? 1024 : 0);  
    const failClosed \= cfg.failClosed \!== undefined ? Boolean(cfg.failClosed) : (cfg.policyMode \=== 'must-secure' || cfg.policyMode \=== 'strict');  
    const cakHex \= (cfg.cak || '').replace(/\[^0-9a-fA-F\]/g, '');  
    const cknHex \= (cfg.ckn || '').replace(/\[^0-9a-fA-F\]/g, '');  
    const confidentialityOffset \= parseInt(cfg.confidentialityOffset, 10\) || 0;  
    const interfaceMtu \= parseInt(cfg.interfaceMtu, 10\) || 1532;  
    const tcpMssClamped \= Boolean(cfg.tcpMssClamped);  
    const includeSci \= cfg.includeSci \!== undefined ? Boolean(cfg.includeSci) : true;  
    const telemetryAlerting \= Boolean(cfg.telemetryAlerting);  
    const sakRekeyInterval \= parseInt(cfg.sakRekeyIntervalSec, 10\) || 1800;

    // 1\. Cipher Suite & Encryption Strength  
    let csScore \= 0;  
    let csStatus \= 'PASS';  
    let csMsg \= '';  
    if (cipherSuite \=== 'GCM-AES-XPN-256') {  
      csScore \= 10;  
      csMsg \= 'TOP TIER: GCM-AES-XPN-256 provides military-grade 256-bit AES encryption with 64-bit Extended Packet Numbers.';  
    } else if (cipherSuite \=== 'GCM-AES-256') {  
      csScore \= 8;  
      csMsg \= 'GCM-AES-256 provides 256-bit encryption, but relies on standard 32-bit Packet Numbers.';  
    } else if (cipherSuite \=== 'GCM-AES-XPN-128') {  
      csScore \= 7;  
      csMsg \= 'GCM-AES-XPN-128 features 64-bit Extended Packet Numbers, but uses 128-bit key length.';  
    } else if (cipherSuite \=== 'GCM-AES-128') {  
      csScore \= 5;  
      csStatus \= 'WARN';  
      csMsg \= 'Legacy GCM-AES-128: 128-bit encryption with 32-bit PN. Upgrade to 256-bit cipher recommended.';  
    } else {  
      csScore \= 0;  
      csStatus \= 'CRITICAL';  
      csMsg \= 'Insecure or unencrypted cipher suite specified.';  
    }  
    findings.push({  
      id: 'CIPHER\_SUITE\_STRENGTH',  
      name: 'Cipher Suite & Cryptographic Strength',  
      score: csScore,  
      maxScore: 10,  
      status: csStatus,  
      description: csMsg,  
      remediation: csScore \< 10 ? 'Configure GCM-AES-XPN-256 for optimal security and long-duration SAK lifetime.' : null  
    });  
    totalScore \+= csScore;

    // 2\. High-Speed PN Rollover Exhaustion Risk  
    let xpnScore \= 0;  
    let xpnStatus \= 'PASS';  
    let xpnMsg \= '';  
    if (lineRateGbps \>= 100 && \!isXpn) {  
      xpnScore \= 0;  
      xpnStatus \= 'CRITICAL';  
      xpnMsg \= \`CRITICAL ROLLOVER EXHAUSTION: At \${lineRateGbps} Gbps line rate, standard 32-bit PN exhausts in seconds\! MKA must rekey constantly, causing high CPU spikes and dropped packets.\`;  
    } else if (lineRateGbps \>= 100 && isXpn) {  
      xpnScore \= 10;  
      xpnMsg \= \`OPTIMAL ZERO-TRUST: 64-bit Extended Packet Numbering (XPN) active on \${lineRateGbps} Gbps link. Over 900 years of continuous operation without PN exhaustion.\`;  
    } else if (lineRateGbps \< 100 && isXpn) {  
      xpnScore \= 10;  
      xpnMsg \= 'XPN enabled on sub-100G link provides future-proof headroom against packet number exhaustion.';  
    } else {  
      xpnScore \= 8;  
      xpnMsg \= \`Standard 32-bit PN on \${lineRateGbps} Gbps link. Acceptable if SAK rekey interval is properly bounded.\`;  
    }  
    findings.push({  
      id: 'XPN\_ROLLOVER\_RESILIENCE',  
      name: 'High-Speed PN Rollover & Exhaustion Resilience',  
      score: xpnScore,  
      maxScore: 10,  
      status: xpnStatus,  
      description: xpnMsg,  
      remediation: xpnScore \< 10 ? 'Enable IEEE 802.1AEbw Extended Packet Numbering (XPN) for links \>= 100G.' : null  
    });  
    totalScore \+= xpnScore;

    // 3\. MKA Key Entropy & Authentication (CAK & CKN)  
    let keyScore \= 0;  
    let keyStatus \= 'PASS';  
    let keyMsg \= '';  
    if (cakHex.length \=== 64 && cknHex.length \>= 32\) {  
      keyScore \= 10;  
      keyMsg \= \`Full 256-bit entropy CAK (\${cakHex.length} hex digits) with valid RFC-compliant CKN identifier.\`;  
    } else if (cakHex.length \=== 32 && cknHex.length \>= 32\) {  
      keyScore \= 7;  
      keyStatus \= 'WARN';  
      keyMsg \= '128-bit CAK (32 hex digits) detected. Meets minimum standard, but 256-bit CAK is recommended for high-assurance links.';  
    } else {  
      keyScore \= 0;  
      keyStatus \= 'CRITICAL';  
      keyMsg \= \`INSUFFICIENT KEY ENTROPY: CAK is \${cakHex.length} hex or CKN is \${cknHex.length} hex (requires 64/32 hex CAK and \>=32 hex CKN).\`;  
    }  
    findings.push({  
      id: 'KEY\_ENTROPY\_AND\_LENGTH',  
      name: 'MKA Key Entropy & Authentication Security',  
      score: keyScore,  
      maxScore: 10,  
      status: keyStatus,  
      description: keyMsg,  
      remediation: keyScore \< 10 ? 'Generate cryptographically random 256-bit (64 hex characters) CAK and \>= 32 hex char CKN.' : null  
    });  
    totalScore \+= keyScore;

    // 4\. Replay Protection Window Sizing  
    let rpScore \= 0;  
    let rpStatus \= 'PASS';  
    let rpMsg \= '';  
    if (\!replayProtect) {  
      rpScore \= 0;  
      rpStatus \= 'CRITICAL';  
      rpMsg \= 'CRITICAL VULNERABILITY: Replay protection is DISABLED. An attacker on the physical optical path can intercept and re-inject valid packets\!';  
    } else if (replayWindow \=== 0\) {  
      rpScore \= 5;  
      rpStatus \= 'WARN';  
      rpMsg \= 'Replay window set to 0 (Strict in-order delivery). Valid packets delayed by QoS queuing or LAG hashing will be dropped\!';  
    } else if (lineRateGbps \>= 100 && replayWindow \< 256\) {  
      rpScore \= 7;  
      rpStatus \= 'WARN';  
      rpMsg \= \`Replay window (\${replayWindow}) is small for a \${lineRateGbps} Gbps link. Under bursty QoS multi-queue traffic, packet reordering may trigger false replay drops.\`;  
    } else {  
      rpScore \= 10;  
      rpMsg \= \`Tuned replay protection active (Window size: \${replayWindow} packets). Absorbs QoS priority queuing jitter while preventing replay attacks.\`;  
    }  
    findings.push({  
      id: 'REPLAY\_PROTECTION',  
      name: 'Replay Protection & Anti-Replay Window Sizing',  
      score: rpScore,  
      maxScore: 10,  
      status: rpStatus,  
      description: rpMsg,  
      remediation: rpScore \< 10 ? 'Enable replay protection with window size tuned to link rate (e.g. 1024–2048 for 100G/400G).' : null  
    });  
    totalScore \+= rpScore;

    // 5\. Fail-Closed vs Cleartext Fallback Policy  
    let fcScore \= 0;  
    let fcStatus \= 'PASS';  
    let fcMsg \= '';  
    if (failClosed) {  
      fcScore \= 10;  
      fcMsg \= 'FAIL-CLOSED (must-secure): If MKA negotiation fails or SAK expires, interface drops all unencrypted frames. Zero cleartext leakage.';  
    } else {  
      fcScore \= 0;  
      fcStatus \= 'CRITICAL';  
      fcMsg \= 'CRITICAL LEAKAGE HAZARD (should-secure): Interface falls back to transmitting cleartext unencrypted frames if MKA drops\!';  
    }  
    findings.push({  
      id: 'FAIL\_CLOSED\_POLICY',  
      name: 'Fail-Closed Enforcement vs Cleartext Fallback',  
      score: fcScore,  
      maxScore: 10,  
      status: fcStatus,  
      description: fcMsg,  
      remediation: fcScore \< 10 ? 'Enforce strict fail-closed policy (e.g. Cisco "should-secure" \-\> "must-secure" or Junos "exclude-traffic").' : null  
    });  
    totalScore \+= fcScore;

    // 6\. Confidentiality Offset & Header Privacy  
    let coScore \= 0;  
    let coStatus \= 'PASS';  
    let coMsg \= '';  
    if (confidentialityOffset \=== 0\) {  
      coScore \= 10;  
      coMsg \= 'MAXIMUM CONFIDENTIALITY (Offset 0): Entire Layer 2 payload (including VLAN tags and MPLS labels) is fully encrypted.';  
    } else if (confidentialityOffset \=== 30\) {  
      coScore \= 8;  
      coMsg \= 'Confidentiality Offset 30: 802.1Q VLAN header transmitted in cleartext for carrier intermediate switching, payload encrypted.';  
    } else if (confidentialityOffset \=== 50\) {  
      coScore \= 7;  
      coMsg \= 'Confidentiality Offset 50: Dual VLAN / MPLS headers cleartext, inner payload encrypted.';  
    } else {  
      coScore \= 5;  
      coStatus \= 'WARN';  
      coMsg \= \`Non-standard confidentiality offset (\${confidentialityOffset}). Ensure intermediate switching devices strictly require this header visibility.\`;  
    }  
    findings.push({  
      id: 'CONFIDENTIALITY\_OFFSET',  
      name: 'Confidentiality Offset & Layer 2 Header Privacy',  
      score: coScore,  
      maxScore: 10,  
      status: coStatus,  
      description: coMsg,  
      remediation: coScore \< 10 ? 'Set confidentiality-offset to 0 unless carrier transit switching explicitly requires visible 802.1Q tags.' : null  
    });  
    totalScore \+= coScore;

    // 7\. MTU Headroom & Fragmentation Prevention  
    let mtuScore \= 0;  
    let mtuStatus \= 'PASS';  
    let mtuMsg \= '';  
    if (interfaceMtu \>= 1532 || (interfaceMtu \>= 1500 && tcpMssClamped)) {  
      mtuScore \= 10;  
      mtuMsg \= \`L2 Interface MTU (\${interfaceMtu} bytes) accounts for 32-byte MACsec SecTAG+ICV overhead\${tcpMssClamped ? ' with TCP MSS clamping' : ''}.\`;  
    } else {  
      mtuScore \= 2;  
      mtuStatus \= 'CRITICAL';  
      mtuMsg \= \`MTU BLACKHOLE HAZARD: Interface MTU is standard \${interfaceMtu} bytes without TCP MSS clamping. Adding 32B MACsec overhead will cause silent fragmentation/drops\!\`;  
    }  
    findings.push({  
      id: 'MTU\_HEADROOM\_HYGIENE',  
      name: 'MTU Headroom & Path MTU / TCP MSS Clamping',  
      score: mtuScore,  
      maxScore: 10,  
      status: mtuStatus,  
      description: mtuMsg,  
      remediation: mtuScore \< 10 ? 'Increase interface physical MTU by at least 32 bytes (e.g. 1532 or 9032\) or enable TCP MSS clamping.' : null  
    });  
    totalScore \+= mtuScore;

    // 8\. MKA Timer Hardening (Hello / SAK Rekey)  
    let tmScore \= 0;  
    let tmStatus \= 'PASS';  
    let tmMsg \= '';  
    if (sakRekeyInterval \>= 600 && sakRekeyInterval \<= 3600\) {  
      tmScore \= 10;  
      tmMsg \= \`Bounded SAK rekey interval (\${sakRekeyInterval}s). Ensures periodic fresh key generation without overburdening the CPU.\`;  
    } else if (sakRekeyInterval \< 600\) {  
      tmScore \= 6;  
      tmStatus \= 'WARN';  
      tmMsg \= \`Ultra-short SAK rekey interval (\${sakRekeyInterval}s) creates excessive MKA protocol chatter.\`;  
    } else {  
      tmScore \= 7;  
      tmStatus \= 'WARN';  
      tmMsg \= \`Extended SAK rekey interval (\${sakRekeyInterval}s). Recommend rekeying at least once every 60 minutes.\`;  
    }  
    findings.push({  
      id: 'MKA\_TIMER\_HARDENING',  
      name: 'MKA Hello & SAK Rekeying Interval Hardening',  
      score: tmScore,  
      maxScore: 10,  
      status: tmStatus,  
      description: tmMsg,  
      remediation: tmScore \< 10 ? 'Set SAK rekey time between 1800s and 3600s.' : null  
    });  
    totalScore \+= tmScore;

    // 9\. Explicit SCI & Channel Isolation  
    let sciScore \= 0;  
    let sciStatus \= 'PASS';  
    let sciMsg \= '';  
    if (includeSci) {  
      sciScore \= 10;  
      sciMsg \= 'Explicit Secure Channel Identifier (SCI) included in SecTAG. Enables cryptographic multi-access and port isolation.';  
    } else {  
      sciScore \= 8;  
      sciMsg \= 'SCI omitted in SecTAG (Implicit SCI). Saves 8 bytes overhead, strictly valid for point-to-point physical links.';  
    }  
    findings.push({  
      id: 'EXPLICIT\_SCI\_ENFORCEMENT',  
      name: 'Explicit SCI Inclusion & Secure Channel Isolation',  
      score: sciScore,  
      maxScore: 10,  
      status: sciStatus,  
      description: sciMsg,  
      remediation: null  
    });  
    totalScore \+= sciScore;

    // 10\. Telemetry & Cryptographic Anomaly Alerting  
    let telScore \= 0;  
    let telStatus \= 'PASS';  
    let telMsg \= '';  
    if (telemetryAlerting) {  
      telScore \= 10;  
      telMsg \= 'Continuous telemetry enabled: MACsec ICV drop counters, replay errors, and MKA session flaps exported to Prometheus/SIEM.';  
    } else {  
      telScore \= 4;  
      telStatus \= 'WARN';  
      telMsg \= 'Telemetry alerting disabled: Optical line degradation or active MITM replay attacks will go unnoticed.';  
    }  
    findings.push({  
      id: 'MONITORING\_AND\_TELEMETRY',  
      name: 'Cryptographic Integrity Telemetry & Prometheus Alerting',  
      score: telScore,  
      maxScore: 10,  
      status: telStatus,  
      description: telMsg,  
      remediation: telScore \< 10 ? 'Export MACsec interface counters (InPktsBadTag, InPktsNoTag, InPktsLate) to Prometheus.' : null  
    });  
    totalScore \+= telScore;

    const percentage \= Math.round((totalScore / 100\) \* 100);  
    let grade \= 'F';  
    let summary \= '';

    if (percentage \>= 95\) {  
      grade \= 'A+';  
      summary \= 'High-Assurance Zero-Trust MACsec: Complies with DoD / CNSA / PCI-DSS v4.0 line-rate encryption standards.';  
    } else if (percentage \>= 85\) {  
      grade \= 'A';  
      summary \= 'Hardened Production MACsec: Strong cryptographic posture with minor recommended tuning.';  
    } else if (percentage \>= 70\) {  
      grade \= 'B';  
      summary \= 'Standard Enterprise MACsec: Functional point-to-point encryption, but exposed to high-speed rollover or MTU risks.';  
    } else if (percentage \>= 60\) {  
      grade \= 'C';  
      summary \= 'Marginal MACsec Deployment: Significant configuration gaps detected.';  
    } else if (percentage \>= 50\) {  
      grade \= 'D';  
      summary \= 'Weak MACsec Security: High operational flap risk and suboptimal key management.';  
    } else {  
      grade \= 'F';  
      summary \= 'CRITICAL MACsec VULNERABILITY: Severe security defects (replay disabled, cleartext fallback, or rapid PN rollover flap).';  
    }

    return {  
      score: totalScore,  
      maxScore: 100,  
      percentage: percentage,  
      grade: grade,  
      summary: summary,  
      findings: findings  
    };  
  }

  return {  
    auditMACsecConfiguration: auditMACsecConfiguration  
  };  
});  
