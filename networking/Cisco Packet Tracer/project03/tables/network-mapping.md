# Network Mapping Matrix

| Local Device | Local Interface | Connected Device | Connected Interface |
| :--- | :--- | :--- | :--- |
| **ISP1** | Gig0/0/0 | FW1 | Gig1/1 |
| **ISP2** | Gig0/0/0 | FW2 | Gig1/1 |
| **FW1** | Gig1/1 | ISP1 | Gig1/0/1 |
| | Gig1/3 | CORE1 | Gig1/0/1|
| **FW2** | Gig1/1 | ISP2 | Gig1/0/1 |
| | Gig1/3 | CORE2 | Gig1/0/1 |
| **CORE1** | Gig1/0/1 | FW1 | Gig1/3 |
| | Port-Channel1 | CORE2 | Port-Channel1 |
| | Gig1/0/15 | CORE2 | Gig1/0/15 |
| | Gig1/0/16 | CORE2 | Gig1/0/16 |
| | Gig1/0/22 | DHCP-Srv | Gig0 |
| | Gig1/0/2 | AAA-Srv | Gig0 |
| | Gig1/0/10 | DNS-Srv | Gig0 |
| | Gig1/0/10 | ACCESS1 | Gig0/1 |
| | Gig1/0/11 | ACCESS2 | Gig0/1 |
| | Gig1/0/12 | ACCESS3 | Gig0/2 |
| **CORE2** | Gig1/0/1 | FW2 | Gig1/3 |
| | Port-Channel1 | CORE1 | Port-Channel1 |
| | Gig1/0/15 | CORE1 | Gig1/0/15 |
| | Gig1/0/16 | CORE1 | Gig1/0/16 |
| | Gig1/0/5 | WLC | Gig1 |
| | Gig1/0/10 | ACCESS1 | Gig0/2 |
| | Gig1/0/11 | ACCESS2 | Gig0/2 |
| | Gig1/0/12 | ACCESS3 | Gig0/1 |
| **ACCESS1** | Gig0/1 | CORE1 | Gig1/0/10 |
| | Gig0/2 | CORE2 | Gig1/0/10 |
| | Fa0/1 | PC0 | Fa0 |
| | Fa0/5 | PC1 | Fa0 |
| **ACCESS2** | Gig0/1 | CORE1 | Gig1/0/11 |
| | Gig0/2 | CORE2 | Gig1/0/11 |
| | Fa0/1 | PC2 | Fa0 |
| | Fa0/5 | PC3 | Fa0 |
| **ACCESS3** | Gig0/1 | CORE2 | Gig1/0/12 |
| | Gig0/2 | CORE1 | Gig1/0/12 |
| | Fa0/1 | LWAP1 | Gig0 |
| | Fa0/2 | LWAP2 | Gig0 |
| | Fa0/3 | LWAP3 | Gig0 |
| **WLC** | Gig1 | CORE2 | Gig1/0/5 |
| **DHCP-Srv** | Gig0 | CORE1 | Gig1/0/22 |
| **AAA-Srv** | Gig0 | CORE1 | Gig1/0/2 |
| **DNS-Srv** | Gig0 | CORE1 | Gig1/0/23 |
| **PC0** | Fa0 | ACCESS1 | Fa0/1 |
| **PC1** | Fa0 | ACCESS1 | Fa0/5 |
| **PC2** | Fa0 | ACCESS2 | Fa0/1 |
| **PC3** | Fa0 | ACCESS2 | Fa0/5 |
| **LWAP1** | Gig0 | ACCESS3 | Fa0/1 |
| **LWAP2** | Gig0 | ACCESS3 | Fa0/2 |
| **LWAP3** | Gig0 | ACCESS3 | Fa0/3 |
| **Tablet PC1** | Wireless0 | LWAP | WLAN |
| **Smartphone0** | Wireless0 | LWAP | WLAN |
| **Laptop0** | Wireless0 | LWAP | WLAN |
