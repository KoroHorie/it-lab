# Network Mapping

| Local Device | Local Interface | Connected Device | Connected Interface |
| :--- | :--- | :--- | :--- |
| ISP1 | Gig0/0 | FW1 | Gig0/0 |
|  | Gig0/1 | FW2 | Gig0/1 |
|  | Gig0/2 | Cloud1 | eth0 |
| ISP2 | Gig0/0 | FW2 | Gig0/0 |
|  | Gig0/1 | FW1 | Gig0/1 |
|  | Gig0/2 | Cloud2 | eth0 |
| FW1 | Gig0/0 | ISP1 | Gig0/0 |
|  | Gig0/1 | ISP2 | Gig0/1 |
|  | Gig0/2 | CORE1 | Gig0/0 |
|  | Gig0/3 | CORE2 | Gig2/0 |
| FW2 | Gig0/0 | ISP2 | Gig0/0 |
|  | Gig0/1 | ISP1 | Gig0/1 |
|  | Gig0/2 | CORE2 | Gig0/0 |
|  | Gig0/3 | CORE1 | Gig2/0 |
| CORE1 | Gig0/0 | FW1 | Gig0/2 |
|  | Gig0/1 | ACCESS1 | Gig0/0 |
|  | Gig0/2 | ACCESS2 | Gig0/0 |
|  | Gig0/3 | ACCESS3 | Gig0/0 |
|  | Port-channel1 | CORE2 | Port-channel1 |
|  | Gig1/0 | CORE2 | Gig1/0 |
|  | Gig1/1 | CORE2 | Gig1/1 |
|  | Gig1/2 | DHCP-Srv | eth0 |
|  | Gig1/3 | NetAutomation | eth0 |
|  | Gig2/0 | FW2 | Gig0/3 |
| CORE2 | Gig0/0 | FW2 | Gig0/2 |
|  | Gig0/1 | ACCESS1 | Gig0/1 |
|  | Gig0/2 | ACCESS2 | Gig0/1 |
|  | Gig0/3 | ACCESS3 | Gig0/1 |
|  | Port-channel1 | CORE1 | Port-channel1 |
|  | Gig1/0 | CORE1 | Gig1/0 |
|  | Gig1/1 | CORE1 | Gig1/1 |
|  | Gig2/0 | FW1 | Gig0/3 |
| ACCESS1 | Gig0/0 | CORE1 | Gig0/1 |
|  | Gig0/1 | CORE2 | Gig0/1 |
|  | Gig0/3 | webterm1 | eth0 |
| ACCESS2 | Gig0/0 | CORE1 | Gig0/2 |
|  | Gig0/1 | CORE2 | Gig0/2 |
|  | Gig0/3 | webterm3 | eth0 |
| ACCESS3 | Gig0/0 | CORE1 | Gig0/3 |
|  | Gig0/1 | CORE2 | Gig0/3 |
|  | Gig0/3 | webterm2 | eth0 |
| DHCP-Srv | eth0 | CORE1 | Gig1/2 |
| NetAutomation | eth0 | CORE1 | Gig1/3 |
| webterm1 | eth0 | ACCESS1 | Gig0/3 |
| webterm2 | eth0 | ACCESS2 | Gig0/3 |
| webterm3 | eth0 | ACCESS3 | Gig0/3 |
