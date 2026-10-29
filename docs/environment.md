# Lab Environment

## 1. Overview

This document describes the host machine, virtualisation platform, virtual machine configuration, and initial system validation for the SOC-Lab project.

The purpose is to establish a documented and reproducible environment before deploying Wazuh.

| Property | Value |
|---|---|
| Project | SOC-Lab — Windows & Linux Security Monitoring |
| Environment type | Local virtualised lab |
| Status | Initial environment setup |
| Last updated | 2026-10-29 |
| Maintainer | YN211619 |

## 2. Host Machine

| Property | Value |
|---|---|
| Hardware | MacBook Air M4 |
| Memory | 16 GB unified memory |
| Host operating system | macOS Tahoe 26.4 |
| CPU architecture | Apple Silicon / ARM64 |
| Available disk space before setup | 512GB |

## 3. Virtualisation Platform

| Property | Value |
|---|---|
| Software | VMware Fusion Pro |
| Version | [Installed version] |
| Host architecture | Apple Silicon |
| VM networking mode | NAT / Shared with my Mac |
| Installation source | [https://support.broadcom.com](https://support.broadcom.com) |

## 4. Virtual Machine: wazuh-server

### 4.1 System Configuration

| Property | configuration |
|---|---|
| VM name | `wazuh-server` | 
| Operating system | Ubuntu Server 24.04 LTS | 
| Operating system architecture | ARM64 | 
| Virtual CPUs | 2 | 
| Memory | 6 GB | 
| Virtual disk | 20 GB | 
| Disk allocation | Dynamic | 
| Network mode | NAT | 
| Hostname | `wazuh-server` | 
| Administrative username | `socadmin` | 
| OpenSSH server | Enabled | Yes |


### 4.2 Installation Media

| Property | Value |
|---|---|
| ISO filename | `ubuntu-24.04.5-live-server-arm64.iso` |
| Download source | [[Official Ubuntu download URL]](https://ubuntu.com/download/server/arm) |
| SHA-256 checksum | f12484464c7d4d73e2167e0a7dbda1e669f8fae9bcf778dc0944937034b320cb |
| Checksum verification | Verified |
| Installation date | 2026-10-28 |

### 4.3 Installed System Information

The following commands were used to validate the installed operating system.

```bash
hostnamectl
uname -m
lsb_release -a
```

**Observed results:**

```text
Static hostname: wazuh-server
[...]
Virtualization: vmware
Operating System: Ubuntu 24.04.5 LTS
Kernel: Linux 6.8.0-147-generic
Architecture: arm64
[...]

aarch64

No LSB modules are available.
Distributor ID: Ubuntu
Description: Ubuntu 24.04.5 LTS
Release: 24.04
Codename: noble

```

## 5. Network Configuration

### 5.1 Network Settings

| Property | Value |
|---|---|
| Network mode | NAT |
| Network interface | enp2s0 |
| Private lab IP address | 172.16.217.129 |
| Default gateway | 172.16.217.2 |
| DNS configuration | 172.16.217.2 |
| Internet access | Working |


### 5.2 Network Validation

Commands used:

```bash
ip -br addr
ip route
ping -c 3 1.1.1.1
getent hosts archive.ubuntu.com
```

| Test | Expected result | Actual result |
|---|---|---|
| Network interface | Interface is present | 172.16.217.129 |
| IP configuration | Valid address assigned | 172.16.217.129 |
| Default route | Route is present | 172.16.217.2 |
| IP connectivity | Requests succeed | 0% packet loss |
| DNS resolution | Hostname resolves to an IP address | [Result] |

**Observations:**



## 6. Resource Validation

Commands used:

```bash
free -h
df -h /
nproc
```

| Property | Observed value |
|---|---|
| Total memory visible to the VM | 5.8Gi |
| Available memory | [5.4Gi |
| Root filesystem size | 9.8G |
| Root filesystem usage | 334Mi |
| Number of available CPUs | 2 |


## 7. System Updates and Remote Administration

Commands used:

```bash
sudo apt update
sudo apt list --upgradable
systemctl status ssh
```

| Check | Result |
|---|---|
| Package repository access | Success |
| System updates | Completed |
| SSH service installed | Yes |
| SSH service status | Active |
| SSH connectivity tested | Yes |

**Notes:**



## 8. Initial Validation Summary

| Validation | Status | Evidence or notes |
|---|---|---|
| VMware Fusion installed | Pass | [Notes] |
| Ubuntu installation completed | Pass | [Notes] |
| ARM64 architecture confirmed | Pass | [Notes] |
| Hostname configured | Pass | [Notes] |
| Network configuration checked | Pass | [Notes] |
| Internet connectivity tested | Pass | [Notes] |
| DNS resolution tested | Pass | [Notes] |
| System updates applied | Pass | [Notes] |
| SSH service checked | Pass | [Notes] |

### Overall Status



## 9. Known Limitations

- The virtual machine shares the host's physical resources.
- Performance may be constrained by the MacBook's 16 GB of unified memory.
- The laboratory is intended for educational and defensive security testing, not production deployment.
- Network reachability depends on the virtual network configuration.
- Resource usage and stability must be re-evaluated after deploying Wazuh.

Additional limitations identified during setup:

- [Limitation 1]
- [Limitation 2]

## 10. Next Steps

Once the environment has been validated:

1. Install the Wazuh central components.
2. Verify the status of the Wazuh services.
3. Access the Wazuh web dashboard.
4. Document the installation and operational checks.
5. Prepare the environment for the first monitored endpoint.

## 11. References

- [Ubuntu Server documentation](https://documentation.ubuntu.com/server/)
- [Ubuntu ARM64 installation images](https://cdimage.ubuntu.com/ubuntu/releases/noble/release/)
- [Wazuh documentation](https://documentation.wazuh.com/current/index.html)
- [Wazuh quickstart guide](https://documentation.wazuh.com/current/quickstart.html)
- [VMware Fusion official support portal](https://support.broadcom.com/)

