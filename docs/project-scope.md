# Project Scope — SOC Lab Windows/Linux

## Objective
To set up an educational SOC lab using Wazuh to practise event collection, detection, alert analysis and report writing, as part of a portfolio for a junior SOC Analyst role.

## Scope
- Central Wazuh server running on Ubuntu Server 24.04 LTS ARM64.
- Windows endpoint: Windows 11 Pro.
- Linux endpoint: Ubuntu.
- Wazuh agents on both endpoints.
- Scenarios: authentication, file integrity, Windows events and security assessment.

## Out of scope
- Monitoring of the host Mac: it runs UTM and provides access to the Dashboard, but is not an endpoint for this project.
- Production deployment or public exposure of the Dashboard.
- Testing on third-party systems without authorisation.
- Real malware, destructive testing and collection of personal data.
- Guarantee of comprehensive detection: results depend on logs, rules and configurations.

## Success criteria
1. The four core Wazuh services are operational.
2. The Windows and Linux agents are connected.
3. Events from both endpoints are visible in Wazuh.
4. Each scenario includes a procedure, observations, evidence and conclusion.

## Publication
