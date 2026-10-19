# Session 1: Wazuh server setup

**Date:** 2026-10-18
**Goal:** Get the Wazuh SIEM running in a VM on my laptop.

## What I did

1. Installed VMware Workstation on Windows.
2. Downloaded the official prebuilt Wazuh OVA (v4.14.9, Amazon Linux 2023).
3. Imported it into VMware (File → Open → .ova).
4. Set resources: 4 vCPU, 8 GB RAM, bridged networking.
5. Booted and logged in as `wazuh-user`; changed the default system password with `passwd`.
6. Found the VM's IP with `ip a` and opened the Wazuh dashboard in my browser at `https://<VM-IP>`.

## Problems and fixes

### Problem 1: VM wouldn't boot
**Error:** "The guest operating system has disabled the CPU. Power off or reset the virtual machine."

**Cause:** Known compatibility issue between the Wazuh OVA and AMD processors in VMware Workstation.

**Fix:** Powered off the VM, closed VMware, and added CPUID compatibility lines from the Wazuh docs to the end of the VM's `.vmx` file. The VM then booted normally.

**Lesson:** Read the vendor's troubleshooting section first. Also learned the difference between the `.vmx` config file and the `.vmx.lck` lock file VMware creates while a VM is open.

### Problem 2: VM had no IPv4 address
**Symptom:** `ip a` showed `eth0` as UP but with only an IPv6 link-local (`fe80::`) address and no `inet` IPv4 line.

**Cause:** The VM's bridged adapter wasn't on a network that was handing out DHCP addresses.

**Fix:** Corrected the bridged network setup; after a reboot the VM got an IPv4 address from the home router.

**Lesson:** An interface being "UP" doesn't mean it has an IP. `ip a` shows the difference.

## Result
✅ Wazuh dashboard reachable and logged in.

## Next session
- Change the default dashboard `admin` password
- Snapshot the VM
- Install the Wazuh agent on the Raspberry Pi
