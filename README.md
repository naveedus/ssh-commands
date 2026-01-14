# SSH Cheat Sheet

A hands-on, copy-paste friendly reference for SSH usage, designed for Linux engineers, system administrators, DBAs, and SREs.

---

## Table of Contents

### Part 1 — SSH Fundamentals (For Everyone)
1. Basic SSH Connection
2. Using Custom Ports and Users
3. SSH Key-Based Authentication
4. Power of the SSH Config File
5. SSH Tunneling & Port Forwarding (Concepts)
6. File Transfers with SCP & Rsync
7. Troubleshooting SSH Connections
8. SSH Security Best Practices

### Part 2 — SSH Options Explained (Beginner → Advanced)
1. Understanding SSH Command Structure
2. Connection & Authentication Options
3. Debugging and Verbose Modes
4. Port Forwarding Options
5. Jump Hosts & Bastion Servers
6. Session and Terminal Controls
7. Security & Authentication Flags
8. Networking & Advanced Routing
9. Encryption & Algorithm Selection  
10. ControlMaster & Connection Reuse

---

## Part 1 — SSH Fundamentals (For Everyone)

### 1. Basic SSH Connection

```bash
ssh user@server_ip
```

### Connects to a remote server using the default SSH port (22).
ssh hostname
Uses settings defined in ~/.ssh/config, if available.

### 2. Using Custom Ports and Users
```bash
ssh -p 2222 user@server_ip
```
Connect to a server running SSH on a non-standard port.
```bash
ssh user@hostname
```
Log in using a specific remote user.

### 3. SSH Key-Based Authentication
Generate a New SSH Key:
```bash
ssh-keygen -t ed25519
```
Copy the Public Key to the Server:
```bash
ssh-copy-id user@server_ip
```
Once configured, password-based login is no longer required.

### 4. Power of the SSH Config File (Highly Recommended)
Edit your SSH configuration file:
```bash
nano ~/.ssh/config
```
Example configuration:
```
Host prod-db
  HostName 10.10.10.20
  User postgres
  Port 22
  IdentityFile ~/.ssh/id_ed25519
```
Connect using the alias:
```bash
ssh prod-db
```
Benefits:
- Cleaner commands
- Faster connections
- Fewer mistakes

### 5. SSH Tunneling & Port Forwarding
Local Port Forwarding:
```bash
ssh -L 5432:localhost:5432 user@server_ip
```
Access a remote PostgreSQL instance as if it were running locally.
Remote Port Forwarding:
```bash
ssh -R 9000:localhost:3000 user@server_ip
```
Expose a local service to the remote machine.
Dynamic Port Forwarding (SOCKS Proxy):
```bash
ssh -D 8080 user@server_ip
```
Useful for secure browsing and proxy-based traffic routing.

### 6. File Transfers with SCP & Rsync
Copy a File to a Remote Server:
```bash
scp file.txt user@server_ip:/path/
```
Copy a Directory:
```bash
scp -r mydir user@server_ip:/path/
```
Faster & Smarter Sync (Recommended):
```bash
rsync -avz file.txt user@server_ip:/path/
```

### 7. Troubleshooting SSH Connections
Enable verbose output:
```bash
ssh -v user@server_ip
```
Maximum verbosity:
```bash
ssh -vvv user@server_ip
```
Force a specific SSH key:
```bash
ssh -i ~/.ssh/mykey user@server_ip
```

### 8. SSH Security Best Practices
Recommended hardening steps:
- Disable password authentication
- Use SSH keys only
- Change the default SSH port
- Enable Fail2Ban
- Restrict access with AllowUsers
Example (sshd_config):
```
PasswordAuthentication no
PermitRootLogin no
```
Restart the SSH service:
```bash
sudo systemctl restart sshd
```

## Part 2 — SSH Options Explained (Beginner → Advanced)

This section breaks down commonly used SSH command-line options in simple terms.

### SSH Command Structure
```
ssh [options] user@host
```
user → Remote Linux username
host → Server IP or hostname
options → Flags that modify SSH behavior

### Core & Common Options (Must-Know)
- `-p` — Specify Port
```bash
ssh -p 2222 user@server
```
Default SSH port is 22. Changing ports is a common security practice.

- `-l` — Login Name
```bash
ssh -l postgres server_ip
```
Equivalent to:
```bash
ssh postgres@server_ip
```

- `-i` — Identity File (SSH Key)
```bash
ssh -i ~/.ssh/id_ed25519 user@server
```
Useful when managing multiple SSH keys.

- `-v, -vv, -vvv` — Verbose / Debug Mode
```bash
ssh -v user@server
```
Helps diagnose:
- Authentication failures
- Incorrect keys
- Network issues
More v means more detail.

- `-F` — Custom SSH Config File
```bash
ssh -F myconfig user@server
```
Default configuration file:
```
~/.ssh/config
```

### Authentication & Security Options
- `-o` — Custom SSH Option
```bash
ssh -o StrictHostKeyChecking=no user@server
```
Pass advanced options inline.
Common examples:
```
-o PasswordAuthentication=no
-o ConnectTimeout=5
```

- `-A` — Agent Forwarding
```bash
ssh -A user@server
```
Allows your local SSH keys to be used on remote systems (commonly for jump hosts).
⚠️ Use cautiously on untrusted servers.

- `-a` — Disable Agent Forwarding
```bash
ssh -a user@server
```
A safer default when forwarding is not needed.

- `-K` — GSSAPI / Kerberos Authentication
```bash
ssh -K user@server
```
Primarily used in enterprise environments.

### Port Forwarding & Tunneling Options
- `-L` — Local Port Forwarding
```bash
ssh -L 5432:localhost:5432 user@server
```
Common use cases:
- PostgreSQL
- MySQL
- Internal web services

- `-R` — Remote Port Forwarding
```bash
ssh -R 8080:localhost:3000 user@server
```
Expose a local service to a remote system.

- `-D` — Dynamic Port Forwarding (SOCKS Proxy)
```bash
ssh -D 1080 user@server
```
Routes traffic through a secure SSH tunnel.

- `-W` — Forward Standard Input/Output
```bash
ssh -W host:port user@server
```
Mostly used internally by ProxyJump.

### Jump Hosts & Multi-Hop Access
- `-J` — Jump Host / Bastion Server
```bash
ssh -J user@jumpserver user@target
```
Modern and clean alternative to complex SSH tunnels.

### Session & Terminal Control
- `-N` — No Remote Command
```bash
ssh -N -L 5432:localhost:5432 user@server
```
Opens a tunnel without starting a shell.

- `-T` — Disable Pseudo-TTY
```bash
ssh -T user@server
```
Common in automation scripts.

- `-t` — Force TTY Allocation
```bash
ssh -t user@server "sudo systemctl restart postgresql"
```
Required for commands needing interactive input.

### Background & Connection Control
- `-f` — Run in Background
```bash
ssh -f -N -L 5432:localhost:5432 user@server
```
Ideal for persistent tunnels.

- `-S` — Control Socket
```bash
ssh -S /tmp/ssh.sock user@server
```
Used for connection sharing.

- `-O` — Control Existing Connections
```bash
ssh -O exit user@server
```
Send commands to an existing SSH session.

### Encryption & Algorithms
- `-c` — Cipher Selection
```bash
ssh -c aes256-gcm@openssh.com user@server
```

- `-m` — MAC Algorithm
```bash
ssh -m hmac-sha2-256 user@server
```

- `-Q` — Query Supported Algorithms
```bash
ssh -Q cipher
ssh -Q mac
```

### Network-Level Options
- `-B` — Bind to Interface
```bash
ssh -B eth0 user@server
```

- `-b` — Source Address
```bash
ssh -b 192.168.1.10 user@server
```

### Final Tip
If an SSH command becomes long or hard to remember, move it into ~/.ssh/config.
Short aliases and configuration files make SSH safer, faster, and easier to use.
SSH is more than a login tool — it’s a foundation for secure access, automation, and production-grade operations. Master the basics, and you can manage any Linux system. Master the options, and you can confidently handle complex infrastructures.
Save this cheat sheet — you’ll come back to it more often than you expect.

### This work was not entirely human-created, with partial use of AI for formatting. 
