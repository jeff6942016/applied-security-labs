# Detection Engineering and Purple Team

![Focus](https://img.shields.io/badge/Focus-Detection%20Engineering%20%26%20Purple%20Team-1F3864)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh%20%7C%20Suricata%20%7C%20Sysmon-2E7D32)
![Framework](https://img.shields.io/badge/Framework-MITRE%20ATT%26CK%20%7C%20D3FEND%20%7C%20CIS%20Top%2018-455A64)
![Course](https://img.shields.io/badge/Course-SPR708%20%2F%20SPR600-6A1B9A)

A purple-team project that builds an instrumented, segmented enterprise environment, emulates a defined threat actor against it, and engineers detections for every stage of the attack. The point is not to attack or defend in isolation but to close the loop: run the adversary's real techniques, see what the sensors catch and miss, then improve the coverage.

## Overview

The problem this addresses is the gap between "we have a SIEM" and "we can actually detect an attack." A monitoring stack that has never been tested against real adversary behavior gives a false sense of security. This project takes the opposite approach. A threat actor is chosen first, its tactics, techniques, and procedures (TTPs) are mapped to MITRE ATT&CK, and the environment is then attacked along exactly those TTPs so each detection can be proven against genuine malicious traffic rather than assumed.

The environment was built from scratch on isolated VirtualBox and Docker infrastructure: an Active Directory domain, supporting network services, a routed topology with Suricata inspecting traffic, and Wazuh as the central SIEM correlating endpoint and network events. Hardening followed the CIS Top 18 Controls and was validated with functional tests rather than treated as a checklist. A separate component, a safe ransomware simulator, was built to understand encryption-and-extortion behavior from the attacker side so it can be recognized on the defender side.

### Why a self-hosted Wazuh and Suricata stack, not a managed cloud SIEM

| Consideration | Self-hosted Wazuh + Suricata (chosen) | Managed cloud SIEM |
|---------------|---------------------------------------|--------------------|
| Learning value | Every log source, agent, and rule is configured by hand and fully understood | Much of the pipeline is abstracted away |
| Detection authoring | Full control to write and tune custom Suricata and Sysmon rules per ATT&CK technique | Often limited to vendor content packs |
| Cost | Free and open source, runs on lab hardware | Ingestion-based billing |
| Realism for the exercise | Mirrors an on-prem enterprise, which is where AD and SMB attacks like this occur | Better suited to cloud-native telemetry |

The tradeoff is that a self-hosted stack takes more effort to stand up and has no vendor support, which is acceptable and even desirable for a learning environment.

## Architecture

```mermaid
flowchart LR
    subgraph Attacker
        K[Kali Linux]
    end
    subgraph Enterprise LAN
        V[WIN-VICTIM<br/>domain workstation]
        AD[Active Directory<br/>Domain Controller]
        SVC[Services VM<br/>ERP / web / file]
    end
    subgraph Monitoring
        SUR[Suricata IDS<br/>on router]
        WZ[Wazuh SIEM<br/>central correlation]
        SYS[Sysmon<br/>host telemetry]
    end

    K -->|attack traffic| V
    V --> AD
    V --> SVC
    V -. routed traffic .-> SUR
    V -. host events .-> SYS
    SUR --> WZ
    SYS --> WZ
```

## Key Concepts Demonstrated

- Intelligence-led emulation: selecting a threat actor and driving the whole exercise from its known TTPs, so testing reflects real risk instead of a random tool sweep
- MITRE ATT&CK and D3FEND mapping: every attack technique is tied to an ATT&CK ID and every defensive countermeasure to a D3FEND tactic
- Detection engineering: authoring and tuning custom Suricata and Sysmon rules, then validating them against live attack traffic
- CIS Top 18 Controls: applying and functionally testing hardening controls rather than assuming they work
- Defense in depth: layering network segmentation, least-privilege group policy, MFA, and monitoring so that a single failure is not fatal
- Attacker behavior modeling: building a controlled ransomware simulator to understand encryption, C2, and double-extortion mechanics from the inside

## Skills Demonstrated

- SIEM and detection: Wazuh, Suricata, Sysmon, custom rule authoring, alert correlation and tuning with Atomic Red Team
- Adversary emulation: multi-phase attack chains mapped to ATT&CK, executed against a live environment
- Hardening and governance: CIS Top 18 Controls, Group Policy, password and lockout policy, MFA, PKI
- Secure development: a standard-library-only Python ransomware simulator with strict lab-isolation safety controls
- Infrastructure: Active Directory, VirtualBox and Docker lab design, routed topology, IDS placement

## Walkthrough

### 1. Prepare: threat modeling and the PEIR process

The exercise followed a Prepare, Execute, Identify, Remediate (PEIR) process. The Prepare phase established the threat actor, mapped its TTPs to ATT&CK, defined the infrastructure scope, and assessed the target environment's maturity so the exercise had a realistic baseline. As CTI lead, this threat-actor selection and TTP mapping was the part I owned, and it set the plan every later phase followed.

<details>
<summary>PEIR process evidence</summary>

![PEIR process](spr708-lab1-peir/spr708-lab1-peir-001.png)
![PEIR process](spr708-lab1-peir/spr708-lab1-peir-002.png)
![PEIR process](spr708-lab1-peir/spr708-lab1-peir-003.png)
![PEIR process](spr708-lab1-peir/spr708-lab1-peir-004.png)

</details>

### 2. Harden: secure configuration to the CIS Top 18

Before attacking, the environment was hardened and the controls were documented and functionally tested: disabled services, enforced password and lockout policies, restrictive Group Policy Objects, MFA, firewall rules, an access-control matrix, and a VPN for secure remote access. This gives the exercise a defensible baseline to attack, so any successful technique reflects a real residual risk rather than a trivially open box.

<details>
<summary>Hardening evidence (GPOs, password policy, MFA, firewall)</summary>

![Hardening](spr708-lab2-hardening/spr708-lab2-hardening-001.png)
![Hardening](spr708-lab2-hardening/spr708-lab2-hardening-002.png)
![Hardening](spr708-lab2-hardening/spr708-lab2-hardening-003.png)
![Hardening](spr708-lab2-hardening/spr708-lab2-hardening-004.png)
![Hardening](spr708-lab2-hardening/spr708-lab2-hardening-005.png)

</details>

### 3. Instrument: identity controls and monitoring

Identity controls (account management, least privilege, MFA) were extended with a logging and monitoring architecture. Wazuh collected host and Suricata events centrally, Sysmon added detailed host telemetry, and detection rules were written for brute force, PowerShell abuse, and suspicious logins, with network-layer Suricata rules alongside them. This is the sensor coverage the attack phase would later test.

<details>
<summary>Monitoring and detection-rule evidence</summary>

![Monitoring](spr708-lab3-identity-monitoring/spr708-lab3-identity-monitoring-001.png)
![Monitoring](spr708-lab3-identity-monitoring/spr708-lab3-identity-monitoring-002.png)
![Monitoring](spr708-lab3-identity-monitoring/spr708-lab3-identity-monitoring-003.png)
![Monitoring](spr708-lab3-identity-monitoring/spr708-lab3-identity-monitoring-004.png)
![Monitoring](spr708-lab3-identity-monitoring/spr708-lab3-identity-monitoring-005.png)

</details>

### 4. CIS Top 18 monitoring implementation

A dedicated build mapped each CIS Control to a concrete monitoring implementation in Wazuh, with evidence of operational effectiveness and an APT-style emulation (APT29) to validate detection efficiency.

<details>
<summary>CIS Controls monitoring evidence</summary>

![CIS monitoring](spr600-cis-monitoring/spr600-cis-monitoring-001.png)

</details>

### 5. Execute: multi-phase adversary emulation

The attack was run in six phases against the instrumented environment, each phase mapped to ATT&CK and each generating evidence that was checked against the detection stack: initial access and staging, remote command execution, persistence, discovery and lateral movement, C2 callback, and finally detection validation. The blue-team analysis then documented what was detected, what was missed, and the improvement recommendations, which is the core purple-team output.

<details>
<summary>Attack phases and detection-validation evidence</summary>

![Attack simulation](spr708-lab4-attack-simulation/spr708-lab4-attack-simulation-001.png)
![Attack simulation](spr708-lab4-attack-simulation/spr708-lab4-attack-simulation-002.png)
![Attack simulation](spr708-lab4-attack-simulation/spr708-lab4-attack-simulation-005.png)
![Attack simulation](spr708-lab4-attack-simulation/spr708-lab4-attack-simulation-010.png)
![Attack simulation](spr708-lab4-attack-simulation/spr708-lab4-attack-simulation-020.png)
![Attack simulation](spr708-lab4-attack-simulation/spr708-lab4-attack-simulation-030.png)

The full set of 52 phase-by-phase screenshots is in the [attack-simulation folder](./spr708-lab4-attack-simulation).

</details>

### 6. Understand the payload: safe ransomware simulator

To understand ransomware behavior from the attacker side, the team built a controlled ransomware simulator in Python using the standard library only. It executes a full double-extortion workflow (file encryption, exfiltration, C2 communication, persistence, and payment-triggered decryption) entirely inside an isolated lab, gated behind mandatory safety checks (a lab marker file, an environment variable, and a confined target directory). Testing encrypted and decrypted 35 files with zero data loss. Understanding this workflow directly informs what to detect: the file-access patterns, the C2 beacon, and the persistence it drops.

<details>
<summary>Ransomware simulator evidence (C2 dashboard, encryption, decryption)</summary>

![Ransomware simulator](spr708-assignment2/spr708-assignment2-002.png)
![Ransomware simulator](spr708-assignment2/spr708-assignment2-003.png)
![Ransomware simulator](spr708-assignment2/spr708-assignment2-005.png)
![Ransomware simulator](spr708-assignment2/spr708-assignment2-007.png)
![Ransomware simulator](spr708-assignment2/spr708-assignment2-008.png)

</details>

## Team and My Role

This was a Group 11 project (SPR708), with the CIS Controls monitoring build from SPR600. Roles were divided as coordinator (Evan Bettencourt), red team lead (Easton Soares), blue team lead (Jacob Williams), and CTI lead (myself). My work centered on the threat intelligence that drove the exercise: selecting the threat actor, mapping its TTPs to MITRE ATT&CK, and writing the reporting that tied attack evidence to detections. I worked closely enough with the red-team execution and the blue-team detection engineering to explain both, which is why the walkthrough above covers the whole chain rather than only my section.

## Lessons Learned

- Detection is only real once it is tested: several rules that looked correct on paper only proved themselves (or revealed gaps) when run against live attack traffic
- The biggest gaps are usually visibility, not tooling: missed techniques were more often a logging or tuning problem than a missing product
- Mapping to ATT&CK first makes the whole exercise coherent: it turns "try some attacks" into a targeted test of specific, documented adversary behavior
- Building the attacker's payload teaches the defender's job: implementing the ransomware simulator made its detectable artifacts obvious in a way that reading about ransomware never did
