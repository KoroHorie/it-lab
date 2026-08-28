# Network Mapping

| Local Device | Local Interface | Connected Device | Connected Interface |
| :--- | :--- | :--- | :--- |
| **R1** | G0/0 | S1A | G0/1 |
| | G0/1 | S1B | G0/1 |
| | S0/0/0 | R3 | S0/0/0 |
| **R2** | G0/1 | S2A | G0/1 |
| | G0/0 | R3 | G0/0 |
| **R3** | G0/0 | R2 | G0/0 |
| | S0/0/0 | R1 | S0/0/0 |
| | G0/1 | S3B | G0/1 |
| **S1A** | F0/1 | PC3 | F0 |
| | G0/1 | R1 | G0/1 |
| **S1B** | F0/2 | PC4 | F0 
| | G0/1 | R1 | G0/1 |
| **S2A** | F0/1 | PC2 | F0 |
| | G0/1 | R2 | G0/1 |
| **S3A** | F0/1 | PC0 | F0 |
| | F0/2 | S3B | F0/2 |
| **S3B** | G0/1 | R3 | G0/1 |
| | F0/2 | S3A | F0/2 |
| | F0/3 | S3C | F0/3 |
| **S3C** | F0/2 | PC1 | F0/2 |
| | F0/3 | S3B | F0/3 |
| **PC0** | F0 | S3A | F0/1 |
| **PC1** | F0 | S3C | F0/2 |
| **PC2** | F0 | S2A | F0/1 |
| **PC3** | F0 | S1A | F0/1 |
| **PC4** | F0 | S1B | F0/2 |
