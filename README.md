<div align="center">

# 🔍 SIEM Implementation — Brute Force & FIM Detection

**Wazuh-based SOC lab: RDP brute-force simulation, File Integrity Monitoring, and custom detection engineering**

*Author: João Victor Campbell Fontenele*

![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-blue?style=flat-square)
![MITRE](https://img.shields.io/badge/Framework-MITRE%20ATT%26CK%20T1110-red?style=flat-square)
![Ubuntu](https://img.shields.io/badge/Server-Ubuntu-E95420?style=flat-square)
![Windows](https://img.shields.io/badge/Agent-Windows%2011-0078D4?style=flat-square)
![Kali](https://img.shields.io/badge/Attacker-Kali%20Linux-557C94?style=flat-square)

</div>

---

## Overview

This project implements and validates a Wazuh SIEM solution in a controlled lab environment, focused on detecting and correlating security events related to RDP brute-force attacks and File Integrity Monitoring (FIM).

The scenario simulates a realistic attack against the RDP service while evaluating the SIEM's ability to:

- Centralize and normalize logs from heterogeneous sources
- Identify malicious behavioral patterns
- Correlate multiple events within short time windows
- Escalate criticality based on frequency and context
- Generate actionable alerts for SOC workflows

A custom detection rule was developed to demonstrate detection engineering and threshold tuning for environment-specific needs.

---

## Architecture

| Role | System | Function |
|------|--------|----------|
| 📊 SIEM (All-in-One) | Ubuntu Server | Wazuh manager, indexer, and dashboard |
| 🪟 Target + Agent | Windows 11 | RDP service, Wazuh agent, FIM monitoring |
| ⚔️ Attacker | Kali Linux | Offensive tooling (Hydra, Nmap) |

---

## Project Scope

### 1. Infrastructure Deployment
Wazuh was deployed in All-in-One architecture on Ubuntu Server, consolidating the manager, indexer, and dashboard into a single node — appropriate for lab-scale validation.

### 2. Windows Agent Integration
A Wazuh agent was installed on the Windows 11 target to forward Windows Security Event Logs to the SIEM, enabling centralized visibility over authentication events (Event ID 4625 — failed logon).

### 3. File Integrity Monitoring (FIM)
FIM was configured to monitor critical directories on the Windows host. Any unauthorized file creation, modification, or deletion triggers a dedicated alert, validating the SIEM's capability beyond network-layer detection.

### 4. RDP Brute-Force Simulation

````bash
hydra -l administrator -P rockyou.txt rdp://TARGET_IP
````

Hydra was used to simulate a credential-stuffing attack against the RDP service (TCP/3389). The volume of failed authentications generated observable patterns in the Windows event log, collected in real time by the Wazuh agent.

### 5. Custom Correlation Rule

A custom rule was developed in `local_rules.xml` to detect the RDP brute-force pattern:

````xml
<!-- Rule 100300: Single RDP login failure -->
<rule id="100300" level="5">
  <if_sid>60122</if_sid>
  <description>RDP authentication failure detected.</description>
  <mitre><id>T1110</id></mitre>
</rule>

<!-- Rule 100301: RDP brute force pattern (8 failures in 60s) -->
<rule id="100301" level="12" frequency="8" timeframe="60">
  <if_matched_sid>100300</if_matched_sid>
  <description>RDP brute force attack detected — critical alert.</description>
  <mitre><id>T1110</id></mitre>
</rule>
````

Threshold tuning reduced false positives while maintaining high sensitivity to genuine attack patterns.

### 6. Validation

Detection efficacy was confirmed end-to-end: attack execution → log collection → rule triggering → critical alert in the Wazuh dashboard.

---

## Results

| Capability | Outcome |
|-----------|---------|
| Log centralization | Windows Security Events forwarded in real time |
| Brute-force detection | Rule 100301 triggered within seconds of attack onset |
| FIM | File tampering alerts generated on target host |
| MITRE mapping | T1110 (Brute Force) correctly attributed |
| False positive rate | Reduced through custom threshold tuning |
| SOC readiness | Actionable critical alerts with full event context |

---

## Key Concepts Demonstrated

- **MTTD reduction** — threats detected in near-real time rather than post-incident
- **Detection engineering** — custom rule development and threshold calibration
- **Defense in depth** — network-layer (RDP) and host-layer (FIM) monitoring in parallel
- **Incident prioritization** — severity escalation based on event frequency and context
- **SOC workflows** — alert triage and correlation in a centralized dashboard

---

## Tech Stack

- [Wazuh](https://wazuh.com/) — open-source SIEM & XDR platform
- [Hydra](https://github.com/vanhauser-thc/thc-hydra) — network login brute-forcer
- [Nmap](https://nmap.org/) — network discovery and service enumeration
- [MITRE ATT&CK](https://attack.mitre.org/techniques/T1110/) — adversary tactics framework
- Ubuntu Server, Windows 11, Kali Linux

---

## Disclaimer

> This lab was conducted in an isolated private network for educational purposes only. All tests were performed against systems owned and controlled by the author. Do not replicate these techniques against systems without explicit written authorization.

````

---

Algumas sugestões para elevar ainda mais o perfil:

**Adicionar screenshots** do dashboard do Wazuh mostrando o alerta de nível 12 sendo gerado — nada convence mais um recrutador do que ver o alerta real na tela.

**Criar uma pasta `/rules`** no repositório com o `local_rules.xml` completo — isso mostra que você fez detection engineering de verdade, não só seguiu um tutorial.

**Linkear os dois projetos entre si** — no README do SSH lab mencionar este projeto como um segundo módulo, e vice-versa. Cria a sensação de um portfólio coeso e crescente.
````
