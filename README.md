# Home-Lab

Home lab built to get hands-on experience with SOC analyst workflows, using Kali Linux for attack simulation and a Wazuh SIEM for detection and triage.

## Goal

Simulate a small, realistic network with an attacker, an endpoint, and a server, then use Wazuh to detect and investigate the resulting activity. The lab is intentionally scoped down to what a single laptop can run, with an emphasis on safe isolation so attack traffic never reaches the home network.

## Environment

Hypervisor: VMware Workstation, running on a laptop (8 cores, 16GB RAM), so each VM is sized conservatively.

| VM | Role | RAM | Cores |
|---|---|---|---|
| Kali Linux | Attacker box, used to run tools like Hydra against lab targets | 4GB | 2 |
| Fedora Workstation | Simulated end-user endpoint / attack target | 3GB | 2 |
| Fedora Server | Log source / server-side target | 2GB | 2 |

## Network Design

Attack traffic is kept fully isolated from the home network using VMware's Virtual Network Editor:

- All lab VMs sit on a dedicated Host-only network (VMnet0), with no host virtual adapter attached. This means the VMs can reach each other, but there is no path from the lab to the host machine or the physical network.
- IP addressing is static rather than DHCP-assigned, to keep the topology predictable during exercises.

| VM | IP Address |
|---|---|
| Kali Linux | 192.168.233.10 |
| Fedora Workstation | 192.168.233.20 |
| Fedora Server | 192.168.233.30 |

Connectivity between all three VMs has been verified via ping.

Because the isolated network has no route to the host, log forwarding to Wazuh is handled over a separate management network rather than the attack network itself, so monitoring traffic and attack traffic never mix. Fedora Workstation and Fedora Server each have a second NIC on VMnet1 (Host-only, connected to the host) dedicated to this purpose. Kali does not need this second NIC, since it's the attacker rather than something being monitored.

| VM | Management IP (VMnet1) |
|---|---|
| Fedora Workstation | 192.168.236.128 |
| Fedora Server | 192.168.236.129 |

## Detection Stack

Wazuh 4.12.0 runs in Docker on the host machine. Wazuh agents are installed on Fedora Workstation and Fedora Server, forwarding event data to the manager over the VMnet1 management network for correlation and alerting.

## Exercises Completed

- Deployed the Wazuh manager, then installed and registered agents on both target VMs, verifying each shows active in the dashboard.
- Ran SSH brute-force simulations against lab targets using Hydra, then reviewed and triaged the resulting alerts in Wazuh.

## Planned Additions

- Write-ups of individual exercises (attack performed, detection generated, analysis of the alert) under a `/docs` folder.
- Additional detection rules and use cases as the lab grows.
