## Lab Update: SSH Setup and Hardening on Fedora Server (Sep 28, 2026)

### Objective
Establish SSH access from the Fedora Workstation (192.168.233.20) to the Fedora Server (192.168.233.30) over the isolated VMnet0 attack network, move from password to key-based authentication, and harden the sshd configuration.

### What I Did

**1. Installing OpenSSH on an isolated network**
The first `dnf install` attempt hung because VMnet0 has no internet access, so dnf couldn't reach Fedora's mirrors to fetch repo metadata. This is expected on an isolated segment and a reminder that package management on air-gapped hosts needs a plan (a local repo from the install ISO, pre-downloaded RPMs, or temporary network access).

**2. Enabling sshd and the firewall rule**
```bash
sudo systemctl enable --now sshd
sudo firewall-cmd --permanent --add-service=ssh && sudo firewall-cmd --reload
```

**3. First connection and host key verification**
Connected from the workstation with `ssh jesse@192.168.233.30`. On first connect, I accepted the server's ED25519 host key fingerprint, which is now stored in `~/.ssh/known_hosts`. Any future change to that key will trigger a warning, which protects against man-in-the-middle attacks.

**4. Setting a hostname on the server**
The server's hostname was `localhost`, which made it hard to tell which machine a shell belonged to. Renamed it:
```bash
sudo hostnamectl set-hostname fedora-srv
```

**5. Key-based authentication**
Generated an ED25519 key pair on the workstation and copied the public key to the server:
```bash
ssh-keygen -t ed25519
ssh-copy-id jesse@192.168.233.30
```
Verified that login succeeds without a password prompt.

**6. Hardening sshd**
Created a drop-in config to disable password logins and direct root login:
```bash
sudo tee /etc/ssh/sshd_config.d/00-hardening.conf <<'EOF'
PasswordAuthentication no
PermitRootLogin no
EOF
sudo sshd -t && sudo systemctl restart sshd
```
The `00-` prefix matters: sshd uses the first value it reads for each setting, and Fedora ships its own `50-redhat.conf` in the same directory. A lower number ensures these settings take precedence. `sshd -t` validates the config before the restart so a typo can't take the service down.

**7. Verification**
Attempted a login with public key auth explicitly disabled:
```bash
ssh -o PubkeyAuthentication=no jesse@192.168.233.30
```
Result: `Permission denied (publickey,gssapi-keyex,gssapi-with-mic)`. The server no longer offers `password` as an authentication method.

Confirmed key-based login from the workstation still works after the restart:
```bash
ssh -v jesse@192.168.233.30 2>&1 | grep -i "authenticated"
```
Result: `Authenticated to 192.168.233.30 ([192.168.233.30]:22) using "publickey"`.

### Design Decision: Leaving Cockpit Enabled
The login banner advertises Cockpit, Fedora's web admin console, on port 9090. I'm intentionally leaving it running. With SSH locked down to key-only auth, Cockpit provides a second, web-based attack surface on the server target for future red team exercises from the Kali VM, and a separate source of authentication events to detect in Wazuh.

### Next Steps
- Confirm SSH and Cockpit authentication events from fedora-server are reaching the Wazuh manager
- Plan attack scenarios from Kali against both services (SSH brute force against key-only auth, web login attacks against Cockpit)
