# 🛡️ Security Engineering & SIEM Detection Lab (Wazuh + Sysmon + Atomic Red Team)

## 📌 Project Overview
This project demonstrates the deployment and configuration of a fully functional Security Operations Center (SOC) home laboratory for log centralization, endpoint monitoring, and threat detection. The primary objective was to establish deep visibility over a physical Windows 11 endpoint, forward advanced telemetry to a centralized Linux-based SIEM manager (Wazuh), and validate detection capabilities using real-world adversary simulation techniques aligned with the MITRE ATT&CK framework.

To optimize computing resources and prevent performance bottlenecks, the architecture integrates a live physical machine using strictly controlled exclusion policies within Windows Defender.

---

## 🏗️ Lab Architecture & Environment

## 🏗️ Lab Architecture & Environment

* **SIEM / Centralized Manager:** [Wazuh Server v4.14.5](https://documentation.wazuh.com/current/quickstart.html) hosted on a Linux Virtual Machine (VMware).
* **Monitored Endpoint:** Physical Windows 11 Desktop monitored via the [Wazuh Agent](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html) + [Microsoft Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon).
* **Adversary Emulation Engine:** Red Canary's [Invoke-AtomicRedTeam](https://github.com/redcanaryco/invoke-atomicredteam) framework.
---

## 🛠️ Step-by-Step Implementation

### 1. Wazuh Agent Deployment & Endpoint Enrollment
* Generated a customized `.msi` agent deployment script from the Wazuh Manager dashboard pointing to the server's static IP address.
* Executed a silent deployment via an administrative PowerShell terminal on the Windows 11 host.
* Verified successful enrollment and authenticated handshakes; the host status shifted to `Active` on the centralized SIEM console.

### 2. Advanced Endpoint Telemetry with Microsoft Sysmon
Standard operating system logs often lack the granularity required for advanced threat hunting. To capture process trees, network connections, and registry modifications, **Sysmon v15.20** was deployed utilizing *SwiftOnSecurity's* production-hardened configuration template.

* **Installation Command:**
    ```powershell
    .\Sysmon64.exe -i sysmonconfig.xml -accepteula
    ```
* **SIEM Log Pipeline Integration (`ossec.conf`):** The local Wazuh agent configuration was modified to ingest and parse Sysmon's dedicated Windows Event Channel stream:
    ```xml
    <localfile>
      <location>Microsoft-Windows-Sysmon/Operational</location>
      <log_format>eventchannel</log_format>
    </localfile>
    ```
* Restarted the Wazuh service (`Restart-Service -Name Wazuh`) to apply the configuration change and establish the new data pipeline.

### 3. Adversary Emulation Framework Setup
* Configured a strict directory exclusion rule within Windows Defender (`C:\Wazuh-Testes`) to create a safe zone for hosting and executing attack binaries without global antivirus interference.
* Installed the `Invoke-AtomicRedTeam` PowerShell module directly to the system and automated the download of the entire open-source library of curated atomic test scripts.

---

## 🎯 Adversary Simulation & Threat Hunting

With telemetry pipelines fully active, multiple security test cases were executed to evaluate the SIEM’s parsing and alerting logic.

### Test Case 1: Suspicious PowerShell Execution (MITRE ATT&CK T1059.001)
* **Objective:** Emulate an attacker attempting to leverage obfuscated PowerShell commands to retrieve or execute malicious tooling (specifically mimicking behaviors associated with *Mimikatz* invocation).
* **Execution:** `Invoke-AtomicTest T1059.001 -TestNumbers 1`

### Test Case 2: Malicious Registry Modification (MITRE ATT&CK T1112)
* **Objective:** Simulate defensive evasion techniques where malware alters system registry configurations to hide known file extensions (e.g., trying to disguise a `.exe` payload as a `.pdf`).
* **Execution:** `Invoke-AtomicTest T1112 -TestNumbers 1`

---

## 📊 SIEM Analysis & Detection Results

Immediately following the adversary simulation, the **Wazuh (Explore/Discover)** engine ingested a massive spike of incoming security events from the endpoint. Triaging the ingested raw JSON payloads validated the precision of the telemetry chain:

1.  **Registry Tampering Visibility:** The system captured the exact execution of `reg.exe` attempting to write to the path:
    `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced /v HideFileExt /d 1 /f`
2.  **Process Tree Analysis:** The events clearly mapped the lineage of the threat, exposing the parent process relationship where `cmd.exe` directly spawned anomalous background `powershell.exe` payloads.

<img width="1897" height="893" alt="Screenshot 2026-06-03 153451" src="https://github.com/user-attachments/assets/e5268b12-ba30-474f-80da-b71c3486dc44" />


---

## 💡 Core Competencies Demonstrated
* **SIEM/XDR Engineering:** End-to-end agent enrollment, manager synchronization, and log source orchestration.
* **Log Pipeline Architecture:** Parsing structured Windows Event Logs and configuring custom system event collection channels.
* **Purple Team Methodologies:** Bridging the gap between Red Team simulation frameworks and Blue Team ingestion rules.
* **Incident Triaging:** Reviewing forensic markers, isolating parent/child process relationships, and mapping logs to the MITRE ATT&CK framework.
