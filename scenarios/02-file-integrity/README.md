# Scenario 02 – File Integrity Monitoring (FIM)

**Date completed:** 11 November 2026
**Status:** Validated
**Wazuh:** 4.14.9

## 1. Objective

To configure and test Wazuh’s File Integrity Monitoring (FIM) functionality on two Windows and Linux endpoints.

The objective is to detect in real time:
- The creation of files.
- The modification of files.
- The deletion of files.

The tests were carried out on dedicated test files, without modifying any system files.

## 2. Architecture

| Component | Name | IP Address |
|---|---|---|
| Wazuh Server | wazuh-server | 192.168.64.6 |
| Windows Endpoint | windows11-arm64 | 192.168.64.5 |
| Linux Endpoint | ubuntu-linux-arm64 | 192.168.64.4 |

Wazuh agents:
- Windows: ID `001`
- Linux: ID `002`

## 3. FIM Configuration

A specific directory was created on each endpoint.

| Endpoint | Monitored directory |
|---|---|
| Linux | `/home/ubuntusoc/fim-lab` |
| Windows | `C:\FIM-Lab` |

### Linux Configuration

File: `/var/ossec/etc/ossec.conf`

Line added to the existing `<syscheck>` block:

```xml
<directories realtime="yes">/home/ubuntusoc/fim-lab</directories>
```

### Windows Configuration

File: `C:\Program Files (x86)\ossec-agent\ossec.conf`

Line added to the existing `<syscheck>` block:

```xml
<directories realtime="yes">C:\FIM-Lab</directories>
```

### Validation and restart

The Linux configuration was tested with:

```bash
sudo /var/ossec/bin/wazuh-syscheckd -t
```

Return code obtained: `0`.

Both agent services were then restarted and checked:

- Linux: `wazuh-agent` – `active (running)`.
- Windows: `WazuhSvc` – `Running`.

The Wazuh server services were also active.

## 4. Tests and results

### Test 1 – Creating a file

**Linux action:**

```bash
echo ‘FIM test - Linux’ > /home/ubuntusoc/fim-lab/test-fim.txt
```

**Windows action:**

```powershell
‘FIM test - Windows’ | Out-File -FilePath ‘C:\FIM-Lab\test-fim.txt’
```

**Results:**

| Field | Linux | Windows |
|---|---|---|
| Rule ID | `554` | `554` |
| Description | File added to the system | File added to the system |
| Level | 5 | 5 |
| Event | `added` | `added` |
| Mode | `realtime` | `realtime` |
| Path | `/home/ubuntusoc/fim-lab/test-fim.txt` | `c:\fim-lab\test-fim.txt` |

The alerts contained the MD5, SHA-1 and SHA-256 hashes of the files created.

**DQL query:**

```text
rule.groups: ‘syscheck’
AND syscheck.event: ‘added’
AND (agent.name: ‘ubuntu-linux-arm64’ OR agent.name: ‘windows11-arm64’)
```

**Conclusion:** the creation of the files was detected in real time on both endpoints.

### Test 2 – Modifying a file

**Linux action:**

```bash
echo ‘FIM modification - Linux’ >> /home/ubuntusoc/fim-lab/test-fim.txt
```

**Windows action:**

```powershell
Add-Content -Path ‘C:\FIM-Lab\test-fim.txt’ -Value ‘FIM Modification – Windows’
```

**Results:**

| Field | Linux | Windows |
|---|---|---|
| Rule ID | `550` | `550` |
| Description | Integrity checksum changed | Integrity checksum changed |
| Level | 7 | 7 |
| Event | `modified` | `modified` |
| Mode | `realtime` | `realtime` |
| Size before | 17 bytes | 40 bytes |
| Size after | 42 bytes | 96 bytes |

The alerts indicated the following modified attributes:

```text
size, mtime, md5, sha1, sha256
```

Examples of SHA-256 changes:

- Linux: `43b098f7303c3ed5e24d2dcd87084cdf5f47268f13d8d5f679fc7c1d34f9c618` → `1d7068b2528a676f74b67a022f64ae9399a26 831b938ce05085dbd3607ff6604`
- Windows: `11c124ef0a9416e581771784a2e2af598a234d7e7462c9b58f3365b31eaf1c8e` → `11f8d83c0622f17f8a9a1499ddd34e4ebab16f7 1e0963eceb758fdc7a32e0289`

**DQL query:**

```text
rule.groups: ‘syscheck’
AND syscheck.event: ‘modified’
AND (agent.name: ‘ubuntu-linux-arm64’ OR agent.name: ‘windows11-arm64’)
```

**MITRE ATT&CK match observed:**
- Technique: `T1565.001`
- Name: Stored Data Manipulation
- Tactic: Impact

**Conclusion:** Wazuh detected the modifications and recorded the changes to the cryptographic fingerprints on both endpoints.

### Test 3 – Deleting a file

**Linux action:**

```bash
rm /home/ubuntusoc/fim-lab/test-fim.txt
```

**Windows action:**

```powershell
Remove-Item ‘C:\FIM-Lab\test-fim.txt’
```

**Results:**

| Field | Linux | Windows |
|---|---|---|
| Rule ID | `553` | `553` |
| Description | File deleted | File deleted |
| Level | 7 | 7 |
| Event | `deleted` | `deleted` |
| Mode | `realtime` | `realtime` |

Both alerts indicate:

```text
File deleted.
Mode: realtime
```

**DQL query:**

```text
rule.groups: ‘syscheck’
AND syscheck.event: ‘deleted’
AND (agent.name: ‘ubuntu-linux-arm64’ OR agent.name: ‘windows11-arm64’)
```

**MITRE ATT&CK mappings observed:**
- `T1070.004` – File Deletion.
- `T1485` – Data Destruction.

These correlations are technical classifications of events. They do not prove that a malicious activity has taken place.

**Conclusion:** The deletion of the files was detected and the previously known metadata for the files was retained in the alerts.

## 5. Summary of Wazuh rules

| Rule ID | Description | Level | Event |
|---|---|---:|---|
| `554` | File added to the system | 5 | Creation |
| `550` | Integrity checksum changed | 7 | Modification |
| `553` | File deleted | 7 | Deletion |

All three rules were observed on Windows and Linux.

## 6. Technical skills utilised

- Configuring Syscheck in `ossec.conf`.
- Enabling real-time monitoring.
- Verifying the Wazuh configuration.
- Restarting and checking the agents.
- Viewing alerts in the Wazuh Dashboard.
- Using Discover and the DQL language.
- Analysing creation, modification and deletion events.
- Comparing MD5, SHA-1 and SHA-256 hashes.
- Analysing file metadata.
- Initial correlation of events with MITRE ATT&CK.

## 7. Limitations and precautions

- The tests are controlled and do not constitute evidence of intrusion.
- Monitoring is limited to the two configured laboratory directories.
- FIM events alone do not allow the identity of the person who made a modification or deletion to be determined.
- A modification alert does not necessarily indicate malicious activity.
- Cryptographic hashes can detect a change in content, but do not in themselves prove the origin of that change.
- This scenario validates real-time monitoring, but does not constitute a comprehensive audit of all files on the systems.

## 8. Conclusion

Scenario 02 – File Integrity Monitoring has been validated.

Wazuh correctly detected the three operations tested on the two endpoints:
- Creation: rule `554`.
- Modification: rule `550`.
- Deletion: rule `553`.

The events were transmitted to the server and viewed in the Wazuh Dashboard via Discover and DQL.

**Result:** real-time file integrity monitoring works on the laboratory’s Windows and Linux endpoints.
