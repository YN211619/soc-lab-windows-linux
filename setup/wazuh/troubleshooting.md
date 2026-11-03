# Wazuh Troubleshooting — issues encountered

This document details the symptoms, confirmed causes and fixes applied during installation. 

## 1. Root volume full

### Symptoms
`df -h /` showed approximately 28 GB total, 27 GB used and only 84 MB free (later 371 MB). `systemd-journald` reported `No space left on device`.

### Diagnosis
Measurements showed approximately 19 GB in `/var`, of which approximately 19 GB was in `/var/ossec`, and approximately 3.4 GB in `/usr`. Inodes were utilised at approximately 7 per cent. The virtual disk `vda` was configured to 60 GB, but the root volume was only using approximately 28 GB. Details of the subdirectories within `/var/ossec` were not retained; the precise cause of this space usage is therefore not confirmed.

### Resolution
The LVM logical volume and the filesystem were extended. Following the operation, `df -h /` showed approximately 56 GB in total, 27 GB used and 28 GB available (49 per cent used).

Command used after checking the volumes:

```bash
sudo lvextend -l +100%FREE -r /dev/ubuntu-vg/ubuntu-lv
```

Only use this if `sudo vgs`, `sudo lvs` and `lsblk` confirm the volume name and available free space. The path may vary.

## 2. Indexer fails after reboot

### Symptom
The log contained:

```text
Error opening log file “/var/log/wazuh-indexer/gc.log”: No such file or directory
Error: Could not create the Java Virtual Machine.
```
### Cause confirmed
The log directory expected by Java was missing. The error creating the JVM was a consequence of the inability to open the log file.

### Fix applied

```bash
sudo install -d -o wazuh-indexer -g wazuh-indexer -m 0750 /var/log/wazuh-indexer
sudo systemctl start wazuh-indexer
```

Manager and Filebeat were then restarted:

```bash
sudo systemctl start wazuh-manager
sudo systemctl start filebeat
sudo systemctl is-active wazuh-indexer wazuh-manager filebeat
```

All three services returned `active`.

## 3. First Dashboard installation failed

The wizard displayed:

```text
Wazuh dashboard installation failed.
Removing existing Wazuh installation.
Installation cleaned.
```

After expanding the storage and restarting the other three services, the standard Dashboard installation was restarted:

```bash
sudo bash wazuh-install.sh --wazuh-dashboard dashboard
```
All four services returned `active`.

The exact cause of the initial Dashboard failure has not been definitively established.

## 4. Diagnostic commands

```bash
sudo systemctl is-active wazuh-indexer wazuh-manager filebeat wazuh-dashboard
df -h /
df -i /
sudo vgs
sudo lvs
lsblk
sudo journalctl -u wazuh-indexer -b --no-pager -n 100
```

