# Born2beroot - System Administration Project

*This project has been created as part of the 42 curriculum by fichmawi (fahd ichmawin).*

## Description

Born2beroot is a comprehensive system administration project that introduces virtualization, operating system installation, and server configuration fundamentals. This project involves creating a secure virtual machine with encrypted partitions using LVM, implementing strict security policies, configuring SSH and firewall services, and establishing robust password and sudo policies.

Through this project, I gained hands-on experience with:
- Virtual machine setup and management
- Disk partitioning with LVM and encryption (LUKS)
- User and group management
- Security hardening (SSH, UFW, AppArmor)
- System monitoring and automation (cron jobs)
- Server administration best practices

## Table of Contents

- [Instructions](#instructions)
  - [Prerequisites](#prerequisites)
  - [Installation Steps](#installation-steps)
  - [Compilation / Execution](#compilation--execution)
  - [Verification Commands](#verification-commands)
- [Detailed Configuration](#detailed-configuration)
  - [Sudo Configuration](#sudo-configuration)
  - [Password Policy](#password-policy)
  - [SSH Configuration](#ssh-configuration)
  - [UFW Firewall](#ufw-firewall)
  - [Monitoring Script](#monitoring-script)
  - [Cron Jobs](#cron-jobs)
- [User Management](#user-management)
- [Bonus Part](#bonus-part)
- [Evaluation Preparation](#evaluation-preparation)
- [Resources](#resources)
- [AI Usage](#ai-usage)
- [Project Description](#project-description)

## Instructions

### Prerequisites

- **VirtualBox** 
- **Debian 13 ISO image** 
- **12GB of disk space** for the virtual machine
- **1GB of RAM** allocated to the VM
- Basic understanding of Linux commands

### Installation Steps

#### 1. Create a new Virtual Machine in VirtualBox

```
Name: Born2beroot
Type: Linux
Version: Debian (64-bit)
Memory: 1024 MB
Hard disk: 12 GB (VDI format, Dynamically allocated)
```

#### 2. Attach the Debian ISO

```
Settings → Storage → Controller: IDE
Add optical drive → Select debian-13.3.0-amd64-netinst.iso
```

#### 3. Boot and Install Debian

- Start the VM and boot from the ISO
- Choose **Install** (not Graphical Install)
- Language, Location
- **Hostname**: `fichmawi42` 
- **Domain name**: Leave empty
- **Root password**: Choose a strong password (remember it!)
- **User account**: Create your main user (fichmawi)
- Choose manual partitioning (see Partitioning Scheme below)

#### 4. Partitioning Scheme

**Important**: Use encrypted LVM partitions as shown below:

```
NAME                        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda                           8:0    0   12G  0 disk  
├─sda1                        8:1    0  791M  0 part  /boot
├─sda2                        8:2    0    1K  0 part  
└─sda5                        8:5    0 11.2G  0 part  
  └─sda5_crypt              254:0    0 11.2G  0 crypt 
    ├─fichmawi42--vg-root   254:1    0  7.1G  0 lvm   /
    ├─fichmawi42--vg-swap_1 254:2    0  620M  0 lvm   [SWAP]
    └─fichmawi42--vg-home   254:3    0  3.4G  0 lvm   /home
sr0                          11:0    1 1024M  0 rom 
```

**Partitioning Steps**:
1. Select Manual partitioning
2. Create `/boot` partition (791M, Primary, ext4)
3. Create encrypted volume on remaining space
4. Configure LVM with volume group `fichmawi42-vg`
5. Create logical volumes: `root` (7.1G), `swap` (620M), `home` (3.4G)

#### 5. Post-Installation Configuration

- Install and configure **sudo**
- Set up **SSH** on port 4242
- Configure **UFW** firewall
- Implement **password policy**
- Create users and groups (`sudo`, `user42`)
- Set up **monitoring script**
- Configure **crontab** for automated monitoring

### Compilation / Execution

This project doesn't require compilation. To run the virtual machine:

```bash
# Start VirtualBox GUI
VirtualBox &

# Connect via SSH from host machine
ssh fichmawi@localhost -p 4242
```

### Verification Commands

```bash
# Check OS version
cat /etc/os-release
lsb_release -a

# Check partitions
lsblk

# Check sudo is installed
sudo -V

# Check UFW status
sudo ufw status numbered

# Check SSH status
sudo systemctl status ssh

# Check the monitoring script automation
sudo crontab -l

# Check password policy for a user
sudo chage -l username

# Check hostname
hostnamectl

# Check user groups
getent group sudo
getent group user42

# Check AppArmor status
sudo aa-status

# View sudo logs
sudo cat /var/log/sudo/sudo.log
```

#### Installation

```bash
# Switch to root
su -

# Update package lists
apt update 

# Install sudo
apt install sudo

# Add user to sudo group
usermod -aG sudo fichmawi

# Verify user is in sudo group
getent group sudo
```

#### Sudo Policy Configuration


```
# Sudo security policies
Defaults	passwd_tries=3
Defaults	badpass_message="Wrong password. Try again!"
Defaults	logfile="/var/log/sudo/sudo.log"
Defaults	log_input
Defaults	log_output
Defaults	requiretty
```

**Explanation**:
- `passwd_tries=3`: Limit authentication attempts to 3
- `badpass_message`: Custom error message for wrong password
- `logfile`: Log all sudo commands to this file
- `log_input`: Log command input
- `log_output`: Log command output
- `requiretty`: Require a TTY for sudo (prevents automated attacks)

#### Create Sudo Log Directory

```bash
sudo mkdir -p /var/log/sudo
sudo touch /var/log/sudo/sudo.log
```
## Detailed Configuration
### Sudo Configuration
### Password Policy

#### Configuration Files

**1. Password Aging Controls** (`/etc/login.defs`):

```bash
sudo nano /etc/login.defs
```

Modify these values:

```
PASS_MAX_DAYS   30
PASS_MIN_DAYS   2
PASS_WARN_AGE   7
```

**Explanation**:
- `PASS_MAX_DAYS`: Password expires after 30 days
- `PASS_MIN_DAYS`: Minimum 2 days between password changes
- `PASS_WARN_AGE`: Warn user 7 days before expiration

**2. Password Quality Requirements** (`/etc/pam.d/common-password`):

```bash
sudo apt install libpam-pwquality -y
sudo nano /etc/pam.d/common-password
```

Find the line with `pam_pwquality.so` and modify it:

```
password requisite pam_pwquality.so retry=3 minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```

**Explanation**:
- `retry=3`: Allow 3 attempts to set a valid password
- `minlen=10`: Minimum password length of 10 characters
- `ucredit=-1`: At least 1 uppercase letter required
- `lcredit=-1`: At least 1 lowercase letter required
- `dcredit=-1`: At least 1 digit required
- `maxrepeat=3`: Maximum 3 consecutive identical characters
- `reject_username`: Password cannot contain username
- `difok=7`: At least 7 characters different from old password
- `enforce_for_root`: Apply policy to root user too

#### Apply Password Policy to Existing Users

```bash
# For regular users
sudo chage -M 30 -m 2 -W 7 fichmawi

# For root
sudo chage -M 30 -m 2 -W 7 root

# Check password aging for a user
sudo chage -l fichmawi
```

#### Test Password Policy

```bash
# Try creating a user with a weak password (should fail)
sudo adduser test_user

# Try setting a password that violates policy
# It should reject weak passwords

# Remove test user
sudo userdel test_user
```

### SSH Configuration

#### Installation

```bash
sudo apt update
sudo apt install openssh-server
```

#### SSH Configuration File

Edit the SSH configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Modify these settings:

```
Port 4242
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication yes
```

**Explanation**:
- `Port 4242`: SSH listens on custom port 4242 
- `PermitRootLogin no`: Disable direct root login for security
- `PasswordAuthentication yes`: Allow password authentication

#### Restart SSH Service

```bash
sudo systemctl restart ssh
sudo systemctl status ssh
```

#### Test SSH Connection

From your host machine:

```bash
ssh fichmawi@localhost -p 4242
```

### UFW Firewall

#### Installation and Configuration

```bash
# Install UFW
sudo apt install ufw

# Enable UFW
sudo ufw enable

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH on port 4242
sudo ufw allow 4242

# Check status
sudo ufw status numbered
```

**Output should show**:

```
Status: active

To                         Action      From
--                         ------      ----
4242                       ALLOW       Anywhere
```

#### UFW Management Commands

```bash
# Add a rule
sudo ufw allow 8080

# Delete a rule by number
sudo ufw delete 2

# Disable UFW
sudo ufw disable

# Reset UFW (remove all rules)
sudo ufw reset
```

### Monitoring Script

#### Script Location and Creation

Create the monitoring script:

```bash
sudo nano /usr/local/bin/monitoring.sh
```

#### Monitoring Script Content

```bash
#!/bin/bash

# Architecture and Kernel
arch=$(uname -a)

# Physical CPUs
pcpu=$(grep "physical id" /proc/cpuinfo | sort -u | wc -l)

# Virtual CPUs
vcpu=$(grep "processor" /proc/cpuinfo | wc -l)

# RAM Usage
ram_total=$(free -m | awk '/Mem:/ {print $2}')
ram_used=$(free -m | awk '/Mem:/ {print $3}')
ram_percent=$(free -m | awk '/Mem:/ {printf("%.2f"), $3/$2*100}')

# Disk Usage
disk_total=$(df -h --total | awk '/total/ {print $2}')
disk_used=$(df -h --total | awk '/total/ {print $3}')
disk_percent=$(df -h --total | awk '/total/ {print $5}')

# CPU Load
cpu_load=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)

# Last Boot
last_boot=$(who -b | awk '{print $3, $4}')

# LVM Use
lvm_use=$(if [ $(lsblk | grep "lvm" | wc -l) -gt 0 ]; then echo "yes"; else echo "no"; fi)

# TCP Connections
tcp_connections=$(ss -t | grep ESTAB | wc -l)

# User Log
user_log=$(who | wc -l)

# Network
ip_addr=$(hostname -I | awk '{print $1}')
mac_addr=$(ip link show | grep "ether" | awk '{print $2}')

# Sudo Commands
sudo_cmds=$(journalctl _COMM=sudo | grep COMMAND | wc -l)

# Display Information
wall "	
	#Architecture: $arch
	#CPU physical: $pcpu
	#vCPU: $vcpu
	#Memory Usage: $ram_used/${ram_total}MB ($ram_percent%)
	#Disk Usage: $disk_used/${disk_total} ($disk_percent)
	#CPU load: $cpu_load%
	#Last boot: $last_boot
	#LVM use: $lvm_use
	#Connections TCP: $tcp_connections ESTABLISHED
	#User log: $user_log
	#Network: IP $ip_addr ($mac_addr)
	#Sudo: $sudo_cmds cmd
"
```

#### Make Script Executable

```bash
sudo chmod +x /usr/local/bin/monitoring.sh
```

#### Test the Script

```bash
sudo /usr/local/bin/monitoring.sh
```

### Cron Jobs

#### Configure Crontab

Edit root's crontab:

```bash
sudo crontab -u root -e
```

Add this line to run the script every 10 minutes:

```
*/10 * * * * /usr/local/bin/monitoring.sh
```

**Note**: This runs at every 10-minute mark (00, 10, 20, 30, 40, 50).


#### View Scheduled Jobs

```bash
sudo crontab -l
```

#### Stop/Start Cron

```bash
# Stop cron service
sudo systemctl stop cron

# Start cron service
sudo systemctl start cron

# Disable cron at boot
sudo systemctl disable cron
```

## User Management

### User Roles

The system differentiates between different types of access:

- **Root Account**: Superuser account for critical system administration
- **Primary User** (fichmawi): Non-privileged account for standard operations

### Group Organization

Users are categorized into groups for permission management:

- **sudo group**: Users authorized to perform administrative tasks
- **user42 group**: Project-specific group

### User Management Commands

```bash
# Create new user
sudo adduser username

# Delete user and home directory
sudo userdel username

# Add user to group
sudo usermod -aG groupname username

# Remove user from group
sudo deluser username groupname

# Create new group
sudo groupadd groupname

# Delete group
sudo groupdel groupname

# Check user's groups
groups username

# List all users in a group
getent group groupname

# Check user information
id username

# List all users
cat /etc/passwd

# Change user password
sudo passwd username
```

### Hostname Management

```bash
# View current hostname
hostnamectl

# Change hostname
sudo hostnamectl set-hostname new_hostname

# Edit hosts file
sudo nano /etc/hosts
# Change the line: 127.0.1.1 old_hostname to new_hostname

# Restart to apply changes (or just restart shell)
sudo reboot
```

## Resources

### Documentation
- [Debian Official Documentation](https://www.debian.org/doc/)
- [VirtualBox User Manual](https://www.virtualbox.org/manual/)
- [Linux LVM HOWTO](https://tldp.org/HOWTO/LVM-HOWTO/)
- [UFW Documentation](https://help.ubuntu.com/community/UFW)
- [SSH Security Best Practices](https://www.ssh.com/academy/ssh/hardening)


### AI Usage

AI was used for:
- **Conceptual explanations**: Understanding hypervisors, virtualization concepts, LVM architecture, LUKS encryption, device files (/dev/sda, /dev/sr0), ISO file format, MBR partitioning
- **Comparison research**: Analysis of Debian vs Rocky Linux, AppArmor vs SELinux, UFW vs firewalld, VirtualBox vs UTM

## Project Description

### Debian vs Rocky Linux

**Choice: Debian**

#### Debian

**Pros:**
- Excellent package management with APT
- Large community and extensive documentation
- More beginner-friendly with better community support for newcomers
- Faster learning curve for Linux beginners

**Cons:**
- Packages may be older than cutting-edge distributions
- Less focused on enterprise features compared to Rocky
- Shorter support lifecycle (~5 years vs 10 years for Rocky)
- No official commercial support options

#### Rocky Linux

**Pros:**
- 10 years of support
- Enterprise-grade stability optimized for business workloads
- SELinux enabled by default for stronger mandatory access control

**Cons:**
- Steeper learning curve for beginners
- Smaller community compared to Debian
- More complex initial setup and configuration

**Why Debian for Born2beroot:**
Debian was chosen for this project due to its legendary stability, beginner-friendly nature, excellent documentation, and suitability for learning Linux system administration fundamentals. The APT package manager is more intuitive, and the large community provides better support for newcomers.

### AppArmor vs SELinux

**Choice: AppArmor** (default on Debian)

**AppArmor:**
- **Pros:** Simpler to configure, easier learning curve, path-based access control
- **Cons:** Less granular than SELinux

**SELinux:**
- **Pros:** More powerful and fine-grained security policies, label-based security
- **Cons:** Steeper learning curve, more complex configuration

**Why AppArmor:** For this project, AppArmor provides sufficient security with easier management, making it ideal for learning system administration basics.

### UFW vs firewalld

**Choice: UFW** (Uncomplicated Firewall)

**UFW:**
- **Pros:** Simple syntax, easy to learn, perfect for basic firewall rules, iptables frontend
- **Cons:** Less advanced features than firewalld

**firewalld:**
- **Pros:** Dynamic firewall management, zone-based configuration, more enterprise-oriented
- **Cons:** More complex for beginners, primarily used in RHEL-based systems

**Why UFW:** Its simplicity and straightforward commands make it perfect for learning firewall basics without overwhelming complexity.


### VirtualBox vs UTM
**Choice: VirtualBox**

* **VirtualBox**: A widely used, cross-platform hypervisor that is excellent for x86 architecture.
* **UTM**: A lightweight virtualization tool optimized for macOS and Apple Silicon (M1/M2/M3), utilizing QEMU for high performance on ARM chips

**Why VirtualBox:** Industry-standard virtualization platform with excellent cross-platform support and comprehensive documentation.

### Design Choices

**Partitioning:**
- Separate `/boot` partition (unencrypted) for bootloader access
- Encrypted LVM for security of all user data and system files
- Separate `/home` partition for easier backup and system reinstallation

**Security Policies:**
- Password expiration every 30 days
- Minimum password age of 2 days
- Password complexity requirements (uppercase, lowercase, digits)
- Sudo authentication limited to 3 attempts
- Sudo commands logged for auditing

**Services:**
- SSH on custom port 4242 
- Root login disabled via SSH
- UFW firewall allowing only port 4242
- AppArmor enabled for mandatory access control

**User Management**

#### 1. User Roles
The system differentiates between different types of access to ensure a secure environment:
* **Root Account**: The superuser account, used strictly for critical system administration.
* **Primary User**: A non-privileged account created during installation for standard operations.

#### 2. Group Organization
Users are categorized into groups to manage permissions effectively:
* **sudo Group**: Contains users authorized to perform administrative tasks with elevated privileges.
* **user42 Group**: A specific group created for the project.

#### 3. Account Management Strategy
* **Privilege Separation**: Users do not have administrative rights by default.
* **Auditability**: By using individual accounts instead of a shared root login, all system changes can be traced back to a specific user.