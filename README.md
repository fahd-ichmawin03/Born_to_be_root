*This project has been created as part of the 42 curriculum by fichmawi.*

## Description

Born2beroot is a system administration project that introduces the basics of virtualization, operating system installation, and server configuration. The goal is to create a virtual machine following strict guidelines, implementing security policies, partitioning schemes, and system services.

This project involves setting up a Debian-based server with encrypted partitions using LVM, configuring SSH, implementing a firewall, and establishing a password policy to ensure system security.

## Instructions

### Prerequisites

- VirtualBox 
- Debian 13 ISO image
- At least 12GB of disk space for the virtual machine
- 1GB of RAM allocated to the VM

### Installation Steps

1. **Create a new Virtual Machine in VirtualBox:**
   - Name: Born2beroot
   - Type: Linux
   - Version: Debian (64-bit)
   - Memory: 1024 MB
   - Hard disk: 12 GB (VDI format)

2. **Attach the Debian ISO:**
   - Settings → Storage → Controller: IDE
   - Add optical drive → Select debian-13.3.0-amd64-netinst.iso

3. **Boot and Install Debian:**
   - Start the VM and boot from the ISO
   - Follow the installation
   - Choose manual partitioning (see Partitioning Scheme below)
   - Set up encrypted LVM partitions
   - Configure hostname: 'fichmawi42'

4. **Partitioning Scheme:**
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

5. **Post-Installation Configuration:**
   - Install and configure sudo
   - Set up SSH on port 4242
   - Configure UFW firewall
   - Implement password policy
   - Create users and groups
   - Set up monitoring script


### Compilation / Execution

```bash
# Start VirtualBox
VirtualBox &

# Or from command line
VBoxManage startvm "Born2beroot" --type headless

# Connect via SSH (from host machine)
ssh fichmawi@localhost -p 4242
```

### Verification Commands
```bash
# Check OS version
cat /etc/os-release

# Check partitions
lsblk

# Check sudo is installed
sudo -V

# Check UFW status
sudo ufw status

# Check SSH status
sudo systemctl status ssh

# Check the monitoring script automation
sudo crontab -l

# Check password policy
sudo chage -l user
```

## Resources

### Documentation
- [Debian Official Documentation](https://www.debian.org/doc/)
- [VirtualBox User Manual](https://www.virtualbox.org/manual/)
- [Linux LVM HOWTO](https://tldp.org/HOWTO/LVM-HOWTO/)
- [UFW Documentation](https://help.ubuntu.com/community/UFW)


### AI Usage

- **Conceptual explanations**: Understanding hypervisors, virtualization concepts, LVM architecture, ISO file format.
- **clarification**: Detailed explanation of partition.
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

- 1. User Roles
The system differentiates between different types of access to ensure a secure environment:
* **Root Account**: The superuser account, used strictly for critical system administration.
* **Primary User**: A non-privileged account created during installation for standard operations.

- 2. Group Organization
Users are categorized into groups to manage permissions effectively:
* **sudo Group**: Contains users authorized to perform administrative tasks with elevated privileges.
* **user42 Group**: A specific group created for the project.

- 3. Account Management Strategy
* **Privilege Separation**: Users do not have administrative rights by default.
* **Auditability**: By using individual accounts instead of a shared root login, all system changes can be traced back to a specific user.