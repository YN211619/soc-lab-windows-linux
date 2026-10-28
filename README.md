# SOC Lab — Windows & Linux Security Monitoring

## 1. Project overview

This project consists of a small, isolated cybersecurity lab designed to practise defensive security fundamentals and SOC analyst workflows.

The lab focuses on collecting and analysing security events from Linux and Windows systems using Wazuh. It aims to demonstrate practical knowledge of networking, operating system administration, security monitoring, event analysis, and incident documentation.

**Project status:** Initial planning

## 2. Objectives

- Build a small virtual network with documented connectivity and security boundaries.
- Deploy Wazuh and connect monitored endpoints.
- Understand and analyse Windows and Linux security logs.
- Implement and validate basic security detection scenarios.
- Investigate alerts and document findings, limitations, and remediation measures.
- Produce reproducible technical documentation suitable for a cybersecurity portfolio.

## 3. Proposed architecture

| Component | Role |
|---|---|
| Ubuntu Server ARM64 | Wazuh central components |
| Debian ARM64 | Linux endpoint |
| Windows 11 ARM | Windows endpoint, subject to agent compatibility validation |
| VMware Fusion Pro | Local virtualisation platform |
| Git and GitHub | Version control and public documentation |

The lab will run on an Apple Silicon MacBook Air M4 with 16 GB of unified memory.

## 4. Planned security scenarios

1. Authentication monitoring and analysis of failed login attempts.
2. File integrity monitoring for selected sensitive files.
3. Windows security event collection and investigation.
4. Security configuration assessment and vulnerability review.

Each scenario will include its objective, prerequisites, procedure, observed results, evidence, analysis, and remediation recommendations.

## 5. Repository structure

```text
soc-lab-windows-linux/
├── README.md
├── docs/
│   ├── project-scope.md
│   ├── architecture.md
│   ├── network-design.md
│   └── limitations.md
├── diagrams/
├── setup/
│   ├── wazuh/
│   ├── linux/
│   └── windows/
├── scenarios/
│   ├── 01-authentication-monitoring/
│   ├── 02-file-integrity/
│   ├── 03-windows-events/
│   └── 04-security-assessment/
└── reports/
    ├── incident-report-template.md
    └── final-report.md
```

The repository will be populated progressively as each component is installed and tested.

## 6. Security and scope

All security tests will be performed on systems owned by the project author or explicitly authorised for testing.

The laboratory will remain isolated from production systems. No real credentials, private keys, access tokens, or sensitive personal information will be published.

The project is intended for educational and defensive security purposes. It is not a production-ready SOC deployment.

## 7. Documentation and reproducibility

Configuration choices, software versions, test procedures, and observed results will be documented.

Screenshots and logs will be used as supporting evidence, with sensitive information removed. Expected results will be distinguished from results actually observed in the lab.

## 8. References

- [Wazuh documentation](https://documentation.wazuh.com/current/index.html)
- [Wazuh quickstart](https://documentation.wazuh.com/current/quickstart.html)
- [VMware Fusion — official download and licensing information](https://knowledge.broadcom.com/external/article/368667/download-and-license-free-vmware-desktop.html)

## 9. Current limitations and open questions

- Wazuh resource usage must be validated on the available hardware.
- Windows 11 ARM compatibility with the Wazuh agent must be tested.
- The final network configuration and detection rules will be documented after implementation.

This README will be updated throughout the project to reflect the actual implementation and verified results.
