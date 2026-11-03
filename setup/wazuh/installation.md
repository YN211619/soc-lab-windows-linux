# Wazuh installation — official multi-step wizard

## Environment
- UTM on a MacBook Air M4.
- Ubuntu Server 24.04 LTS ARM64.
- Hostname: `wazuh-server`; user: `socadmin`.
- Observed IP: `192.168.64.6`.
- Wazuh `4.14.9`, centrally installed on a VM.

## Preparation

```bash
sudo apt update
sudo apt upgrade -y
mkdir -p ~/wazuh-install
cd ~/wazuh-install
curl -fLO https://packages.wazuh.com/4.14/wazuh-install.sh
curl -fLO https://packages.wazuh.com/4.14/config.yml
```

Check the architecture and IP address:

```bash
uname -m
ip -4 -br addr
```

## Configuring the nodes

In `config.yml`, the three roles have been defined on the same IP address:

```yaml
nodes:
  indexer:
    - name: node-1
      ip: ‘192.168.64.6’
  server:
    - name: wazuh-1
      ip: ‘192.168.64.6’
  dashboard:
    - name: dashboard
      ip: ‘192.168.64.6’
```

Retain the other settings from the official file and ensure correct YAML indentation.

## Generating certificates and secrets

```bash
sudo bash wazuh-install.sh --generate-config-files
sudo chmod 600 wazuh-install-files.tar
```

## Installing the Indexer

```bash
sudo bash wazuh-install.sh --wazuh-indexer node-1
sudo bash wazuh-install.sh --start-cluster
sudo systemctl status wazuh-indexer --no-pager
```

Expected status: `active (running)`.

## Installing the Wazuh Server

```bash
sudo bash wazuh-install.sh --wazuh-server wazuh-1
sudo systemctl is-active wazuh-manager filebeat
```

The Server role installs Manager and Filebeat.

## Installing the Dashboard

```bash
sudo bash wazuh-install.sh --wazuh-dashboard dashboard
sudo systemctl status wazuh-dashboard --no-pager
```

## Final check

```bash
sudo systemctl is-active wazuh-indexer wazuh-manager filebeat wazuh-dashboard
```

All four services should return `active`. Access from the host:

```text
https://192.168.64.6
```

A self-signed certificate warning may be displayed. Check the address before continuing.

To view the initial password generated locally:

```bash
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

## Official sources
- Indexer: https://documentation.wazuh.com/current/installation-guide/wazuh-indexer/installation-assistant.html
- Server: https://documentation.wazuh.com/current/installation-guide/wazuh-server/installation-assistant.html
- Dashboard: https://documentation.wazuh.com/current/installation-guide/wazuh-dashboard/installation-assistant.html
