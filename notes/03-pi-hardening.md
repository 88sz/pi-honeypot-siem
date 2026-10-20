# Session 3: Hardening the Pi

**Date:** 2026-10-19
**Goal:** Lock down real management access before installing the honeypot.

## Why
Cowrie will act as a fake SSH server on port 22, the port attackers scan most. The real SSH has to be isolated first: key-only, on a different port, and reachable only from my LAN. The honeypot gets attacked; the management channel shouldn't be reachable at all.

## Phase 1: SSH key authentication
- Generated an **ed25519** key pair on my Windows laptop with `ssh-keygen -t ed25519`, protected by a passphrase.
- Copied the **public** key to the Pi's `~/.ssh/authorized_keys`, with permissions locked down (`700` on `~/.ssh`, `600` on the file).
- Confirmed the key login works **before** removing password login.

**Concept:** The private key never leaves the laptop. The Pi checks that I hold it using a cryptographic challenge, so no secret crosses the network.

**Concept:** When I first connected by IP instead of hostname, SSH asked me to verify the host key again. The fingerprint matched the one saved for `raspi4.local`, confirming the same machine and ruling out a man-in-the-middle.

## Phase 2: SSH configuration
Created `/etc/ssh/sshd_config.d/00-hardening.conf`:

```
Port 22222
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
AllowUsers victor
MaxAuthTries 3
```

- The `00-` prefix matters: sshd uses the **first** value it reads for each setting, so this file overrides any distro defaults in the same folder.
- Validated with `sudo sshd -t` before restarting.
- Confirmed the listener with `sudo ss -tlnp | grep ssh`: port 22222 on IPv4 and IPv6.
- Kept the original session open as a lifeline while testing in a new window.

### Verification
| Test | Expected | Result |
|---|---|---|
| `ssh -p 22222 victor@192.168.1.89` with key | Login succeeds | ✅ |
| `ssh -p 22222 -o PubkeyAuthentication=no victor@192.168.1.89` | `Permission denied (publickey)` | ✅ |

## Phase 3: Host firewall (UFW)
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.1.0/24 to any port 22222 proto tcp comment 'Admin SSH from LAN only'
sudo ufw enable
```

Result of `sudo ufw status verbose`:
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)

To          Action     From
22222/tcp   ALLOW IN   192.168.1.0/24   # Admin SSH from LAN only
```

- **Default deny:** block everything inbound and allow only what's needed.
- Outbound stays open so the Wazuh agent can reach the manager and the Pi can get updates.
- Added the allow rule **before** enabling the firewall, to avoid cutting off my own session.
- Also enabled `unattended-upgrades` for automatic security patches.

**Lesson (IPv6):** The admin rule only allows my IPv4 LAN. `raspi4.local` sometimes resolves to the Pi's IPv6 address, which the firewall now blocks, so I connect by IPv4 address. Forgetting IPv6 is a common firewall gap.

## Problems and fixes
### Problem 7: Key passphrase rejected
Mistyped the passphrase a few times. It worked once typed carefully. This is why the key was tested **before** disabling passwords: change one thing, verify it, then remove the fallback.

## Result
✅ Management SSH is key-only, on port 22222, LAN-only behind a default-deny firewall. Port 22 is free for the honeypot.

## Next session
- Install Cowrie and point port 22 at it
- Send Cowrie's JSON logs to Wazuh
