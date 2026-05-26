# Technical Guide: Secure Remote SSH Access & Zero-Trust Tunneling

**Author:** Muhammad Danish Asyraff

**Objective:** To establish secure, remote command-line access to a local Windows host and Windows Subsystem for Linux (WSL) environment from a mobile device (Termux), enabling global access via a zero-trust encrypted tunnel.

## Prerequisites

* **Host Machine:** Windows 10/11 with OpenSSH Server installed and WSL (Kali Linux) enabled.
* **Client Device:** Android device with Termux installed.
* **Network:** Both devices initially connected to the same local Private Wi-Fi network.

---

## Phase 1: Local SSH Server Configuration (Windows Host)

### 1. Verify OpenSSH Server Status

Ensure the OpenSSH service is installed and actively running.
Open **Windows PowerShell as Administrator** and execute:

```powershell
Get-Service sshd

```

*If the status is "Stopped", start it using:* `Start-Service sshd`

### 2. Configure Windows Defender Firewall

Allow inbound TCP traffic on Port 22 so the server can receive SSH connection requests.
In the Administrator PowerShell, execute:

```powershell
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22

```

### 3. Verify Host IP Address

Obtain the local IPv4 address of the host machine to initiate the first connection.

```powershell
ipconfig

```

*(Note the IPv4 address under the active Wireless LAN adapter).*

---

## Phase 2: Client Configuration & Initial Access (Termux)

### 1. Update and Install Dependencies

Open Termux on the mobile device and ensure all packages are updated, followed by installing the OpenSSH client.

```bash
pkg update && pkg upgrade
pkg install openssh

```

### 2. Establish Local Connection

Connect to the Windows host using the local IP address discovered in Phase 1.

```bash
ssh username@<local_ip_address>

```

* **Note:** Accept the ED25519 key fingerprint prompt on the first connection.
* Authenticate using the standard Windows account password.
* Once authenticated, type `wsl` to drop directly into the Kali Linux environment.

---

## Phase 3: Global Remote Access Configuration (Tailscale)

To bypass local network restrictions and securely access the host from any external network (mobile data, public Wi-Fi) without relying on vulnerable port forwarding, a Tailscale VPN tunnel is implemented.

### 1. Host Machine Setup

1. Create a free account at [tailscale.com](https://tailscale.com).
2. Download and install the Tailscale Windows client.
3. Authenticate and connect.
4. Note the newly assigned permanent Tailscale IP address (format: `100.x.x.x`) from the system tray icon.

### 2. Client Device Setup

1. Install the Tailscale application from the Google Play Store.
2. Authenticate using the same account credentials.
3. Ensure the VPN status shows as "Active/Connected".

### 3. Establish Global Connection

Disconnect the mobile device from the local Wi-Fi and switch to cellular data to verify external access. Open Termux and connect using the Tailscale IP:

```bash
ssh username@100.x.x.x

```

---

## Appendix A: Security Options (Optional Public-Key Authentication)

While this setup utilizes standard password authentication, access can be hardened by disabling passwords and enforcing cryptographic keys.

**To implement Key-Based Access:**

1. Generate an ED25519 keypair in Termux: `ssh-keygen -t ed25519`
2. Copy the public key output from: `cat ~/.ssh/id_ed25519.pub`
3. On the Windows Host, paste the key into: `C:\Users\<Username>\.ssh\authorized_keys`
4. In `C:\ProgramData\ssh\sshd_config`, set `PasswordAuthentication no`.
5. Restart the SSH service: `Restart-Service sshd`

*(Note: To revert to password authentication, delete the `authorized_keys` file and set `PasswordAuthentication yes` in the sshd_config file).*

---

## Appendix B: Troubleshooting & Common Errors

* **Connection Hangs / Timeouts:** * Verify the Windows Defender Firewall rule is active.
* Ensure both the host and client devices are on the exact same network and that the Windows network profile is set to "Private" (not Public).


* **"Path Not Found" Error in PowerShell (Notepad):** * When using an Administrator terminal, the `~` directory shortcut may resolve to the `system32` folder instead of the user profile. Use explicit absolute file paths (e.g., `C:\Users\<Username>\.ssh`) when creating or editing files.
* **Permission Denied (publickey):** * If attempting key-based authentication on a Windows Administrator account, verify the `sshd_config` file. Ensure the `#` symbol is properly placed in front of both lines under the `Match Group administrators` block at the bottom of the file to prevent Windows from overriding the custom key location.
