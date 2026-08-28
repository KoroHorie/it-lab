# Configure Ubuntu as a Router (Complete Guide)

This guide explains how to configure an Ubuntu machine as a router using **Netplan**, **IP forwarding**, **iptables NAT**, and optionally an **ISC DHCP Server**.

It assumes:
- `ens18` = WAN interface (connected to upstream network / internet)
- `ens19` = LAN interface (internal network clients)

---

# 1. Configure Network Interfaces (Netplan)

## Command

```bash
sudo cat /etc/netplan/50-cloud-init.yaml
```

## Example Configuration

```yaml
network:
  version: 2
  ethernets:
    ens18:
      addresses:
      - "172.16.2.5/24"
      nameservers:
        addresses:
        - 8.8.8.8
      routes:
      - to: "default"
        via: "172.16.2.1"

    ens19:
      addresses:
      - "192.168.1.1/24"
```

## Explanation

This file configures how Ubuntu assigns IP addresses to network interfaces.

### ens18 (WAN Interface)

- `172.16.2.5/24`
  - Static IP assigned to router’s WAN side
  - `/24` means subnet mask `255.255.255.0`

- `nameservers: 8.8.8.8`
  - DNS server used to resolve domain names

- `default route via 172.16.2.1`
  - Sets upstream gateway
  - Sends unknown traffic to internet router

### ens19 (LAN Interface)

- `192.168.1.1/24`
  - Router becomes gateway for LAN clients

Clients will later use this address as their **default gateway**.

---

# 2. Enable IP Forwarding

Ubuntu must be told to forward packets between interfaces.

Without this step, routing will NOT work.

---

## Method 1 — Permanent Configuration

### Command

```bash
sudo nano /etc/sysctl.conf
```

Find:

```bash
net.ipv4.ip_forward=1
```

Then apply changes:

```bash
sudo sysctl -p
```

### Explanation

This enables packet forwarding inside the Linux kernel.

Effect:

```
LAN → Ubuntu Router → Internet
```

becomes possible.

The command:

```bash
sudo sysctl -p
```

reloads kernel parameters from `/etc/sysctl.conf`.

---

## Method 2 — Temporary Runtime Configuration

### Verify Status

```bash
cat /proc/sys/net/ipv4/ip_forward
```

### Output Meaning

```
0 = disabled
1 = enabled
```

### Enable Temporarily

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

### Explanation

This enables forwarding immediately but resets after reboot.

Useful for testing before making permanent changes.

---

# 3. Configure NAT Using iptables

Now we allow internal devices to access external networks.

This step performs **Network Address Translation (NAT)**.

---

## Enable NAT Masquerading

### Command

```bash
sudo iptables -t nat -A POSTROUTING -o ens18 -j MASQUERADE
```

### Explanation

This rule:

- modifies outgoing packets
- replaces internal private IP
- with router’s public/WAN IP

Example:

```
192.168.1.10 → 8.8.8.8
```

becomes:

```
172.16.2.5 → 8.8.8.8
```

So return traffic knows where to go.

This is exactly how home routers work.

---

## Allow Forwarding from LAN to WAN

### Command

```bash
sudo iptables -A FORWARD -i ens19 -o ens18 -j ACCEPT
```

### Explanation

Allows traffic flow:

```
LAN → Internet
```

Without this rule, packets would be blocked.

---

## Allow Return Traffic

### Command

```bash
sudo iptables -A FORWARD -i ens18 -o ens19 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

### Explanation

Allows only response traffic back from internet.

Example:

```
Client requests webpage
Internet replies
Reply allowed back in
```

Prevents unauthorized inbound connections.

Adds firewall-like protection.

---

# 4. Verify NAT Rules

### Command

```bash
sudo iptables -t nat -L POSTROUTING -v -n
```

### Explanation

Displays NAT table rules.

Flags:

| Option | Meaning |
|-------|---------|
| -t nat | Show NAT table |
| -L | List rules |
| -v | Verbose output |
| -n | Numeric output (no DNS lookup) |

Expected result:

```
MASQUERADE  all  --  0.0.0.0/0  0.0.0.0/0
```

Confirms NAT is active.

---

# 5. Make iptables Rules Persistent

iptables resets after reboot by default.

Install persistence package:

```bash
sudo apt install iptables-persistent
```

Save rules:

```bash
sudo netfilter-persistent save
```

### Explanation

Stores rules so they reload automatically after reboot.

Without this step:

Router stops working after restart.

---

# 6. Optional: Configure Ubuntu as DHCP Server

This allows router to automatically assign IP addresses to LAN clients.

---

## Install DHCP Server

```bash
sudo apt update
sudo apt install isc-dhcp-server -y
```

### Explanation

Installs DHCP service responsible for:

- assigning IP addresses
- defining gateway
- setting DNS servers

for connected LAN devices.

---

## Specify Interface

Edit configuration:

```bash
sudo nano /etc/default/isc-dhcp-server
```

Set:

```bash
INTERFACESv4="ens19"
```

### Explanation

Tells DHCP server:

```
Serve LAN clients only
```

Prevents conflicts with WAN network.

---

## Configure DHCP Address Pool

Edit configuration:

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

Example:

```bash
subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.50 192.168.1.150;
  option routers 192.168.1.1;
  option subnet-mask 255.255.255.0;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  default-lease-time 600;
  max-lease-time 7200;
}
```

### Explanation

Defines automatic IP assignment policy.

| Setting | Meaning |
|--------|---------|
| subnet | Local network |
| range | Assignable IP addresses |
| routers | Default gateway |
| subnet-mask | Network mask |
| domain-name-servers | DNS servers |
| default-lease-time | Default lease duration |
| max-lease-time | Maximum lease duration |

Clients connecting to network automatically receive:

- IP address
- gateway
- DNS

---

## Restart DHCP Service

```bash
sudo systemctl restart isc-dhcp-server
```

### Explanation

Applies configuration changes immediately.

---

## Check Service Status

```bash
sudo systemctl status isc-dhcp-server
```

### Explanation

Shows:

- whether service is running
- errors if configuration failed
- logs for troubleshooting

Look for:

```
active (running)
```

which confirms DHCP server is working correctly.

---

# Final Result

After completing all steps:

Your Ubuntu machine becomes:

- Router
- NAT Gateway
- Firewall (basic state filtering)
- Optional DHCP Server

Equivalent to a small home/enterprise edge router built entirely in Linux.

