# Scenario 01 – Authentication Monitoring

**Date completed:** 11 November 2026
**Status:** Validated
**Environment:** Wazuh 4.14.9 – Windows 11 Pro – Ubuntu 24.04 LTS

## 1. Objective

To set up and verify the collection and detection of authentication failures on Windows and Linux endpoints monitored by Wazuh.

The tests cover:
- Windows login failures.
- Local Linux authentication failures.
- SSH authentication failures.
- SSH attempts using a non-existent user.

All events were deliberately generated in the laboratory.

## 2. Architecture used

| Component | Name | IP Address |
|---|---|---|
| Wazuh Server | wazuh-server | 192.168.64.6 |
| Windows Endpoint | windows11-arm64 | 192.168.64.5 |
| Linux Endpoint | ubuntu-linux-arm64 | 192.168.64.4 |

Wazuh agents:
- Windows: ID `001`
- Linux: ID `002`

## 3. Tests carried out and results

### Test 1 – Failed Windows login

**Action:** Deliberately entering an incorrect password on the Windows login screen, followed by a successful login.

**Evidence detected:**

| Field | Value |
|---|---|
| Windows Event | `4625` |
| Agent | `windows11-arm64` |
| Rule ID | `60122` |
| Wazuh Level | 5 |
| Description | Logon Failure – unknown user or bad password |

**DQL query:**

```text
agent.name: ‘windows11-arm64’
AND rule.id: 60122
```

**Interpretation:** Wazuh detected a failed Windows logon and associated the event with a detection rule.

### Test 2 – Failed local Linux login

**Action:** Deliberate attempt to log in incorrectly via the graphical interface on Ubuntu.

**Evidence detected:**

| Field | Value |
|---|---|
| Agent | `ubuntu-linux-arm64` |
| Rule ID | `2501` |
| Wazuh Level | 5 |
| Groups | `pam`, `syslog`, `authentication_failed` |
| Programme | `gdm-password` |

**Log extract:**

```text
pam_unix(gdm-password:auth): authentication failure
```

**DQL query:**

```text
agent.name: ‘ubuntu-linux-arm64’
AND rule.id: 2501
```

**Interpretation:** The event indicates a PAM authentication failure associated with the GDM graphical login manager. The `rhost` field was empty: the observed elements do not indicate a remote source.

### Test 3 – SSH failure with a non-existent user

**Action:** SSH attempt from the Wazuh server to the Linux endpoint using a misspelt username.

**Evidence detected:**

| Field | Value |
|---|---|
| Source | `192.168.64.6` |
| Destination | `192.168.64.4` |
| Username entered | `ubunutusoc` |
| Rule ID | `5710` |
| Wazuh Level | 5 |
| Description | sshd: Attempt to login using a non-existent user |

**Log extract:**

```text
Invalid user ubunutusoc from 192.168.64.6
```

**DQL query:**

```text
agent.name: ‘ubuntu-linux-arm64’
AND rule.id: 5710
AND data.srcip: ‘192.168.64.6’
```

**Interpretation:** SSH received a login attempt using a non-existent account. This was a deliberate typo during testing, and not evidence of malicious activity.

### Test 4 – Failed SSH connection with an existing user

**Action:** SSH attempt from the Wazuh server (`192.168.64.6`) to Ubuntu (`192.168.64.4`) using the valid account `ubuntusoc`, but with an incorrect password.

**Evidence detected:**

| Field | PAM alert | SSH alert |
|---|---|---|
| Rule ID | `5503` | `5760` |
| Description | PAM: User login failed | sshd: authentication failed |
| Level | 5 | 5 |
| User | `ubuntusoc` | `ubuntusoc` |
| SSH PID | `8462` | `8462` |
| Source IP | `192.168.64.6` | `192.168.64.6` |

**Log extracts:**

```text
pam_unix(sshd:auth): authentication failure
```

```text
Failed password for ubuntusoc from 192.168.64.6
```

**DQL query:**

```text
agent.name: ‘ubuntu-linux-arm64’
AND data.srcip: ‘192.168.64.6’
AND (rule.id: 5503 OR rule.id: 5760)
```

**Interpretation:**
- Rule `5503` indicates an authentication failure reported by PAM.
- Rule `5760` indicates a password failure detected by SSH.
- The identical PID (`8462`) and closely spaced timestamps indicate that the two alerts correspond to the same attempt.

These alerts should therefore not be counted as two separate attempts.

## 4. Summary of observed rules

| Rule ID | Description | Source |
|---|---|---|
| `60122` | Windows login failure | Windows Security |
| `2501` | PAM/GDM authentication failure | Linux |
| `5710` | Attempt with non-existent user | SSH |
| `5503` | PAM authentication failure | Linux / SSH |
| `5760` | SSH authentication failure | SSH |

All alerts observed during these tests had a Wazuh severity level of 5.

## 5. Technical skills required

- Viewing alerts in the Wazuh Dashboard.
- Using Discover.
- Filtering events using DQL.
- Analysing the `agent.name`, `rule.id`, `data.srcip` and `data.dstuser` fields.
- Reading Windows Security logs.
- Analysing Linux PAM and SSH events.
- Correlating multiple events based on PID, IP address and timestamp.
- Distinguishing between a non-existent user and an incorrect password.

## 6. Limitations and precautions

- The events originate from controlled tests: they do not indicate an actual attack.
- A single authentication attempt may generate multiple alerts. Events must be correlated before counting the attempts.
- The local SSH test carried out from Ubuntu to its own IP address (`192.168.64.4`) does not constitute a test from a remote machine.
- There is a one-hour discrepancy between the timestamps in the Linux logs and those displayed in Discover. The Wazuh server and the Linux endpoint do not use the same time zone. This difference must be taken into account during an investigation.
- Wazuh logging is not enabled in this installation (`logall=no` and `logall_json=no`). The analysis presented here is based on the alerts indexed in `wazuh-alerts-*`.

## 7. Conclusion

Scenario 01 has been validated.

Windows and Linux authentication failure events were collected by the Wazuh agents, forwarded to the manager and viewed in the Dashboard.

The tests also made it possible to distinguish between:
- A non-existent account.
- An existing account associated with an incorrect password.
- A local authentication failure.
- A remote SSH authentication failure.

**Result:** the chain for collecting and detecting authentication failures is functioning on both endpoints in the laboratory.

## 8. Possible improvements

- Create a dashboard dedicated to authentication failures.
- Investigate the detection of multiple successive failures.
- Add a frequency analysis by source IP address.
- Set up a documented investigation based on a sequence of alerts.

