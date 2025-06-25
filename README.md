Here is a **step-by-step, beginner-friendly guide** to build a test environment on Windows using VMware Workstation with three Debian instances. We'll start with Windows setup, then install required tools, and finally create the Debian VMs.

---

## ✅ Part 1: Prepare the Windows Host Environment

### 1. Install New PowerShell from Microsoft Store

1. Click the **Start Menu**, search for **Microsoft Store**, and open it.
2. In the store, search for **PowerShell**.
3. Choose **PowerShell (Preview or Stable)** and click **Install**.
4. Once installed, open PowerShell via Start Menu > PowerShell.

### 2. Create SSH Directory and Config File

1. Open the new PowerShell.
2. Run:

   ```powershell
   mkdir -Force $HOME\.ssh
   New-Item -Path $HOME\.ssh\config -ItemType File -Force
   ```

---

## ✅ Part 2: Install Git and Set Up SSH for GitHub

### 3. Download and Install Git

1. Go to [https://git-scm.com/](https://git-scm.com/)
2. Download the Windows installer.
3. Run the installer and accept all default settings (or adjust install path if needed).

### 4. Generate a New SSH Key

1. Open Git Bash (search for it in Start Menu after installing Git).
2. Run:

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```
3. Press Enter to accept the default file path (`/c/Users/yourname/.ssh/id_ed25519`)
4. When prompted, use a passphrase or just press Enter twice.

### 5. Add SSH Key to GitHub

1. Run:

   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
2. Copy the output (your public key).
3. Go to [https://github.com/settings/keys](https://github.com/settings/keys)
4. Click **New SSH Key**, paste your key, give it a title (e.g., "VM Workstation PC"), and save.

### 6. Edit `$HOME/.ssh/config`

1. Open PowerShell and run:

   ```powershell
   notepad $HOME\.ssh\config
   ```
2. Paste:

   ```
   Host github.com
     HostName github.com
     User git
     IdentityFile ~/.ssh/id_ed25519
   ```

---

## ✅ Part 3: Install Required Software on Windows

### 7. Install VMware Workstation Player (Free)

1. Go to [https://www.vmware.com/products/workstation-player.html](https://www.vmware.com/products/workstation-player.html)
2. Click **Try Workstation Player**, then **Download for Windows**.
3. Run the installer, accept defaults, and reboot if required.

### 8. Install Python 3 (Latest Version)

1. Go to [https://www.python.org/downloads/windows/](https://www.python.org/downloads/windows/)
2. Download the latest Windows installer.
3. Run it with:

   * ✅ Check “Add Python to PATH”
   * Click **Install Now**

### 9. Install OpenJDK (Latest Version)

1. Go to [https://jdk.java.net/](https://jdk.java.net/)
2. Choose the latest GA build, download the Windows `.zip` or `.msi`.
3. If `.zip`:

   * Extract it to `C:\Java\jdk-XX`
   * Add `C:\Java\jdk-XX\bin` to **Environment Variables > PATH**
4. If `.msi`, it does this automatically.

### 10. Install Docker Desktop

1. Go to [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. Download and install.
3. After install:

   * Enable **WSL2-based engine**
   * Install **Docker Compose plugin** if not already bundled

---

## ✅ Part 4: Prepare Debian Virtual Machines

### 11. Download Debian ISO (DVD Version)

1. Go to [https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/](https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/)
2. Download the file: `debian-XX.X.X-amd64-DVD-1.iso` (\~4.4 GB)

### 12. Create Three Debian VMs in VMware Workstation Player

**Repeat these steps three times to create 3 identical VMs:**

1. Open **VMware Workstation Player**
2. Click **Create a New Virtual Machine**
3. Choose **Installer disc image file (iso)** and browse to your downloaded Debian ISO.
4. Click **Next**, select **Linux** and choose **Debian 11.x or later**.
5. Name the VM (e.g., debian-node-1) and choose a location.
6. Set disk size: **20 GB**, store as **single file**.
7. Click **Customize Hardware**:

   * Memory: **8192 MB (8 GB)**
   * CPU: Leave at 1 or 2 cores
   * Network Adapter: NAT or Bridged
   * CD/DVD: Verify ISO is selected
8. Click **Finish**.

**Repeat for:**

* debian-node-2
* debian-node-3

---

## ✅ Part 1: Create the `devops` User and SSH Access

### 1. Log in as root (or user with sudo)

### 2. Create the `devops` user and grant sudo access

```bash
adduser devops
usermod -aG sudo devops
```

### 3. Create SSH directory for `devops` and proper permissions

```bash
mkdir -p /home/devops/.ssh
chmod 700 /home/devops/.ssh
touch /home/devops/.ssh/authorized_keys
chmod 600 /home/devops/.ssh/authorized_keys
chown -R devops:devops /home/devops/.ssh
```

---

## ✅ Part 2: Generate and Deploy SSH Keys

### On your **Windows host**, run PowerShell:

Repeat for each VM (change `iceworm1`, `iceworm2`, `iceworm3`):

```powershell
ssh-keygen -t rsa -b 2048 -f $HOME\.ssh\devops.iceworm1
```

Then copy the **public key** to the VM (repeat per VM):

```bash
# On Windows (Git Bash or PowerShell)
scp $HOME/.ssh/devops.iceworm1.pub devops@<VM-IP>:/tmp/iceworm1.pub

# On the VM:
cat /tmp/iceworm1.pub >> /home/devops/.ssh/authorized_keys
rm /tmp/iceworm1.pub
chown devops:devops /home/devops/.ssh/authorized_keys
```

---

## ✅ Part 3: SUDO with NOPASSWD and Logging

```bash
echo "%devops ALL=(ALL) NOPASSWD: LOG_INPUT: ALL" > /etc/sudoers.d/devops
chmod 440 /etc/sudoers.d/devops
```

---

## ✅ Part 4: Install Development Tools

### Update packages first:

```bash
sudo apt update && sudo apt upgrade -y
```

### Install Docker (Official Method)

```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker devops
```

### Install Python, pip, Java, Maven

```bash
sudo apt install -y python3 python3-pip openjdk-17-jdk maven
```

### Optional: Useful developer tools

```bash
sudo apt install -y git curl unzip htop net-tools tmux wget make build-essential
```

---

## ✅ Part 5: Configure `/opt` for Development

```bash
sudo chgrp -R devops /opt
sudo chmod -R g+wrx /opt
```

> `/opt` is now writable and accessible to `devops` for all development tools.

---

## ✅ Summary of What's Now Installed

| Component                     | Purpose                               |
| ----------------------------- | ------------------------------------- |
| **SSH Key Auth**              | For secure remote access              |
| **Sudo (NOPASSWD + logging)** | Privilege elevation without passwords |
| **Docker & Compose**          | Containerized app development         |
| **Python 3 & pip**            | Scripting and automation              |
| **Java + Maven**              | Java development environment          |
| **/opt writable**             | Dedicated dev software directory      |
| **Git, curl, etc.**           | Dev-friendly CLI tools                |

---

