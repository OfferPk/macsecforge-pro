/\*\*  
 \* MACsecForge Pro \- Studio Application Controller  
 \*/

document.addEventListener('DOMContentLoaded', () \=\> {  
  const PRESETS \= {  
    'dci-400g-xpn-cloud': {  
      name: "400G Cloud Data Center Interconnect (AWS DirectConnect / Azure ExpressRoute)",  
      badge: "400G Cloud DCI",  
      description: "High-capacity 400 Gbps Data Center Interconnect with GCM-AES-XPN-256 (64-bit Extended Packet Numbers), 256-bit CAK, strict fail-closed enforcement, Replay Window 2048, and Offset 0.",  
      hostname: "IAD-DCI-SPINE-01",  
      interface: "FourHundredGigE0/0/0/0",  
      lineRateGbps: 400,  
      avgFrameSizeBytes: 512,  
      cipherSuite: "GCM-AES-XPN-256",  
      isXpn: true,  
      cak: "F4A7C12B890E543D7189BA02C45DEF98A012B34C56D78E901234567890ABCDEF",  
      ckn: "0102030405060708090A0B0C0D0E0F101112131415161718191A1B1C1D1E1F20",  
      replayProtect: true,  
      replayWindow: 2048,  
      failClosed: true,  
      policyMode: "must-secure",  
      confidentialityOffset: 0,  
      basePayloadMtu: 9000,  
      interfaceMtu: 9032,  
      tcpMssClamped: true,  
      includeSci: true,  
      sci: "0011223344550001",  
      sakRekeyIntervalSec: 1800,  
      telemetryAlerting: true,  
      an: 0,  
      pn: 12450,  
      sl: 0  
    },  
    'metro-wan-financial-vlan': {  
      name: "10G Low-Latency Financial Metro WAN (Cleartext 802.1Q Transit)",  
      badge: "Financial Metro",  
      description: "Low-latency financial exchange connectivity across metro fiber. GCM-AES-256 with Confidentiality Offset 30 preserving 802.1Q VLAN tag for carrier intermediate switching, tuned 64-packet replay window, and strict fail-closed policy.",  
      hostname: "LDN-EQX-GW-01",  
      interface: "TenGigE0/0/0/1",  
      lineRateGbps: 10,  
      avgFrameSizeBytes: 256,  
      cipherSuite: "GCM-AES-256",  
      isXpn: false,  
      cak: "8A9B0C1D2E3F405162738495A6B7C8D9E0F1A2B3C4D5E6F708192A3B4C5D6E7F",  
      ckn: "A1B2C3D4E5F60718293A4B5C6D7E8F90A1B2C3D4E5F60718293A4B5C6D7E8F90",  
      replayProtect: true,  
      replayWindow: 64,  
      failClosed: true,  
      policyMode: "must-secure",  
      confidentialityOffset: 30,  
      basePayloadMtu: 1500,  
      interfaceMtu: 1536,  
      tcpMssClamped: true,  
      includeSci: true,  
      sci: "005056A1B2C30002",  
      sakRekeyIntervalSec: 1800,  
      telemetryAlerting: true,  
      an: 1,  
      pn: 450123,  
      sl: 0  
    },  
    'vulnerable-legacy-100g': {  
      name: "Vulnerable Legacy 100G Deployment (Flap & Leak Hazard)",  
      badge: "Critical Risk",  
      description: "Unsafe 100G configuration utilizing standard 32-bit PN (exhausts in 28 seconds\!), disabled replay protection, cleartext fallback (should-secure), unadjusted 1500 MTU, and no telemetry monitoring (Grade F).",  
      hostname: "LEGACY-WAN-03",  
      interface: "HundredGigE0/1/0/0",  
      lineRateGbps: 100,  
      avgFrameSizeBytes: 64,  
      cipherSuite: "GCM-AES-128",  
      isXpn: false,  
      cak: "1234567890ABCDEF1234567890ABCDEF",  
      ckn: "01020304",  
      replayProtect: false,  
      replayWindow: 0,  
      failClosed: false,  
      policyMode: "should-secure",  
      confidentialityOffset: 0,  
      basePayloadMtu: 1500,  
      interfaceMtu: 1500,  
      tcpMssClamped: false,  
      includeSci: false,  
      sci: "0000000000000000",  
      sakRekeyIntervalSec: 7200,  
      telemetryAlerting: false,  
      an: 0,  
      pn: 4294000000,  
      sl: 0  
    }  
  };

  let activeTab \= 'cisco';

  // Elements  
  const presetSelect \= document.getElementById('presetSelect');  
  const presetBadge \= document.getElementById('presetBadge');  
  const presetDesc \= document.getElementById('presetDesc');

  const cfgHostname \= document.getElementById('cfgHostname');  
  const cfgInterface \= document.getElementById('cfgInterface');  
  const cfgLineRate \= document.getElementById('cfgLineRate');  
  const cfgAvgFrameSize \= document.getElementById('cfgAvgFrameSize');  
  const cfgCipherSuite \= document.getElementById('cfgCipherSuite');  
  const cfgConfOffset \= document.getElementById('cfgConfOffset');  
  const cfgBaseMtu \= document.getElementById('cfgBaseMtu');  
  const cfgInterfaceMtu \= document.getElementById('cfgInterfaceMtu');

  const cfgCak \= document.getElementById('cfgCak');  
  const cfgCkn \= document.getElementById('cfgCkn');  
  const cfgAn \= document.getElementById('cfgAn');  
  const cfgPn \= document.getElementById('cfgPn');  
  const cfgSl \= document.getElementById('cfgSl');  
  const cfgSci \= document.getElementById('cfgSci');

  const cfgIncludeSci \= document.getElementById('cfgIncludeSci');  
  const cfgFailClosed \= document.getElementById('cfgFailClosed');  
  const cfgReplayProtect \= document.getElementById('cfgReplayProtect');  
  const cfgTcpMss \= document.getElementById('cfgTcpMss');  
  const cfgTelemetry \= document.getElementById('cfgTelemetry');  
  const cfgReplayWin \= document.getElementById('cfgReplayWin');  
  const cfgRekeySec \= document.getElementById('cfgRekeySec');

  // Outputs  
  const riskBadge \= document.getElementById('riskBadge');  
  const mLineRate \= document.getElementById('mLineRate');  
  const mPps \= document.getElementById('mPps');  
  const mPnExhaust \= document.getElementById('mPnExhaust');  
  const mReqMtu \= document.getElementById('mReqMtu');  
  const mRecommendation \= document.getElementById('mRecommendation');

  const auditScoreBadge \= document.getElementById('auditScoreBadge');  
  const auditGradeCircle \= document.getElementById('auditGradeCircle');  
  const auditTitle \= document.getElementById('auditTitle');  
  const auditSummary \= document.getElementById('auditSummary');  
  const findingsContainer \= document.getElementById('findingsContainer');

  const sectagBreakdown \= document.getElementById('sectagBreakdown');  
  const hexDumpView \= document.getElementById('hexDumpView');  
  const btnCopyHex \= document.getElementById('btnCopyHex');

  const configOutputView \= document.getElementById('configOutputView');  
  const btnCopyConfig \= document.getElementById('btnCopyConfig');  
  const tabButtons \= document.querySelectorAll('.tab-btn');

  function gatherConfig() {  
    return {  
      hostname: cfgHostname.value,  
      interface: cfgInterface.value,  
      lineRateGbps: parseFloat(cfgLineRate.value) || 100,  
      avgFrameSizeBytes: parseInt(cfgAvgFrameSize.value, 10\) || 512,  
      cipherSuite: cfgCipherSuite.value,  
      isXpn: cfgCipherSuite.value.includes('XPN'),  
      confidentialityOffset: parseInt(cfgConfOffset.value, 10\) || 0,  
      basePayloadMtu: parseInt(cfgBaseMtu.value, 10\) || 1500,  
      interfaceMtu: parseInt(cfgInterfaceMtu.value, 10\) || 1532,  
      cak: cfgCak.value.trim(),  
      ckn: cfgCkn.value.trim(),  
      an: parseInt(cfgAn.value, 10\) || 0,  
      pn: parseInt(cfgPn.value, 10\) || 1,  
      sl: parseInt(cfgSl.value, 10\) || 0,  
      sci: cfgSci.value.trim(),  
      includeSci: cfgIncludeSci.checked,  
      failClosed: cfgFailClosed.checked,  
      policyMode: cfgFailClosed.checked ? 'must-secure' : 'should-secure',  
      replayProtect: cfgReplayProtect.checked,  
      tcpMssClamped: cfgTcpMss.checked,  
      telemetryAlerting: cfgTelemetry.checked,  
      replayWindow: parseInt(cfgReplayWin.value, 10\) || 0,  
      sakRekeyIntervalSec: parseInt(cfgRekeySec.value, 10\) || 1800  
    };  
  }

  function applyPreset(presetKey) {  
    const p \= PRESETS\[presetKey\];  
    if (\!p) return;

    presetBadge.textContent \= p.badge;  
    presetDesc.textContent \= p.description;

    cfgHostname.value \= p.hostname;  
    cfgInterface.value \= p.interface;  
    cfgLineRate.value \= p.lineRateGbps;  
    cfgAvgFrameSize.value \= p.avgFrameSizeBytes;  
    cfgCipherSuite.value \= p.cipherSuite;  
    cfgConfOffset.value \= p.confidentialityOffset;  
    cfgBaseMtu.value \= p.basePayloadMtu;  
    cfgInterfaceMtu.value \= p.interfaceMtu;

    cfgCak.value \= p.cak;  
    cfgCkn.value \= p.ckn;  
    cfgAn.value \= p.an;  
    cfgPn.value \= p.pn;  
    cfgSl.value \= p.sl;  
    cfgSci.value \= p.sci;

    cfgIncludeSci.checked \= p.includeSci;  
    cfgFailClosed.checked \= p.failClosed;  
    cfgReplayProtect.checked \= p.replayProtect;  
    cfgTcpMss.checked \= p.tcpMssClamped;  
    cfgTelemetry.checked \= p.telemetryAlerting;  
    cfgReplayWin.value \= p.replayWindow;  
    cfgRekeySec.value \= p.sakRekeyIntervalSec;

    updateUI();  
  }

  function updateUI() {  
    const cfg \= gatherConfig();

    // 1\. Calculate PN Exhaustion & MTU Metrics  
    const pnMetrics \= MACsecProtocolEngine.calculatePNDuration({  
      lineRateGbps: cfg.lineRateGbps,  
      avgFrameSizeBytes: cfg.avgFrameSizeBytes,  
      cipherSuite: cfg.cipherSuite,  
      isXpn: cfg.isXpn  
    });

    const mtuMetrics \= MACsecProtocolEngine.calculateMTUHeadroom({  
      basePayloadMtu: cfg.basePayloadMtu,  
      includeSci: cfg.includeSci,  
      vlanCount: cfg.confidentialityOffset \> 0 ? 1 : 0,  
      confidentialityOffset: cfg.confidentialityOffset  
    });

    mLineRate.textContent \= \`\${cfg.lineRateGbps}G\`;  
    mPps.textContent \= \`\${(pnMetrics.maxPacketsPerSec / 1e6).toFixed(1)} M\`;  
    mPnExhaust.textContent \= pnMetrics.timeFormatted;  
    mReqMtu.textContent \= \`\${mtuMetrics.requiredInterfaceL2Mtu} B\`;  
    mRecommendation.textContent \= pnMetrics.recommendation;

    riskBadge.textContent \= pnMetrics.riskLevel;  
    if (pnMetrics.riskLevel \=== 'CRITICAL') {  
      riskBadge.className \= 'badge badge-red';  
      mPnExhaust.className \= 'metric-val alert-red';  
    } else if (pnMetrics.riskLevel \=== 'HIGH' || pnMetrics.riskLevel \=== 'MODERATE') {  
      riskBadge.className \= 'badge badge-amber';  
      mPnExhaust.className \= 'metric-val alert-amber';  
    } else {  
      riskBadge.className \= 'badge badge-emerald';  
      mPnExhaust.className \= 'metric-val';  
    }

    // 2\. Encode SecTAG & Generate Hex Dump  
    const secTagBuf \= MACsecProtocolEngine.encodeSecTAG({  
      v: 0,  
      es: 0,  
      sc: cfg.includeSci ? 1 : 0,  
      scb: 0,  
      e: 1,  
      c: 1,  
      an: cfg.an,  
      sl: cfg.sl,  
      pn: cfg.pn,  
      sci: cfg.sci  
    });

    // Create a mock Ethernet frame containing DMAC, SMAC, SecTAG, Encrypted Data, and ICV  
    const frameLen \= 12 \+ secTagBuf.length \+ 32 \+ 16;  
    const frameBuf \= new Uint8Array(frameLen);  
    // DMAC  
    frameBuf.set(\[0x00, 0x11, 0x22, 0x33, 0x44, 0x55\], 0);  
    // SMAC  
    frameBuf.set(\[0x00, 0xaa, 0xbb, 0xcc, 0xdd, 0xee\], 6);  
    // SecTAG  
    frameBuf.set(secTagBuf, 12);  
    // Mock Encrypted Payload  
    for (let i \= 0; i \< 32; i++) {  
      frameBuf\[12 \+ secTagBuf.length \+ i\] \= (i \* 7 \+ 0x41) & 0xFF;  
    }  
    // Mock ICV (16B AES-GCM Tag)  
    for (let i \= 0; i \< 16; i++) {  
      frameBuf\[frameLen \- 16 \+ i\] \= (i \* 13 \+ 0x89) & 0xFF;  
    }

    hexDumpView.textContent \= MACsecProtocolEngine.formatHexDump(frameBuf);

    // Render SecTAG Byte Breakdown Visualizer  
    sectagBreakdown.innerHTML \= '';  
    const bbItems \= \[  
      { hex: '88 E5', lbl: 'EtherType (0x88E5)', cls: 'bb-ethertype' },  
      { hex: secTagBuf\[2\].toString(16).padStart(2, '0').toUpperCase(), lbl: \`TCI/AN (AN=\${cfg.an})\`, cls: 'bb-tci' },  
      { hex: secTagBuf\[3\].toString(16).padStart(2, '0').toUpperCase(), lbl: \`SL (\${cfg.sl})\`, cls: 'bb-sl' },  
      { hex: \`\${secTagBuf\[4\].toString(16).padStart(2, '0')} \${secTagBuf\[5\].toString(16).padStart(2, '0')} \${secTagBuf\[6\].toString(16).padStart(2, '0')} \${secTagBuf\[7\].toString(16).padStart(2, '0')}\`.toUpperCase(), lbl: \`PN (\${cfg.pn})\`, cls: 'bb-pn' }  
    \];  
    if (cfg.includeSci) {  
      const sciBytes \= \[\];  
      for (let i \= 8; i \< 16; i++) {  
        sciBytes.push(secTagBuf\[i\].toString(16).padStart(2, '0'));  
      }  
      bbItems.push({ hex: sciBytes.join(' ').toUpperCase(), lbl: 'SCI (8 Bytes)', cls: 'bb-sci' });  
    }

    bbItems.forEach(item \=\> {  
      const div \= document.createElement('div');  
      div.className \= \`byte-block \${item.cls}\`;  
      div.innerHTML \= \`\<div class="byte-hex"\>\${item.hex}\</div\>\<div class="byte-lbl"\>\${item.lbl}\</div\>\`;  
      sectagBreakdown.appendChild(div);  
    });

    // 3\. Run 10-Point Zero-Trust Audit  
    const auditRes \= MACsecSecurityAuditor.auditMACsecConfiguration(cfg);  
    auditScoreBadge.textContent \= \`\${auditRes.score} / 100\`;  
    auditGradeCircle.textContent \= auditRes.grade;  
    auditGradeCircle.className \= \`grade-badge-circle grade-\${auditRes.grade.toLowerCase().replace('+', 'plus')}\`;  
    auditTitle.textContent \= auditRes.summary;  
    auditSummary.textContent \= \`Zero-Trust Evaluation: \${auditRes.percentage}% compliance score across 10 security pillars.\`;

    findingsContainer.innerHTML \= '';  
    auditRes.findings.forEach(f \=\> {  
      const div \= document.createElement('div');  
      div.className \= 'finding-item';

      const ptsClass \= f.status \=== 'PASS' ? 'pts-pass' : (f.status \=== 'WARN' ? 'pts-warn' : 'pts-crit');  
      let html \= \`  
        \<div class="finding-top"\>  
          \<span class="finding-name"\>\${f.name}\</span\>  
          \<span class="finding-pts \${ptsClass}"\>\${f.score} / \${f.maxScore} pts \[\${f.status}\]\</span\>  
        \</div\>  
        \<div class="finding-desc"\>\${f.description}\</div\>  
      \`;  
      if (f.remediation) {  
        html \+= \`\<div class="finding-remed"\>\<strong\>Remediation:\</strong\> \${f.remediation}\</div\>\`;  
      }  
      div.innerHTML \= html;  
      findingsContainer.appendChild(div);  
    });

    // 4\. Synthesize Multi-Vendor Configuration  
    renderConfig(cfg);  
  }

  function renderConfig(cfg) {  
    if (\!cfg) cfg \= gatherConfig();  
    let text \= '';  
    if (activeTab \=== 'cisco') {  
      text \= MACsecConfigSynthesizer.generateCiscoIOSXR(cfg);  
    } else if (activeTab \=== 'arista') {  
      text \= MACsecConfigSynthesizer.generateAristaEOS(cfg);  
    } else if (activeTab \=== 'juniper') {  
      text \= MACsecConfigSynthesizer.generateJuniperJunos(cfg);  
    } else if (activeTab \=== 'linux') {  
      text \= MACsecConfigSynthesizer.generateLinuxMACsec(cfg);  
    } else if (activeTab \=== 'prometheus') {  
      text \= MACsecConfigSynthesizer.generatePrometheusRules(cfg);  
    }  
    configOutputView.textContent \= text;  
  }

  // Event Listeners  
  presetSelect.addEventListener('change', (e) \=\> applyPreset(e.target.value));

  const allInputs \= \[  
    cfgHostname, cfgInterface, cfgLineRate, cfgAvgFrameSize, cfgCipherSuite,  
    cfgConfOffset, cfgBaseMtu, cfgInterfaceMtu, cfgCak, cfgCkn, cfgAn, cfgPn,  
    cfgSl, cfgSci, cfgIncludeSci, cfgFailClosed, cfgReplayProtect, cfgTcpMss,  
    cfgTelemetry, cfgReplayWin, cfgRekeySec  
  \];  
  allInputs.forEach(input \=\> {  
    input.addEventListener('input', updateUI);  
    input.addEventListener('change', updateUI);  
  });

  tabButtons.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      tabButtons.forEach(b \=\> b.classList.remove('active'));  
      btn.classList.add('active');  
      activeTab \= btn.getAttribute('data-tab');  
      renderConfig();  
    });  
  });

  btnCopyHex.addEventListener('click', () \=\> {  
    navigator.clipboard.writeText(hexDumpView.textContent).then(() \=\> {  
      btnCopyHex.textContent \= 'Copied\!';  
      setTimeout(() \=\> btnCopyHex.textContent \= 'Copy Hex', 2000);  
    });  
  });

  btnCopyConfig.addEventListener('click', () \=\> {  
    navigator.clipboard.writeText(configOutputView.textContent).then(() \=\> {  
      btnCopyConfig.textContent \= 'Copied\!';  
      setTimeout(() \=\> btnCopyConfig.textContent \= 'Copy Config', 2000);  
    });  
  });

  // Initialize with Default Preset  
  applyPreset('dci-400g-xpn-cloud');  
});  
