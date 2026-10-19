# Pi Honeypot → Wazuh SIEM Pipeline

A home SOC lab: a Raspberry Pi 4 runs a **Cowrie SSH honeypot**, and a **Wazuh agent** ships its logs to a **Wazuh SIEM** server running in a VMware VM. Custom decoders and rules turn honeypot activity into alerts mapped to **MITRE ATT&CK**.

> 🚧 Work in progress. See the progress checklist below.

## Architecture

```
[Attacker / test machine]
          │  SSH (fake)
          ▼
┌──────────────────────┐        logs (1514/1515)      ┌──────────────────────────┐
│ Raspberry Pi 4       │ ───────────────────────────▶ │ Wazuh server (VMware VM) │
│  • Cowrie honeypot   │                              │  • Manager               │
│  • Wazuh agent       │                              │  • Indexer               │
└──────────────────────┘                              │  • Dashboard             │
                                                      └──────────────────────────┘
```

*(A proper diagram will go in `diagrams/`.)*

## Lab setup

| Component | Details |
|---|---|
| Sensor | Raspberry Pi 4, Raspberry Pi OS Lite (64-bit) |
| Honeypot | Cowrie |
| SIEM | Wazuh 4.14.9 (prebuilt OVA, Amazon Linux 2023) |
| Hypervisor | VMware Workstation on Windows |
| VM resources | 4 vCPU, 8 GB RAM, 50 GB disk, bridged networking |

## Progress

- [x] Wazuh server VM imported and running
- [x] Wazuh dashboard accessible
- [ ] Default passwords changed (system + dashboard)
- [ ] Raspberry Pi set up and hardened
- [ ] Wazuh agent installed on Pi and reporting "Active"
- [ ] Cowrie installed and catching login attempts
- [ ] Cowrie logs flowing into Wazuh
- [ ] Custom decoders and rules (MITRE ATT&CK mapped)
- [ ] Attack simulation and detection testing
- [ ] Custom dashboard
- [ ] Findings and lessons learned

## Repo layout

```
├── README.md          ← you are here
├── notes/             ← session logs: what I did, what broke, how I fixed it
├── screenshots/       ← evidence for each milestone
├── diagrams/          ← architecture diagrams
├── wazuh/             ← custom decoders and rules
└── cowrie/            ← Cowrie config (secrets removed)
```

## Troubleshooting log

Real problems hit during the build and how they were solved. Details in `notes/`.

| Problem | Fix |
|---|---|
| Wazuh OVA wouldn't boot in VMware: "The guest operating system has disabled the CPU" | Known issue with AMD CPUs; added CPUID compatibility lines to the `.vmx` file |
| VM had no IPv4 address (only IPv6 link-local) | Fixed network bridging so the VM gets a DHCP address from the home router |

## Lessons learned

*(Filled in at the end.)*
