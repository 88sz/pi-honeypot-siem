# Session 2: Raspberry Pi + Wazuh agent

**Date:** 2026-10-19
**Goal:** Get the Raspberry Pi running and reporting to the Wazuh SIEM.

## What I did

1. Flashed Raspberry Pi OS Lite (64-bit, Debian 13) with Raspberry Pi Imager. Hostname `raspi4`, SSH enabled, Raspberry Pi Connect left **off** (no extra remote-access paths on a future honeypot).
2. Connected the Pi to my laptop with Ethernet through a USB-C adapter.
3. Locked the Wazuh VM's VMware bridge to my Wi-Fi adapter so it keeps a stable IP.
4. Installed the Wazuh agent on the Pi using the dashboard's "Deploy new agent" wizard (DEB aarch64 package, manager `192.168.1.87`, agent name `raspi`).
5. Enabled and started the `wazuh-agent` service.

## Problems and fixes

### Problem 3: Forgot the Pi's login
**Fix:** The Pi had nothing on it yet, so I reflashed the SD card and stored the new credentials in a password manager.
**Lesson:** Credential management matters even in a home lab.

### Problem 4: SSH "Connection refused"
**Symptom:** `raspi4.local` resolved (the Pi was on the network), but SSH on port 22 was refused.
**Cause:** The SSH service wasn't enabled on the fresh image.
**Fix:** Created an empty file named `ssh` in the SD card's boot partition, which makes Raspberry Pi OS enable SSH at boot.
**Lesson:** "Connection refused" means the host is reachable but nothing is listening on the port. "Could not resolve hostname" means the name lookup failed. Different errors point to different layers.

### Problem 5: Wazuh VM lost its IPv4 address after a host reboot
**Symptom:** `ip a` in the VM showed only an IPv6 link-local address.
**Cause:** The VMware bridge (VMnet0) was set to "Automatic" and picked the wrong host adapter after Windows restarted.
**Fix:** In VMware's Virtual Network Editor (admin mode), set VMnet0's "Bridged to" to the Wi-Fi adapter explicitly. Used `Get-NetAdapter` to match the Windows adapter name (`Wi-Fi`) to its hardware description.

### Problem 6: Agent install command had a broken IP
**Symptom:** The wizard's command showed `WAZUH_MANAGER='192.168.187'` (only three octets).
**Fix:** Re-entered the server address from `ip a` and checked the generated command before running it.
**Lesson:** Read commands before pasting them.

## Skills picked up
- Identifying devices from `arp -a` by MAC vendor prefix (`d8:3a:dd` = Raspberry Pi, `00:0c:29` = VMware).
- Checking which host adapter is which with `Get-NetAdapter`.

## Result
✅ Agent `raspi` (ID 001) shows **Active** in the Wazuh dashboard: Debian GNU/Linux 13, agent v4.14.9.

## Next session
- DHCP reservations for the Pi and the VM in the router
- Install Cowrie on the Pi
- Send Cowrie's JSON logs to Wazuh
