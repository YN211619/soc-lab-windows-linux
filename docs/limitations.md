# Limitations — SOC Lab Windows/Linux

## Resources
Host: MacBook Air M4, 16 GB RAM. VM allocations: Wazuh server 8 GB, Windows 2 GB, Ubuntu 2 GB (12 GB in total). There is little spare capacity left on macOS if all three VMs are running simultaneously. VMs will be started as and when required.

## ARM64
The lab uses Apple Silicon and ARM64 VMs. Check the ARM64 compatibility of agent systems and packages. Instructions or tools designed for x86_64 may differ.

## Storage
The server’s virtual disk is configured to 60 GB. The root volume initially had only around 28 GB and became full. LVM and the file system were subsequently expanded to around 56 GB. The space required depends on the volume of events, indexes and retention. The external disk must remain connected and available.

## HTTPS Certificate
The Dashboard is accessed using a self-signed certificate generated for the lab. The browser may display a warning. This configuration is for lab use only; in production, use a trusted certificate.

## Network
The UTM network is virtual. IP addresses may change if DHCP is used. Connectivity depends on the UTM settings and the guest firewalls.

## Scope
Two endpoints are initially planned: Windows 11 Pro and Ubuntu. The host Mac is not monitored. Detections depend on the logs collected, active rules and agent settings. As all core components are on a single VM, there is no high availability.

## Confidentiality
Use non-sensitive test data. Redact any unnecessary private information from captures and reports. Never publish an agent key, enrolment password, secret or configuration file containing credentials.
