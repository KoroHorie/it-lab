# Join a Windows Machine to a Domain Using PowerShell

This guide explains how to configure DNS and join a Windows computer to an Active Directory domain using PowerShell.

---

## Prerequisites

Before joining the domain, make sure:

- The computer can reach the Domain Controller
- You have domain credentials with permission to join devices
- The correct DNS server (Domain Controller IP) is configured
- PowerShell is running **as Administrator**

---

## Step 1: Identify Network Interface Index

Run the following command to determine the interface index of your active network adapter:

```powershell
Get-NetIPConfiguration
```

Look for the interface connected to your network and note the **InterfaceIndex** value (example: `6` or `12`).

---

## Step 2: Configure DNS Server Address

Replace `INDEX` with your actual interface index and provide the IP address of your Domain Controller (DNS Server):

```powershell
Set-DnsClientServerAddress -InterfaceIndex INDEX -ServerAddresses ("x.x.x.x","y.y.y.y")
```

Example:

```powershell
Set-DnsClientServerAddress -InterfaceIndex 6 -ServerAddresses ("192.168.1.10")
```

Verify DNS configuration:

```powershell
Get-DnsClientServerAddress
```

---

## Step 3: Join the Computer to the Domain

Run the following command:

```powershell
Add-Computer -DomainName "YourDomainName" -Credential (Get-Credential) -Restart
```

Example:

```powershell
Add-Computer -DomainName "corp.local" -Credential (Get-Credential) -Restart
```

You will be prompted to enter domain administrator credentials.

After successful authentication, the machine will automatically restart and join the domain.

---

## Step 4: Verify Domain Membership

After restart, confirm the machine successfully joined the domain:

```powershell
systeminfo | Select-String Domain
```

Expected output:

```
Domain: corp.local
```

---

## Step 5: Unjoin the Computer from the Domain (Optional)

If you need to remove the computer from the domain and return it to a workgroup environment, run the following command:

```powershell
Add-Computer -WorkGroupName "WORKGROUP" -Credential (Get-Credential) -Restart
```

You will be prompted for domain credentials that have permission to remove the computer from the domain.

After successful authentication, the machine will restart and return to the default **WORKGROUP** configuration.

---

## Troubleshooting Tips

If the domain join fails:

- Ensure DNS is pointing to the Domain Controller
- Confirm the client can ping the Domain Controller
- Verify time synchronization between client and server
- Make sure firewall rules allow domain communication

Check connectivity:

```powershell
Test-Connection YourDomainControllerIP
```

Check domain resolution:

```powershell
nslookup YourDomainName
```

---

## Summary

Steps performed:

1. Identified network interface index
2. Configured DNS server
3. Joined computer to domain
4. Verified domain membership
5. (Optional) Removed computer from domain and returned to WORKGROUP

You have now successfully joined or removed a Windows device
