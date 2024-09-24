# remote-access
Setting up RDP from desktop to headless environment

## Remote Access Setup from Beelink (Windows) to Intel NUC (Linux)
This document provides step-by-step instructions on setting up RDP and SSH for remote access from your Beelink running Windows to your headless Intel NUC running Ubuntu Mate.

### Requirements
- **Intel NUC** running Ubuntu 24.04 and later.
- **Beelink** running Windows 11 and later.
- Network access between both devices.
- Basic command-line skills (or just ability to copy and paste the code in this document).
- Your username/ip address for the device you'll be remoting into.
---

## 1. RDP Setup on Intel NUC

### 1.1 Install and Configure XRDP for Remote Access

**Why?**  
RDP (Remote Desktop Protocol) allows you to control your NUC from another device as if you were sitting in front of it. XRDP is the Linux package that enables this feature.

**Install XRDP on your Intel NUC:**

```bash
sudo apt update
sudo apt install xrdp
```

### 1.2 Install a Lightweight Desktop Environment
**Why?**
RDP requires a desktop environment to work. Ubuntu Mate is lightweight, but GNOME can interfere with RDP as of 2024. Therefore, using a lighter desktop like XFCE or Mate is recommended for remote access.

### Install XFCE:

```
sudo apt install xfce4 xfce4-goodies
```

This installs the XFCE desktop environment, which is lightweight and works well with XRDP.

Install MATE (optional, if you prefer MATE):

```
sudo apt install ubuntu-mate-desktop
```
You can choose either XFCE or MATE for your remote sessions. Both are lighter than GNOME and better suited for RDP.

### 1.3 Set RDP to Listen on Port 3391
**Why?**
By default, RDP listens on port 3389, but changing it to another port like 3391 adds a layer of security (obscurity), as bots often scan for port 3389.

Edit XRDP config file to change the default port:

```
sudo nano /etc/xrdp/xrdp.ini
```

Find the line port=3389 and change it to port=3391.

Restart XRDP for the changes to take effect:

```
sudo systemctl restart xrdp
```
### 1.4 GNOME Interference with RDP (as of 2024)
**Why?**
GNOME desktop environment can interfere with RDP sessions as of 2024. It is recommended to use lighter environments like XFCE or MATE for remote desktop access via RDP to avoid potential issues.

### 1.5 Unset DBUS in XSession or StartWM.sh
**Why?**
Unsetting DBUS_SESSION_BUS_ADDRESS helps resolve permission errors that can occur during RDP sessions, especially when using lightweight desktop environments.

Edit your .xsession file:

```
nano ~/.xsession
```
Add the following:

```
unset DBUS_SESSION_BUS_ADDRESS
exec startxfce4  # Adjust for your desktop environment
```
OR edit the /etc/xrdp/startwm.sh file:

```
sudo nano /etc/xrdp/startwm.sh
```

Add the same unset command before starting the window manager.

**Note: In many setups, you only need to edit one of these files. If unsure, start with the one your system uses for session management.**

### 1.6 Open Port 3391 in Firewall
**Why?**
To allow RDP traffic through the firewall, we need to open the port where XRDP is listening.

Using UFW (Uncomplicated Firewall):

```
sudo ufw allow 3391/tcp
sudo ufw reload
```

Using iptables:

sudo iptables -A INPUT -p tcp --dport 3391 -j ACCEPT
sudo iptables-save
### 1.7 Port Forwarding (Optional)
**Why?**

If you want to access your Intel NUC from outside your local network, you need to set up port forwarding on your router.

External Port: 3391
Internal IP: [Intel NUC's local IP]
Internal Port: 3391

---
## 2. SSH Setup on Intel NUC
### 2.1 Install OpenSSH Server
**Why?**
SSH allows secure terminal-based remote access to your Intel NUC. It’s essential for server management without needing a desktop environment.

Install OpenSSH:

```
sudo apt update
sudo apt install openssh-server
```
Start and Enable SSH:

```
sudo systemctl start ssh
sudo systemctl enable ssh
```

### 2.2 Configure Firewall to Allow SSH
**Why?**
You need to allow SSH traffic through the firewall so that you can access the Intel NUC remotely.

Using UFW:

```
sudo ufw allow 22/tcp
sudo ufw reload
```

**Using iptables:**
```
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables-save
```

### 2.3 Secure SSH Configuration
**Why?**
Securing SSH helps prevent unauthorized access to your system.

Edit SSH configuration:

```
sudo nano /etc/ssh/sshd_config
```

### Key Security Settings:

Disable root login:
```
PermitRootLogin no
```

Optionally allow only specific users:
```
AllowUsers yourusername
```
Optionally change the SSH port (for added security):
**P.S. I never did this because why bother?**
```
Port 2222
```
Restart SSH to apply changes:
```
sudo systemctl restart ssh
```

### 2.4 SSH Key Authentication (Optional)
**Why?**

Using SSH keys instead of passwords significantly enhances security.

Generate SSH key on your Beelink:

```
ssh-keygen -t rsa
```
Copy the public key to your Intel NUC:

```
ssh-copy-id yourusername@your-nuc-ip
```
Disable password-based login:

Edit /etc/ssh/sshd_config and set:
```
PasswordAuthentication no
```
Restart SSH:

```
sudo systemctl restart ssh
```
## 3. Accessing Your Intel NUC from Beelink
### 3.1 Remote Access via RDP
Why?
RDP allows full graphical access to your Intel NUC from the Beelink.

Open Remote Desktop Connection on Beelink.
Enter your Intel NUC’s IP and port in this format:

```
your-nuc-ip:3391
```
Log in using your NUC’s username and password.
### 3.2 Remote Access via SSH
**Why?**

SSH allows command-line access to manage your Intel NUC. This is pretty good if (but really when) RDP decides to just not work one day. You'll still be able to access some of the stuff you need to. Just without the benefit of the GUI. You maybe be able to make changes to some of the config files you need in order to get RDP running again. 

Open PuTTY or any SSH client on Beelink.
Enter your NUC’s IP and port (default 22 or 2222 if changed).
Log in using your SSH username, password, or key (if configured).
---
## Conclusion
This setup ensures that you can securely and efficiently access your Intel NUC remotely from your Beelink using both RDP for graphical access and SSH for command-line access. Using XFCE or MATE desktop environments avoids compatibility issues with GNOME and improves performance.
