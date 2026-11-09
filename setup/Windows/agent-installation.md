# Installing the Wazuh agent — Windows 11 Pro ARM64

**Validation date:** 5 November 2026
**Agent version:** 4.14.9
**Agent name:** `windows11-arm64`

## 1. Objective

To install and register the Wazuh agent on the Windows 11 Pro VM in order to forward security events to the laboratory’s Wazuh server.

## 2. Environment

| Item | Value |
|---|---|
| System | Windows 11 Pro |
| Windows version detected | `10.0.26200` |
| VM architecture | ARM64 (`ARM64-based PC`) |
| Computer name as seen in Windows events | `WIN_PC1` |
| IPv4 address of the Windows VM | `192.168.64.5` |
| Wazuh server | Ubuntu Server 24.04 LTS |
| Wazuh server IPv4 address | `192.168.64.6` |
| Wazuh Agent version | `4.14.9` |
| Agent name | `windows11-arm64` |
| Agent ID recorded on the Manager side | `001` |

## 3. ARM64 Compatibility Note

The official list of Wazuh 4.14.9 packages includes a Windows MSI installer for Windows XP or later systems, listed as a 32/64-bit package. It does not include a separate Windows ARM64 MSI.

In this lab, however, the installation of the `wazuh-agent-4.14.9-1.msi` file was successful on the Windows 11 ARM64 VM. The service started, the agent connected to the Manager and a Windows event 4625 was detected. This result confirms the behaviour observed in this environment; it does not, however, constitute a general statement of official support for all Windows ARM64 configurations.

Reference: [Wazuh — Packages list](https://documentation.wazuh.com/current/installation-guide/packages-list.html).

## 4. Check network connectivity

From PowerShell running as an administrator, test the agent’s ports to the server:

```powershell
Test-NetConnection 192.168.64.6 -Port 1514
Test-NetConnection 192.168.64.6 -Port 1515
```

**Result observed during installation:** `TcpTestSucceeded: True` for both ports.

- TCP/1514: agent–Manager communication.
- TCP/1515: agent enrolment service.

Port availability confirms TCP connectivity from Windows.

## 5. Download and install the agent

Open PowerShell **as an administrator**, then download the official Wazuh 4.14.9 MSI:

```powershell
$msi = ‘$env:TEMP\wazuh-agent-4.14.9-1.msi’

Invoke-WebRequest `
  -Uri ‘https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.9-1.msi’ `
  -OutFile $msi
```

Run the silent installation, specifying the Manager’s address, the registration server’s address and the agent’s name:

```powershell
$arguments = ‘/i `’$msi`‘ /q WAZUH_MANAGER=192.168.64.6 WAZUH_REGISTRATION_SERVER=192.168.64.6 WAZUH_AGENT_NAME=windows11-arm64 /L*V `’$env:TEMP\ wazuh-agent-install.log`‘’

$process = Start-Process msiexec.exe -ArgumentList $arguments -Wait -PassThru
$process.ExitCode
```

**Result observed:** exit code `0`.

The detailed installation log is saved in:

```text
%TEMP%\wazuh-agent-install.log
```

## 6. Start and check the service

Start the service:

```powershell
Start-Service WazuhSvc
```

Check its status:

```powershell
Get-Service WazuhSvc
```

**Result observed:** `Status = Running`.

Check the installed version:

```powershell
Get-Content ‘C:\Program Files (x86)\ossec-agent\VERSION.json’
```

**Result observed:** version `4.14.9`.

The installation directory used is:

```text
C:\Program Files (x86)\ossec-agent
```

## 7. Check the agent’s connection status

On Windows, check the status file:

```powershell
Select-String `
  -Path ‘C:\Program Files (x86)\ossec-agent\wazuh-agent.state’ `
  -Pattern ‘^status’
```

**Observed result:** `status=“connected”`.

On the Ubuntu server, list the agents:

```bash
sudo /var/ossec/bin/agent_control -l
```

**Result observed:** agent `windows11-arm64`, ID `001`, status `Active`. The output of this command showed `IP: any`, whilst the Wazuh Dashboard displayed the address `192.168.64.5` and the status `Active`.

## 8. Checking Windows event collection

The agent configuration contains a block monitoring the `Security` channel in `eventchannel` format. The section checked in `ossec.conf` was:

```xml
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
  <!-- An event filtering rule follows in the configuration. -->
</localfile>
```

The Windows audit policy for the *Logon* subcategory indicated that successful and failed logon audits were enabled. The audit used, which is independent of the Windows language, was:

```powershell
auditpol /get /subcategory:‘{0CCE9215-69AE-11D9-BED3-505054503030}’
```

To search for failed logon attempts in the Windows event log:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = “Security”
    Id = 4625
} -MaxEvents 5 |
Select-Object TimeCreated, Id, ProviderName, Message |
Format-List
```

**Test carried out:** Locked Windows, deliberately entered an incorrect password, then logged in with the correct password.

**Result observed in Windows:** Event `4625` (*An account failed to log on*) was generated.

## 9. Check detection on the Wazuh server

On Ubuntu, search for alerts associated with Windows event 4625:

```bash
sudo grep -E “‘eventID’[[:space:]]*:[[:space:]]*‘4625’” \
  /var/ossec/logs/alerts/alerts.json | tail -n 5
```

To examine the fields of an alert using `jq`:

```bash
sudo grep -E “‘eventID’[[:space:]]*:[[:space:]]*‘4625’” \
  /var/ossec/logs/alerts/alerts.json | tail -n 1 | jq
```

**Result confirmed in the lab:**

| Field | Value |
|---|---|
| Agent | `windows11-arm64` |
| Windows Event ID | `4625` |
| Wazuh Rule ID | `60122` |
| Description | `Logon failure – unknown user or incorrect password` |
| Rule level | `5` |

This confirms, for this test, the following sequence of events: generation of the event by Windows, collection by the agent, transmission to the Manager and creation of a Wazuh alert.

## 10. Ports and validation results

| Check | Result |
|---|---|
| TCP/1514 from Windows to the server | Accessible (`True`) |
| TCP/1515 from Windows to the server | Accessible (`True`) |
| MSI return code | `0` |
| `WazuhSvc` service | `Running` |
| Local agent status | `connected` |
| Manager-side status | `Active` |
| Dashboard status | `Active` |
| Detection of event 4625 | Confirmed |
| Wazuh rule observed | `60122`, level `5` |

## 11. Troubleshooting and precautions

- If the service has not started, check its status using `Get-Service WazuhSvc` and consult the log file `C:\Program Files (x86)\ossec-agent\ossec.log`.
- If the agent does not appear to be active, check the Manager’s address and the TCP/1514 and TCP/1515 ports, then check the agent log and `/var/ossec/logs/ossec.log` on the server side.
- If no event 4625 is generated, check the Windows audit policy and the `Security` log before modifying the Wazuh collection configuration.
- Wazuh archives (`archives.json`) are not enabled in the current lab configuration (`<logall>no</logall>` and `<logall_json>no</logall_json>`). The absence of this file does not, therefore, in itself indicate a collection failure. Whether to enable archives should be assessed taking into account the available disk space.

## 12. Références

- [Wazuh — Déploiement des agents Wazuh sur les terminaux Windows](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html)
- [Wazuh — Liste des paquets](https://documentation.wazuh.com/current/installation-guide/packages-list.html)
- [Wazuh — Canal d'événements Windows](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/configuration.html)
