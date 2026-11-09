# Installing the Wazuh agent — Ubuntu Linux ARM64

**Validation date:** 9 November 2026
**Agent version:** `4.14.9-1`
**Agent name:** `ubuntu-linux-arm64`
**Wazuh ID:** `002`

## 1. Objective

To install and enrol the Wazuh agent on the Ubuntu Linux ARM64 VM in order to transmit data from the endpoint to the laboratory’s Wazuh server.

## 2. Verified environment

| Item | Value |
|---|---|
| Operating system | Ubuntu 24.04.4 LTS |
| Debian architecture | `arm64` |
| Kernel architecture | `aarch64` |
| Endpoint IPv4 address | `192.168.64.4` |
| Agent name | `ubuntu-linux-arm64` |
| Installed package version | `4.14.9-1` |
| Installed package architecture | `arm64` |
| ID assigned by Wazuh Manager | `002` |
| Agent status on the Manager side | `Active` |
| Wazuh server address | `192.168.64.6` |
| Wazuh Manager version | `4.14.9` |

The endpoint VM and the server communicate over the UTM virtual network. 

## 3. Check the system and architecture

On the Ubuntu endpoint:

```bash
cat /etc/os-release
uname -m
dpkg --print-architecture
ip -br -4 addr
```

Values observed: Ubuntu 24.04.4 LTS, `aarch64`, Debian architecture `arm64`, IPv4 address `192.168.64.4`.

## 5. Add the official Wazuh repository

Install the prerequisites:

```bash
sudo apt-get update
sudo apt-get install -y curl gnupg apt-transport-https
```

Import the Wazuh signing key:

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | \
  sudo gpg --no-default-keyring \
  --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import

sudo chmod 644 /usr/share/keyrings/wazuh.gpg
```

Add the APT repository:

```bash
echo ‘deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main’ | \
  sudo tee /etc/apt/sources.list.d/wazuh.list

sudo apt-get update
```

Check the availability of the required version:

```bash
apt-cache madison wazuh-agent
```

**Result observed:** version `4.14.9-1` was available for installation.

## 6. Install and enrol the agent

As the version of the lab manager was `4.14.9`, the agent was installed in the same version.

Command executed:

```bash
sudo env \
  WAZUH_MANAGER="192.168.64.6" \
  WAZUH_REGISTRATION_SERVER=‘192.168.64.6’ \
  WAZUH_AGENT_NAME="ubuntu-linux-arm64" \
  apt-get install -y wazuh-agent=4.14.9-1
```

The variables indicate:

- `WAZUH_MANAGER`: the address of the Manager to which the agent sends its data.
- `WAZUH_REGISTRATION_SERVER`: the address of the enrolment service.
- `WAZUH_AGENT_NAME`: the name assigned to the agent in Wazuh.

## 7. Enable and start the service

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

Check the service status:

```bash
sudo systemctl status wazuh-agent --no-pager
```

**Result shown:** service active (`active`).

Check the version and architecture actually installed:

```bash
dpkg-query -W -f=“Version: ${Version}\nArchitecture: ${Architecture}\n” wazuh-agent
```

**Result observed:**

```text
Version: 4.14.9-1
Architecture: arm64
```

## 8. Check the registration on the Manager side

On the Wazuh server (`192.168.64.6`):

```bash
sudo /var/ossec/bin/agent_control -l
```

**Reported result:**

```text
ID: 002, Name: ubuntu-linux-arm64, IP: any, Active
```

The Manager has therefore assigned ID `002` to `ubuntu-linux-arm64` and displays the status `Active`. The output of `agent_control -l` shows `IP: any`; the endpoint VM address recorded separately on Ubuntu is `192.168.64.4`.

## 9. Further checks

On the Linux endpoint, check the agent log if there are any issues:

```bash
sudo tail -n 50 /var/ossec/logs/ossec.log
```

Check the service status:

```bash
systemctl is-active wazuh-agent
```

To check the network connection to the Manager:

```bash
sudo ss -tnp | grep “:1514”
```

These commands are diagnostic checks; their results were not recorded in the initial validation, with the exception of the `active` status of the service and the agent.

## 10. Validation summary

| Check | Result |
|---|---|
| Ubuntu | `24.04.4 LTS` |
| Architecture | `arm64` / `aarch64` |
| Endpoint IPv4 address | `192.168.64.4` |
| Wazuh server | `192.168.64.6` |
| TCP/1514 | Accessible |
| TCP/1515 | Accessible |
| Agent version | `4.14.9-1` |
| Package architecture | `arm64` |
| `wazuh-agent` service | Active (`active`) |
| Wazuh ID | `002` |
| Agent name | `ubuntu-linux-arm64` |
| Manager-side status | `Active` |

**Conclusion:** the installation and enrolment of the Linux agent have been validated. Both endpoints planned for the lab, Windows and Linux, are now registered with the Manager. Analysing alerts in the Dashboard and learning DQL form a subsequent phase of the project.

## 11. Maintaining version consistency

The Manager’s version must be equal to or higher than that of the agents to remain within the compatibility matrix documented by Wazuh. Avoid an unintended upgrade of the agent beyond the Manager’s version.

To place the APT package on hold:

```bash
sudo apt-mark hold wazuh-agent
```

Check whether the package is on hold:

```bash
apt-mark showhold | grep -x wazuh-agent
```

Only consider the package to be on hold if the verification command returns `wazuh-agent`.

## 12. Security and precautions

- Never publish an agent key, enrolment password, secret or configuration file containing credentials.
- Ensure Agent and Manager versions are compatible and check updates before applying them.
- Do not enable logging of all Manager archives without first estimating the required disk space.

## 13. References

- [Wazuh — Deploying Wazuh agents on Linux endpoints](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html)
- [Wazuh — Deployment variables for Linux](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/deployment-variables/deployment-variables-linux.html)
- [Wazuh — List of packages](https://documentation.wazuh.com/current/installation-guide/packages-list.html)
- [Wazuh — Wazuh agent installation guide](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html)

