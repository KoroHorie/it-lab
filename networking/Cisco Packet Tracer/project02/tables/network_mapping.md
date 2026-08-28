# Network Mapping Matrix

| Local Device | Local Interface | Connected Device | Connected Interface |
| :--- | :--- | :--- | :--- |
| **R1** | S0/0/1 | R3 | S0/0/0 |
| | G0/0 | SW1 | G0/1 |
| | G0/1 | SW2 | G0/1 |
| **R2** | S0/0/0 | R3 | S0/0/0 |
| | G0/0 | SW3 | G0/1 |
| **R3** | S0/0/0 | R2 | S0/0/0 |
| | S0/0/1 | R1 | S0/0/1 |
| | G0/0 | DNS Server | F0 |
| **SW1** | F0/1 | PC1 | F0 |
| | F0/2 | PC0 | F0 |
| | F0/3 | PC4 | F0 |
| | F0/4 | PC5 | F0 |
| | G0/1 | R1 | G0/0 |
| **SW2** | F0/1 | PC2 | F0 |
| | F0/2 | PC3 | F0 |
| | F0/3 | Access Point | Port 0 |
| | G0/1 | R1 | G0/1 |
| **SW3** | G0/1 | R2 | G0/0 |
| | Port Channel 1 | SW5 | Port Channel 3 |
| | F0/2, F0/3 | SW5 | F0/10, F0/11 |
| | Port Channel 2 | SW4 | Port Channel 1 |
| | F0/4, F0/5 | SW4 | F0/8, F0/9 |
| **SW4** | Port Channel 1 | SW5 | Port Channel 2 |
| | F0/8, F0/9 | SW5 | F0/1, F0/2 |
| | Port Channel 2 | SW3 | Port Channel 2 |
| | F0/1, F0/2 | SW3 | F0/4, F0/5 |
| | F0/3 | PC6 | F0 |
| | F0/4 | PC7 | F0 |
| | F0/5 | PC8 | F0 |
| | F0/6 | PC9 | F0 |
| | F0/7 | PC10 | F0 |
| **SW5** | Port Channel 2 | SW3 | Port Channel 1 |
| | F0/1, F0/2 | SW3 | F0/2, F0/3 |
| | Port Channel 3 | SW4 | Port Channel 1 |
| | F0/10, F0/11 | SW4 | F0/8, F0/9 |
| | F0/3 | Srv1 | F0 |
| | F0/4 | Srv2 | F0 |
| **Access Point** | Port 0 | SW2 | F0/3 |
| **PC6** | F0 | SW4 | F0/3|
| **PC7** | F0 | SW4 | F0/4|
| **PC8** | F0 | SW4 | F0/5|
| **PC9** | F0 | SW4 | F0/6|
| **PC10** | F0 | SW4 | F0/7|
| **Srv1** | F0 | SW5 | F0/3|
| **Srv2** | F0 | SW5 | F0/4|
| **DNS Server** | F0 | R3 | G0/0|
