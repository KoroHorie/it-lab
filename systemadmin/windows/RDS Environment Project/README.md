# RDS Environment Project

## Requirements
- **Hypervisor:** VMware ESXi 7.0 (Or any Hypervisor or Virtualization Platform)
- **Windows Server 2022 ISO:** (Not necessarily but it's much better to use)
- **DNS Server:** Pi-Hole (I'm utilizing my existing Pi-hole DNS Server since my goal here is to access the RDSH Servers outside the Domain)
- **Windows Server Roles**

| Server | Role | Description |
|--------|------|-------------|
| AD-SRV | AD DS + DNS | Identity/authentication, users/groups, domain and DNS |
| RDCB-SRV | RD Connection Broker | Maintains the RDS Deployment/session state and determines which RDSH should handle a connection |
| RDSH01 RDSH02| RD Session Host | Provides the actual Windows session/desktop and applications |
| RDWA-SRV | RD Web Access | Presents the published RDS resources to users through the web portal/feed and provides the connection resource they launch |

Objective:
- **To design, deploy and understand a centralized Windows Remote Desktop Services (RDS) environment** consisting of Active Directory, DNS, Connection Broker, Session Hosts, and Web Access.
- **To understand how RDS components interact** to authenticate users, publish remote desktop/applications, manage our user sessions, and distribute connections across available Session Host servers.
- **To provide users with centralized access to remote Windows desktops and applications** through an RDS deployment while maintaining centralized authentication and session management.
- **To establish a foundation for extending the environment** with additional RDS components such as RD Gateway for secure external access.

## Initial Setup

[IMAGE1]

VMware ESXi was utilized for creating a Windows Server RDS Environment.

## Phase 1 - Active Directory Setup and Joining all Windows Server to a Domain

[IMAGE2 - 5]

## Phase 2 - Setup Remote Desktop Connection Broker to manage RDSH Servers

[IMAGE6 - 8]

## Phase 3 - Register host record on DNS Pi-hole and Testing

[IMAGE9 - 15]
