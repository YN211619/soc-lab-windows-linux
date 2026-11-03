# Architecture — SOC Lab Windows/Linux

## Overview
The MacBook Air M4 is the physical host. UTM hosts three VMs: a Wazuh Ubuntu Server, a Windows 11 Pro endpoint and an Ubuntu Linux endpoint. The Mac is not an endpoint within the initial scope.

## Central server
The four components are installed on a single VM:
- **Wazuh Manager**: receives events from the agents, applies rules and generates alerts.
- **Wazuh Indexer**: stores and indexes searchable data.
- **Filebeat**: forwards alerts from the Manager to the Indexer.
- **Wazuh Dashboard**: web interface for viewing and administration.

## Endpoints
- **Windows 11 Pro**: Wazuh agent to be installed; targets Windows event and authentication scenarios.
- **Ubuntu Linux**: Wazuh agent to be installed; targets Linux authentication and file integrity scenarios.

## Workflow
1. The agents transmit events to the Manager.
2. The Manager analyses the events using decoders and rules.
3. Filebeat forwards alerts to the Indexer.
4. The Dashboard queries the Indexer to display events and alerts.

## Common ports
| Port | Purpose |
|---:|---|
| 1514 | Agent/Manager communication (protocol depends on configuration) |
| 1515/TCP | Agent registration |
| 55000/TCP | Wazuh API |
| 443/TCP | Dashboard HTTPS |
| 9200/TCP | Indexer API, must not be exposed to endpoints |
| 9300/TCP | Internal communication within the Indexer cluster |

Check the actual configuration before creating network rules.

## Progress
- [x] Ubuntu server installed.
- [x] Indexer installed.
- [x] Manager and Filebeat installed.
- [x] Dashboard installed and accessible.
- [ ] Windows agent connected.
- [ ] Ubuntu agent connected.
- [ ] Scenarios implemented and documented.
