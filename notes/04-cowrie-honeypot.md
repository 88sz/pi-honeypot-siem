# Session 4: Cowrie honeypot

**Date:** 2026-10-20 to 2026-10-21
**Goal:** Install Cowrie on port 22, test it by attacking it myself, and make it survive reboots.

## What Cowrie is
Cowrie is a **medium-interaction SSH honeypot**. It pretends to be a real Linux server: fake login, fake filesystem, fake command output. It logs every credential, command, and download attempt. Attackers interact with a simulation, not the real Pi.

## Part A: Install (least privilege)
- Installed dependencies: Python tooling, crypto libraries, build tools, and `authbind`.
- Created a dedicated **`cowrie`** user with `adduser --disabled-password`: no password, no sudo. If an attacker ever escaped the emulated shell, they'd land as a powerless user.
- Installed Cowrie **3.1.3** with pip into its own Python **virtual environment** (`~/my-honeypot/cowrie-env`), isolated from the system Python.
- Ran `cowrie init` to create the config, log, download, and TTY-recording directories.

## Part B: Configure
Backed up the generated config, then set only the values I needed in `etc/cowrie.cfg`:

```ini
[honeypot]
hostname = srv-db01

[ssh]
listen_endpoints = tcp:22:interface=0.0.0.0
```

- **`srv-db01`:** a believable database-server hostname, more tempting than a default name.
- **Port 22:** ports under 1024 normally require root. Instead of running Cowrie as root, I used **authbind** to grant only the `cowrie` user permission for only port 22:
  ```
  sudo touch /etc/authbind/byport/22
  sudo chown cowrie:cowrie /etc/authbind/byport/22
  sudo chmod 770 /etc/authbind/byport/22
  ```
- **Staged exposure:** opened port 22 in UFW only to my own LAN subnets for testing. Exposing it to the internet is a later, deliberate decision.

Verified with `ss -tln`:
```
0.0.0.0:22       ← Cowrie (fake SSH)
0.0.0.0:22222    ← real SSH (hardened)
[::]:22222       ← real SSH (IPv6)
```

## Part C: Attacking my own honeypot
From my laptop I ran `ssh root@<pi>` on port 22 and acted like an attacker: `whoami`, `uname -a`, `cat /etc/passwd`, `ls -la /root`, `wget http://example.com/`, `exit`. I watched the log live with `tail -f var/log/cowrie/cowrie.log`.

### What the log captured
| Log evidence | Why it matters to a defender |
|---|---|
| `public key attempt for user root ... fingerprint 86:9d:...` | My SSH client offered its key first. Key fingerprints are stable, so they can link the same attacker across different IPs. |
| `login attempt [root/...] succeeded` | Exact username and password captured. At scale this shows which credentials bots try. |
| `uname -a` returned `x86_64 ... Debian 6.1.0` | Deception: the Pi is really ARM on a newer kernel. |
| `CMD: cat /ect/passwd` | Every command is logged, typos included. Typos suggest a human rather than a bot. |
| `Downloaded URL (http://example.com/) with SHA-256 25ddf2c8...` | Cowrie fetched the file and quarantined it under its SHA-256 hash. Real malware hashes can be checked on VirusTotal without running anything. |
| `Closing TTY Log: var/lib/cowrie/tty/...` | Full session recording that can be replayed keystroke by keystroke. |
| `Connection lost after 172483 milliseconds` | Session length (about 3 min). Bots tend to take seconds; humans take minutes. |

**Key concept:** every line carries a **session ID** (for example `d187fda9bf7b`). That's what will let Wazuh correlate a successful login with the commands that followed.

Cowrie accepts most passwords but rejects a few obvious ones, so it looks like a real server with a weak password instead of an obvious trap.

## Part D: Persistence with systemd
A sensor that silently stops after a reboot is a **visibility gap**. I wrote `/etc/systemd/system/cowrie.service`:

```ini
[Unit]
Description=Cowrie SSH Honeypot
After=network-online.target
Wants=network-online.target

[Service]
Type=forking
User=cowrie
Group=cowrie
WorkingDirectory=/home/cowrie/my-honeypot
Environment=AUTHBIND_ENABLED=yes
Environment=VIRTUAL_ENV=/home/cowrie/my-honeypot/cowrie-env
Environment=PATH=/home/cowrie/my-honeypot/cowrie-env/bin:/usr/bin:/bin
ExecStart=/home/cowrie/my-honeypot/cowrie-env/bin/cowrie start
ExecStop=/home/cowrie/my-honeypot/cowrie-env/bin/cowrie stop
PIDFile=/home/cowrie/my-honeypot/var/run/cowrie.pid
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

- Runs as `cowrie`, never root.
- Restarts automatically if it crashes.
- Enabled with `sudo systemctl enable --now cowrie`.

**Verified by rebooting the Pi.** After `sudo reboot`, `systemctl status cowrie` showed `active (running)` without any manual steps.

## Result
✅ Cowrie runs on port 22 as an unprivileged user, captures credentials, commands, and downloads, and starts on boot.

## Next session
- Point the Wazuh agent at Cowrie's JSON log (`var/log/cowrie/cowrie.json`)
- Confirm Cowrie events show up in Wazuh
