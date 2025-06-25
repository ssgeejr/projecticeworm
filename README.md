
# Project Iceworm: VMware Debian Development Environment Setup

## 🔧 Overview
This guide walks you through setting up a full-featured software development environment using VMware Workstation on a Windows host. It creates **three Debian virtual machines** named:
- `admin` – primary control and configuration node
- `alpha` – development node A
- `beta` – development node B

All nodes are identically configured for real-world DevOps training.

---

## 🖥️ Windows Host Setup

### 1. Install PowerShell (Modern Version)
1. Open **Microsoft Store** → Search "PowerShell"
2. Install the latest stable or preview version.
3. Launch PowerShell from the Start Menu.

### 2. Create SSH Directory and Config File
Open PowerShell and run:
```powershell
mkdir -Force $HOME\.ssh
New-Item -Path $HOME\.ssh\config -ItemType File -Force
```

### 3. Install Git
1. Download from: https://git-scm.com/
2. Run the installer → Accept defaults.

### 4. Generate SSH Keys (Repeat for Each VM)
Run in PowerShell:
```powershell
ssh-keygen -t rsa -b 2048 -f $HOME\.ssh\devops.admin
ssh-keygen -t rsa -b 2048 -f $HOME\.ssh\devops.alpha
ssh-keygen -t rsa -b 2048 -f $HOME\.ssh\devops.beta
```

### 5. Configure GitHub Access (Optional)
1. Run: `cat $HOME\.ssh\devops.admin.pub`
2. Copy the output to GitHub → Settings → SSH Keys → New Key
3. Edit your config file:
```plaintext
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/devops.admin
```

### 6. Install Required Software
- **VMware Workstation Player**: [Download](https://www.vmware.com/products/workstation-player.html)
- **Python 3**: [Download](https://www.python.org/)
- **OpenJDK**: [Download](https://jdk.java.net/)
- **Docker Desktop**: [Download](https://www.docker.com/products/docker-desktop/)

### Install PyCharm Community Edition (Windows)

1. Go to the official download page:  
   [https://www.jetbrains.com/pycharm/download/#section=windows](https://www.jetbrains.com/pycharm/download/#section=windows)

2. Click **Download** under "Community Edition".

3. Run the installer and follow these steps:
   - Check **"Add launchers dir to the PATH"** (recommended).
   - Optionally check **"Create Desktop Shortcut"**.
   - Click **Install** and wait for the setup to complete.

4. Once installed, you can launch PyCharm from the Start Menu or desktop icon.

> ✅ PyCharm will prompt you to import settings (choose "Do not import settings" if this is your first install), then guide you through the initial setup wizard.

---

## 📦 Debian VM Installation (VMware)

### Download Debian ISO
- [Debian DVD ISO (amd64)](https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/)
  - Download: `debian-XX.X.X-amd64-DVD-1.iso`

### Create VMs in VMware (Repeat 3 Times)

For each VM: `admin`, `alpha`, `beta`:

1. Open VMware Player → **Create New VM**
2. Use ISO file downloaded above
3. Set OS: Linux → Debian 11.x or later
4. Name VM (`admin`, `alpha`, or `beta`)
5. Disk: 20 GB, single file
6. Customize Hardware:
   - Memory: 8192 MB
   - CPUs: 1–2
   - Network: NAT or Bridged
7. Click Finish

---

## 🧱 Debian Post-Install Configuration (All VMs)

### 1. Create User `devops`
```bash
adduser devops
usermod -aG sudo devops
```

### 2. Set Up SSH Access
```bash
mkdir -p /home/devops/.ssh
chmod 700 /home/devops/.ssh
touch /home/devops/.ssh/authorized_keys
chmod 600 /home/devops/.ssh/authorized_keys
chown -R devops:devops /home/devops/.ssh
```

### 3. Copy Public Key (From Host)
On **host**:
```powershell
scp $HOME\.ssh\devops.admin.pub devops@<admin-IP>:/tmp/
scp $HOME\.ssh\devops.alpha.pub devops@<alpha-IP>:/tmp/
scp $HOME\.ssh\devops.beta.pub devops@<beta-IP>:/tmp/
```

On each **VM**:
```bash
cat /tmp/devops.*.pub >> /home/devops/.ssh/authorized_keys
rm /tmp/devops.*.pub
chown devops:devops /home/devops/.ssh/authorized_keys
```

### 4. Configure Passwordless Sudo with Logging
```bash
echo "%devops ALL=(ALL) NOPASSWD: LOG_INPUT: ALL" | sudo tee /etc/sudoers.d/devops
chmod 440 /etc/sudoers.d/devops
```

---

## 🧰 Install Dev Tools on Each VM

### Update System
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Docker and Docker Compose
```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker devops
```

### Install Python, pip, Java, Maven
```bash
sudo apt install -y python3 python3-pip openjdk-17-jdk maven
```

### Developer Utilities
```bash
sudo apt install -y git curl wget unzip htop net-tools tmux make build-essential
```

---

## 🗂️ Configure `/opt` for Group Access

```bash
sudo chgrp -R devops /opt
sudo chmod -R g+wrx /opt
```

---

## ✅ Final Checklist

| Configuration | Status |
|---------------|--------|
| devops user created | ✅ |
| SSH keys installed | ✅ |
| Passwordless sudo | ✅ |
| Docker + Compose | ✅ |
| Java + Maven + Python | ✅ |
| Git, curl, etc. | ✅ |
| Writable `/opt` | ✅ |

---

## 🚀 Next Steps

- Snapshot these base images before modifying
- Automate future setup with **Ansible** and **Terraform**
- Add CI/CD stack (Jenkins, SonarQube)
- Practice provisioning with containerized dev pipelines
