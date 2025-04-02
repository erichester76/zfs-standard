# ZFS Server Configuration with Ansible

This repository contains Ansible playbooks to automate the setup and configuration of ZFS-based storage servers. These playbooks manage system initialization, networking, ZFS storage, file sharing (NFS and Samba), and security hardening. They are optimized for servers with NVMe drives for caching and support both static and dynamic inventories.

---

## File Layout

The repository is organized into multiple YAML files, each handling a specific part of the configuration process. Below is the file structure and their purposes:

- **`main.yml`**  
  The primary playbook that orchestrates the configuration by including all other playbooks in the proper sequence.

- **`system_setup.yml`**  
  Performs initial system setup: updates the system, installs required packages, sets up time synchronization (Chrony), configures DNS, and manages repositories.

- **`network.yml`**  
  Configures networking: bonds NICs for redundancy, enables jumbo frames (MTU 9000), and tunes TCP settings for performance.

- **`luks_encryption.yml`**  
  Manages LUKS encryption for disks: encrypts multipath devices and sets up automatic decryption during boot.

- **`zfs.yml`**  
  Configures the ZFS storage pool: creates pools with encrypted multipath devices, sets up datasets, and applies optimizations like `noop` I/O scheduling.

- **`nfs_ganesha.yml`**  
  Sets up NFS-Ganesha for NFS shares: configures exports and tunes worker threads for concurrency.

- **`samba.yml`**  
  Configures Samba for SMB/CIFS sharing: sets up shares, optimizes socket options, and enables asynchronous I/O.

- **`security.yml`**  
  Applies security hardening: disables unnecessary services, secures SSH, configures firewall rules, manages SELinux, and enables audit logging.

- **`gather_multipath.yml`**  
  Retrieves multipath device information dynamically and sets them as facts for use in other playbooks.

- **`vars/main.yml`**  
  Stores variables used across playbooks, such as repository URLs, HTTPS proxy settings, and multipath device lists.

- **`inventory/hosts.ini`**  
  A static inventory file listing servers under the `zfs_servers` group.

- **`inventory/netbox_inventory.yml`**  
  A dynamic inventory file using the Netbox plugin to fetch devices with the "zfs-server" role and "active" status.

---

## Summary of Settings

The playbooks configure the following settings on ZFS servers:

### System Setup
- **Packages**: Updates the system and installs tools like ZFS, NFS-Ganesha, Samba, and `cryptsetup`.
- **Repositories**: Adds custom repos for packages like Ganesha and Samba+.
- **Time Sync**: Configures Chrony with specified NTP servers.
- **DNS**: Sets up DNS servers for name resolution.
- **Proxy**: Configures an HTTPS proxy for restricted environments.

### Networking
- **NIC Bonding**: Combines two NICs for redundancy and bandwidth.
- **Jumbo Frames**: Sets MTU to 9000 for improved throughput.
- **TCP Tuning**: Optimizes TCP buffers and settings.

### Storage (ZFS)
- **LUKS Encryption**: Encrypts multipath devices with automatic boot decryption.
- **ZFS Pool**: Creates a RAID-Z2 pool with spares using encrypted devices.
- **ZFS Dataset**: Configures datasets with `recordsize=64K`, `compression=lz4`, and `atime=off`.
- **I/O Scheduler**: Uses `noop` for multipath devices.
- **ZFS Tuning**: Sets up ARC, L2ARC, and ZIL with NVMe caching and enables prefetching.

### File Sharing
- **NFS-Ganesha**: Configures NFS exports, tunes worker threads by CPU cores, and optimizes metadata caching.
- **Samba**: Sets up SMB shares with optimized sockets, asynchronous I/O, and SMB3 enforcement.

### Security
- **Services**: Disables unused services (e.g., Bluetooth, CUPS).
- **SSH**: Disables root login.
- **Firewall**: Enables `firewalld` with allowed services (NFS, Samba).
- **SELinux**: Sets to permissive mode (configurable).
- **Audit**: Configures `auditd` for command logging.
- **Fail2ban**: Protects against brute-force attacks.

---

## Usage

1. **Clone the Repository**  
   ```bash
   git clone https://github.com/your-repo/zfs-ansible.git
   cd zfs-ansible
