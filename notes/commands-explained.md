# Commands explained

Every command used in this project, what it does, and why I used it. This is my study guide.

## Networking and troubleshooting
| Command | What it does | Why I used it |
|---|---|---|
| `ip a` | Lists a Linux machine's network interfaces and their addresses | Find the Wazuh VM's IP; spot when an interface had no IPv4 address |
| `ipconfig` (Windows) | Same idea on Windows | Check the laptop's adapters |
| `Get-NetAdapter` (PowerShell) | Lists Windows adapters with their hardware descriptions | Match "Wi-Fi" to the adapter name VMware shows |
| `arp -a` (Windows) | Shows the IP-to-MAC address cache: devices the laptop has recently talked to | Find the Pi by its MAC vendor prefix (`d8:3a:dd` = Raspberry Pi, `00:0c:29` = VMware) |
| `ping <host>` | Sends a small "are you there?" packet | Tell "host is down" apart from "service is blocked" |
| `ping -4 <name>` | Ping forcing IPv4 | Get the Pi's IPv4 address by hostname |
| `ss -tlnp` | Lists listening TCP ports (`-t` TCP, `-l` listening, `-n` numeric, `-p` process) | Prove which services are actually listening and on which ports |

**Reading errors:**
- `Could not resolve hostname`: name lookup failed. The device can't be found by that name.
- `Connection refused`: the device answered, but nothing is listening on that port.
- `Connection timed out`: no answer at all. Often a firewall silently dropping the traffic, or the wrong IP.
- `TTL=64` in ping replies suggests Linux; `TTL=128` suggests Windows.

## SSH
| Command | What it does | Why I used it |
|---|---|---|
| `ssh user@host` | Opens a remote shell | Manage the Pi and the VM |
| `ssh -p 22222 user@host` | SSH on a specific port | Real SSH lives on 22222 now |
| `ssh-keygen -t ed25519` | Creates a public/private key pair | Key-based login instead of passwords |
| `type ...id_ed25519.pub \| ssh ... "cat >> ~/.ssh/authorized_keys"` | Sends the public key to the Pi and adds it to the allowed list | Authorize my laptop's key |
| `ssh -o PubkeyAuthentication=no ...` | Tries to log in without the key | Prove password login is disabled |
| `sudo sshd -t` | Tests the SSH server config for errors without applying it | Avoid breaking SSH with a typo |

**Host key prompt:** the first time you connect, SSH shows the server's fingerprint. If it ever changes unexpectedly, that could be a man-in-the-middle.

## Services (systemd)
| Command | What it does |
|---|---|
| `sudo systemctl start <svc>` | Start a service now |
| `sudo systemctl stop <svc>` | Stop it |
| `sudo systemctl restart <svc>` | Stop and start, to load new settings |
| `sudo systemctl enable <svc>` | Start it automatically at boot |
| `sudo systemctl enable --now <svc>` | Enable at boot **and** start now |
| `sudo systemctl status <svc>` | Show whether it's running, plus recent log lines (`q` to exit) |
| `sudo systemctl daemon-reload` | Make systemd re-read unit files after creating or editing one |

## Firewall (UFW)
| Command | What it does |
|---|---|
| `sudo ufw default deny incoming` | Block all inbound connections unless a rule allows them |
| `sudo ufw default allow outgoing` | Let the Pi start outbound connections (updates, Wazuh agent) |
| `sudo ufw allow from 192.168.1.0/24 to any port 22222 proto tcp` | Allow one port from one network only |
| `sudo ufw enable` | Turn the firewall on (add your SSH allow rule **first**) |
| `sudo ufw status verbose` | Show the rules and defaults |

**CIDR:** `192.168.1.0/24` means every address from `192.168.1.0` to `192.168.1.255`. The `/24` says the first three numbers are fixed.

## Packages and files
| Command | What it does |
|---|---|
| `sudo apt update` | Refresh the list of available packages |
| `sudo apt full-upgrade -y` | Install all updates |
| `sudo apt install -y <pkg>` | Install a package |
| `sudo tee <file> <<'EOF' ... EOF` | Write a block of text into a file that needs admin rights |
| `cp file file.orig` | Back up a config before changing it |
| `tail -f <file>` | Watch new lines appear in a log live (`Ctrl + C` to stop) |

## Users and permissions
| Command | What it does |
|---|---|
| `sudo <command>` | Run one command with admin rights |
| `sudo adduser --disabled-password cowrie` | Create a user that can't log in with a password |
| `sudo su - cowrie` | Switch to acting as the cowrie user (`exit` to go back) |
| `chown user:group <file>` | Change who owns a file |
| `chmod 700` / `600` / `770` | Set who can read/write/run a file (owner / group / others) |

## Cowrie
| Command | What it does |
|---|---|
| `python3 -m venv cowrie-env` | Create a private Python environment |
| `source cowrie-env/bin/activate` | Step into it (prompt shows `(cowrie-env)`) |
| `python -m pip install cowrie` | Install Cowrie into the venv |
| `cowrie init` | Create Cowrie's config and data folders |
| `AUTHBIND_ENABLED=yes cowrie start` | Start Cowrie using authbind for port 22 |
| `cowrie stop` | Stop it |

## Wazuh
| Command | What it does |
|---|---|
| `wget ...wazuh-agent...deb && sudo WAZUH_MANAGER='<ip>' ... dpkg -i ...` | Download and install the agent, pointing it at the manager |
| `sudo systemctl status wazuh-agent` | Check the agent is running |
| `sudo tail -n 20 /var/ossec/logs/ossec.log` | Read the agent's own log when troubleshooting |
