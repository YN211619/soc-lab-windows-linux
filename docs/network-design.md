# Network Design — SOC Lab Windows/Linux

## Known configuration
The VMs are hosted in UTM. The network mode used during installation is Shared Network (to be confirmed in UTM).

Observed server address:
- `wazuh-server`: `192.168.64.6`

The addresses of the Windows and Ubuntu endpoints are not yet known and have not been assigned here.

## Logical diagram
```text

                                                         ____  Ubuntu Server
                                                        |        Wazuh Server 
                                                        |        192.168.64.6 
                                                        |       
MacBook Air M4 (Host) -- UTM -- Shared virtual network -|----  Windows 11
                                                        |        Endpoint 
                                                        |        DHCP to be obtained
                                                        |
                                                         ----  Ubuntu Linux
                                                                 Endpoint
                                                                 DHCP to be obtained

```

## Required data flows
| Source | Destination | Port | Purpose |
|---|---|---:|---|
| Windows Agent | Manager | 1514 | Events |
| Ubuntu Agent | Manager | 1514 | Events |
| Endpoints | Manager | 1515/TCP | Initial registration if used |
| Mac / browser | Dashboard | 443/TCP | HTTPS interface |
| Manager/Filebeat | Indexer | 9200/TCP | Alert transmission |

The protocols and ports must be confirmed in the actual configuration. 

## Checks
On each Linux VM:

```bash
ip -4 -br addr
ping -c 4 192.168.64.6
```

A successful ping does not prove that the application ports are open. If `nc` is installed:

```bash
nc -vz 192.168.64.6 1514
nc -vz 192.168.64.6 1515
```

The IP address observed may change if DHCP is used. Check it before installing the agents; consider a DHCP reservation or a static address if the UTM allows it.

## Security
Restrict the Dashboard to the laboratory network, do not expose the Indexer.
