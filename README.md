# SOC L1 Detection & Incident Response Lab

A home-lab Security Operations Center project covering attack simulation, log ingestion, detection engineering, SOAR automation, and AI-assisted triage — built end-to-end on a 3-machine lab synced into Microsoft Sentinel + Defender XDR.

---

## Architecture

```
                    ┌─────────────────────────┐
                    │   Microsoft Sentinel       │
                    │   (Log Analytics: SOCLAb)  │
                    │   + Microsoft Defender XDR │
                    └────────────┬────────────┘
                                 │
                 Custom log table: AttackLogs_CL
                 (ingested via Python / Logs Ingestion API)
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                         │
 Windows Server 2025      Ubuntu 24.04 LTS              Kali Linux
 192.168.189.140 (DC01)   192.168.189.141               192.168.189.129
 Domain: SOCLAB.COM       SSH target                     Attacker machine
        │
 Microsoft Entra ID (Azure AD)
 synced via Azure AD Connect
 (hybrid identity — on-prem AD ↔ cloud)
```

**Note on scope:** the Sentinel/Log Analytics agent is connected to Microsoft Entra ID (for identity sync and RBAC), not installed as a monitoring agent directly on the VMs. Logs from the VMs were collected locally (Sysmon, Windows Event Log, `auth.log`, IIS logs) and ingested into Sentinel's custom `AttackLogs_CL` table via a Python ingestion pipeline, which reflects how many real environments centralize logs from systems that can't run a native agent.

---

## Attacks Simulated

**Windows Server (DC01):**
| # | Attack | MITRE ATT&CK |
|---|---|---|
| 1 | RDP Brute Force | T1110 |
| 2 | Phishing — Credential Harvesting (fake Microsoft 365 login) | T1566, T1598 |
| 3 | New Admin / Backdoor Account Created | T1136, T1098, T1068 |
| 4 | Malicious File Download (`do.exe`, `Analysis (1).exe`) | T1204.002, T1105 |
| 5 | C2 Beaconing (HTTP-based, randomized URIs, session cookie) | T1071.001, T1571 |
| 6 | Log Clearing (`wevtutil`, EventCode 1102) | T1070.001 |

**Ubuntu:**
| # | Attack | MITRE ATT&CK |
|---|---|---|
| 7 | SSH Brute Force | T1110 |
| 8 | Malicious Command Execution | T1059 |
| 9 | Log Clearing (`auth.log`) | T1070.002 |
| 10 | Data Exfiltration (`/etc/shadow` access + SFTP transfer) | T1003, T1041 |

Each attack has its own folder with KQL detection queries, evidence screenshots, and the full incident response write-up (triage, classification, IOCs, recommendation, escalation).

---

## SOAR Automation

- **Ticket creation:** analytics rules are wired to a ServiceNow ticket-creation playbook (Automated response → Run playbook), triggered on incident creation.
- **Host isolation/un-isolation:** a Logic App was designed with a manual-trigger "Isolate / Un-isolate" workflow (branches on input action → create/delete NSG deny-all rule), scoped to avoid auto-isolating on every alert — a deliberate decision, since automatic isolation on every alert risks isolating on false positives. Analyst-triggered isolation was chosen instead.

---

## AI-Assisted Triage (GitHub Copilot + MCP)

GitHub Copilot was connected to the Sentinel workspace via an MCP (Model Context Protocol) server, allowing natural-language incident triage directly from the editor:

> **Prompt:** "List open incidents using sentinel-triage MCP"
> **Prompt:** "I want the evidence of incident 14 and tell me if this is true positive or false positive"

Copilot queried the workspace and returned a structured response for the SSH Brute Force incident — evidence (99 failed attempts from the attacker IP in a 5-minute window), MITRE mapping, and an assessment flagging that compromise was not yet confirmed. This prompted a manual follow-up hunt that found a **successful** SSH login from the same attacker IP shortly after, which changed the triage conclusion.

This reflects how AI-assisted triage realistically works in modern SOCs: it accelerates evidence-gathering and flags next investigative steps, but the analyst still owns the final determination.

---

## Known Limitations (documented honestly, not hidden)

- Ticket automation was fully wired and tested for a subset of incidents; for the remainder, evidence collection and IR documentation are complete, but ticket automation wiring was deprioritized under time constraints — a known next step.
- The Logic App isolate/un-isolate workflow was fully designed and partially tested; full end-to-end testing was paused to conserve Azure subscription credits.
- Phishing email delivery (SMTP) was not captured in packet data — only the resulting web-based credential harvesting (page load → form submission) was captured and analyzed. This is noted explicitly in that incident's evidence section rather than implied.

---

## Repo Layout

```SOC Analyst

```Ubuntu Attack

---Windows server or Windows attack

## Tools & Skills Demonstrated

Microsoft Sentinel · Microsoft Defender XDR · KQL (Kusto Query Language) · Microsoft Entra ID (Azure AD) · Azure AD Connect (hybrid identity) · Logic Apps (SOAR) · ServiceNow ticket automation · GitHub Copilot + MCP (AI-assisted triage) · Sysmon · Wireshark/Scapy (pcap analysis) · MITRE ATT&CK mapping · Incident Response documentation
