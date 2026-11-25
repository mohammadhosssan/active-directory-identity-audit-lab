# Active Directory identity lifecycle and audit lab

An isolated Windows Server 2022 lab that follows a fictional Finance user from CSV onboarding to an account disable, then traces the change back to the domain controller’s Security log. The lab runs on an Apple silicon Mac using UTM x86_64 emulation.

## Verified results

| Check | Result | Evidence |
| --- | --- | --- |
| Domain and OUs | `DC01.adlab.local` hosts `adlab.local` with separate **Lab Users** and **Lab Groups** OUs. | [Domain and OUs](evidence/01-domain-and-ou-structure.png) |
| CSV provisioning | Created five users, three global security groups and five role memberships. Finance and IT had two members each. Operations had one. | [First run](evidence/03-provisioning-run.png) · [Role memberships](evidence/02-role-groups-and-members.png) |
| Repeatability | The rerun created zero users and added zero memberships. All eight user and group object IDs matched the first run. | [Rerun](evidence/03b-provisioning-rerun.png) · [Object ID comparison](evidence/02-role-groups-and-members.png) |
| Identity audit | `mpatel` went from enabled and “Never logged on” to disabled and “Disabled account.” The other four accounts showed “Never logged on.” | [HTML audit report](evidence/04-audit-report.png) |
| Security event | DC01 recorded event **4725**, record **2441**, for `mpatel`. The exported CSV matched the original Security log record. | [Security event and CSV](evidence/05-security-events.png) |

## Reproduce the lab

1. Create a Windows Server 2022 Desktop Experience VM and copy the project via a read-only ISO. The tested setup used 2 vCPUs, 8 GiB RAM and an 80 GiB virtual disk. The host-only adapter had a static lab IP and no IPv4 or IPv6 default route. [Mac and UTM setup](docs/MAC-UTM-SETUP.md) · [General VM setup](docs/VM-SETUP.md)
2. Promote DC01, verify the domain with `Get-ADDomain`, provision twice, then run a baseline identity audit. [Windows PowerShell runbook](docs/LAB-GUIDE.md)
3. Enable account and group management auditing, disable `mpatel`, rerun the audit and match the exported event ID and record ID to the Security log. [Screenshot checklist](docs/SCREENSHOT-CHECKLIST.md)

## Interpret the findings

The HTML report shows directory state, while the event provides the actor, target and time of the change. “Never logged on” is expected for freshly provisioned identities. The report shows one prioritized finding per user, and the replicated logon timestamp can lag.

## Resume bullet

- Built an isolated Active Directory lab, automated CSV provisioning for five users across three role groups, verified repeatability by comparing object IDs, and traced a flagged account disable to Windows Security event 4725.

## Interview walkthrough

“I built a single domain controller and used PowerShell to provision five fictional users from a CSV. The role groups model how you would assign access by job function. On the second run, I compared object IDs so I could tell whether provisioning reused the same objects. Then I disabled one Finance user. The HTML report showed the new state, and I matched event 4725 in the export to the original Security record to verify who made the change and which account it affected.”
