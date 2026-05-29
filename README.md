# AD Lab Setup — MARVEL.local

Personal Active Directory lab built following TCM Security — Practical Ethical Hacking.
Used for practicing AD attack chains in preparation for OSCP.

## Lab Architecture

![Lab topology](assets/lab-topology.png)

## Machines

| Machine | OS | Role | IP |
|---|---|---|---|
| TONYSTARK | Windows Server 2019 | Domain Controller + ADCS | 192.168.x.x |
| Workstation-1 | Windows 10 Enterprise | Domain joined | 192.168.x.x |
| Workstation-2 | Windows 10 Enterprise | Domain joined | 192.168.x.x |
| Kali | Kali Linux | Attacker | 192.168.x.x |

## Domain: MARVEL.local

### Users

| Username | Type | Notes |
|---|---|---|
| Administrator | DA | Default |
| fcastle | DA (duplicate) | Test DA |
| pparker | Domain User | Standard user |
| SQLService | Service Account | SPN: `MARVEL/SQLService.MARVEL.local:60111`, password in description |

### Shares
- `\\TONYSTARK\hackme` — shared folder for LNK file attack demos

### Group Policy
- **Disable Windows Defender** — applied to entire domain (Enforced)

![GPO config](assets/gpo-defender-off.png)

## Services configured
- **ADCS** — Certificate Authority (Certification Authority role)
- **DNS** — DC as DNS server for all workstations
- **SMB signing** — disabled (enables SMB Relay)

![ADCS setup](assets/adcs-config.png)

## Attacks demonstrated

| Attack | Tool | Notes |
|---|---|---|
| LLMNR Poisoning | Responder | Capture NTLMv2 hashes |
| SMB Relay | ntlmrelayx | Requires SMB signing disabled |
| IPv6 MITM | mitm6 + ntlmrelayx | Run max 10 min |
| Kerberoasting | GetUserSPNs.py | SQLService SPN |
| Token Impersonation | Metasploit incognito | Requires DA session |
| LNK File Attack | PowerShell + Responder | Via hackme share |
| GPP / cPasswords | gpp-decrypt | Old SYSVOL XML |
| Pass the Hash | psexec.py / NXC | After credential dump |
| Golden Ticket | Mimikatz | After NTDS.dit dump |
| ZeroLogon (demo) | CVE-2020-1472 | DO NOT run in production |
| PrintNightmare (demo) | CVE-2021-1675 | LPE version |

## Setup steps

1. ![DC setup](assets/dc-setup.png) Install AD DS + ADCS on TONYSTARK
2. ![Users OU](assets/users-ou.png) Create users and OUs in AD Users & Computers
3. Configure SPN for SQLService: `setspn -a MARVEL/SQLService.MARVEL.local:60111 MARVEL\SQLService`
4. Create GPO "Disable Windows Defender" and set to Enforced
5. Join workstations to MARVEL.local domain
6. Create `hackme` share on DC
7. Run BloodHound ingestion: `bloodhound-python -d MARVEL.local -u fcastle -p Password1 -ns <DC-IP> -c all`

![BloodHound graph](assets/bloodhound-output.png)

## Tools used
- Responder, ntlmrelayx, mitm6 (attack phase)
- BloodHound + neo4j, PlumHound, PingCastle (enumeration)
- Mimikatz, secretsdump.py, GetUserSPNs.py (post-compromise)
- crackmapexec / netexec (lateral movement)
