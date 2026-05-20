#  Active Directory, Windows Server, DHCP

A complete step-by-step IT infrastructure setup guide covering DNS, Active Directory Domain Services, Windows Server 2025, Windows 10 client integration, DHCP

---


##  Project Overview
This project documents the setup of a smallenterprise IT environment built from scratch using Windows Server 2025. The infarastructure includes a fully functional domain controller with Active Directory, DNS, DHCP, Group Policy and a windows 10 client joined to the domain.

| Component | Description |
|---|---|
| Windows Server 2025 | Primary Domain Controller setup |
| DNS | Name resolution for domain and clients |
| Active Directory (AD DS) | Users, Groups, OUs, and Group Policy |
| DHCP | Automatic IP assignment for network clients |
| Windows 10 | Domain join and client configuration |

---

### Project Architecture
       ┌────────────────────────────────────────────────────────┐
       │                 THE PUBLIC INTERNET                    │
       └────────────────────────────────────────────────────────┘
                                   │
                                   ▼ [Traffic to public websites]
       ┌────────────────────────────────────────────────────────┐
       │             EDGE ROUTER / DEFAULT GATEWAY              │
       │                 (   192.168.1.2)                       │
       └────────────────────────────────────────────────────────┘
                                   │
           ┌───────────────────────┴───────────────────────┐
           │                                               │
           ▼ [Static IP: 192.168.1.1 ]                     ▼ [Dynamic IP via DHCP]
    ┌──────────────────────────────────────┐       ┌──────────────────────────────────────┐
    │        WINDOWS SERVER (DC01)         │       │          WINDOWS 10 CLIENT           │
    ├──────────────────────────────────────┤       ├──────────────────────────────────────┤
    │  • Domain:   cis.net                 │       │  • Hostname: MANAGER.cis.net         │
    │  • Identity: Active Directory        │       │  • Status: Domain-Joined Endpoint    │
    ├──────────────────────────────────────┤       ├──────────────────────────────────────┤
    │      ACTIVE DIRECTORY STRUCTURE      │       │          DHCP LEASE RECEIVED         │
    │                                      │       │ ┌──────────────────────────────────┐ │
    │  [cis.net (Root)]                    │       │ │ IP Address:  192.168.1.4         │ │
    │    ├──   computer-OU                 │       | │ Subnet Mask: 255.255.255.0       │ │
    │    │    └──   PC01 (Windows 10)      │ ════> │ │ Gateway:     192.168.1.2         │ │
    │    │                                 │       │ │ Primary DNS: 192.168.1.1         │ │
    │    └──   Users org U                 │       │ └──────────────────────────────────┘ │
    │         └──   Ambe Peter             │       │                                      │
    ├──────────────────────────────────────┤       │                                      │
    │               SERVICES               │       │                                      │
    │ ┌──────────────────────────────────┐ │       │                                      │
    │ │ DHCP Server (Leases IP & DNS)    │ │       │                                      │
    │ └──────────────────────────────────┘ │       │                                      │
    │ ┌──────────────────────────────────┐ │ <═══  │ [Queries AD for User Login Auth]     │
    │ │ DNS Server (Resolves Hostnames)  │ │  DNS  │                                      │
    │ └──────────────────────────────────┘ │ Traffic                                      │
    └──────────────────────────────────────┘       └──────────────────────────────────────┘

##  Prerequisites

- [ ] Windows Server 2022 ISO installed and activated
- [ ] Static IP assigned to the server **before** AD DS installation
- [ ] Windows 10 Pro client machine on the same network (internal)

---

##  Infrastructure Summary

```
Domain Name        : cis.net
Domain Controller  : DC01 (Windows Server 2022)
Server IP          : 192.168.1.1
DHCP Scope         : 192.168.1.1 – 192.168.1.9
Subnet Mask        : 255.255.255.0
Default Gateway    : 192.168.1.2
Client OS          : Windows 10 Pro
```

---

##  DNS Configuration
### On the Domain Controller 

1. Press Win + R → type ncpa.cpl → **Enter**
2. Right-click your network adapter → **Properties**
3. Double-click **IPv4** and set:
   Preffered DNS Server : 192.168.1.1
4. Change name to Server

---

## 1. Windows Server 2025 

### 1.1 Set a Static IP Address

1. Press `Win + R` → `ncpa.cpl` → **Enter**
2. Right-click adapter → **Properties**
3. Double-click **IPv4** and configure:

```
IP Address      : 192.168.1.1
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.1.2
Preferred DNS   : 192.168.1.1

```

## 2. Active Directory Domain Services (AD DS)

### 2.1 Install the AD DS Role

1. Server Manager → Manage → **Add Roles and Features**
2. Select Role-based or feature-based installation, Next
3. Select Server.cis.net -> Next
4. Check Active Directory Domain Services → Add Features and  Install

### 2.2 Promote Server to Domain Controller

1. Click the yellow flag in Server Manager at the top right → Promote this server to a domain controller
2. Select Add a new forest
3. Root domain name: `cis.net` → Next
4. Forest/Domain Functional Level: `Windows Server 2025` 
5. DSRM srong password → Next
6. Ignore the DNS delegation warning → Next
7. Confirm NetBIOS name: `CIS` → Next
8. Leave default AD database paths → Next
9. Review → Install (server will restart automatically)

 After reboot, log in with `CIS\Administrator`


### 2.3 Create Organizational Units (OUs)

1. On server manager, click on tools and navigate to Active Directory Users and computers
2. Right-click `cis.net` → New → Organizational Unit
3. Create 2 organisational Units, 1 for computers and the other for Users
4. Name OU for computers as (Computer-OU and that of users as ( Users org OU)

```


### 2.5 Create User Accounts

1. On cis.net
2. Right-click → New → User

First Name  : Ambe
Last Name   : Peter
Logon Name  : ambe@cis.net
Password    : (-----) (strong password)
Options     : User should not change passsword
Move this user (Ambe Peter to User org OU)


```

### 2.7 Configure Group Policy (GPO)

1. On server manager, click manage and navigate to Group Policy Management (GPO)
2. Expand Forest:cis.net -> Domains -> cis.net
3. Right-click on cis.net and create a GPO 
4. Name: `WORKSTATION
5. Right-click GPO → **Edit**
6. Navigate to: `Computer Configuration > Policies > Windows Settings > Security Settings`


Account Lockout : Lock after 5 failed attempts
Password Policy : Minimum 8 characters, complexity required
Audit Policy    : Log success & failure for logon events



## 3. DHCP Server Configuration

### 3.1 Install the DHCP Role

1. Server Manager → Manage → Add Roles and Features
2. Select DHCP Server → Add Features → Install
3. Click Complete DHCP configuration from the notification flag
4. Click Commit to authorize DHCP in Active Directory
5. Reboot 

### 3.2 Create a DHCP Scope

1. On server manager, click on manage and navigate to DHCP
2. Expand server.cis.net
3. Right-click IPv4 → New Scope
4. Name: CIS Network


|Start IP Address | 192.168.1.1|
|-------------|-----------------|
|End IP Address   |192.168.1.9|
|Subnet Mask      | 255.255.255.0|
|Lease Duration   | 8 Days|
|Default Gateway  | 192.168.1.2|
|DNS Server       |192.168.1.1|
|Exclusions       | 192.168.1.1, 192.168.1.2|
5. Select **Yes, I want to activate this scope now** → **Finish**

---

## 4. Windows 10 – Domain Join & Configuration

### 4.1 Configure DNS on Windows 10

### On Windows 10 Clients

1. Go to `ncpa.cpl` → right-click adapter → **Properties**
2. Open **IPv4** → set:

``
 Obtain DNS Server Address automatically

``



### Verify DNS is Working

```cmd
nslookup cis.net
ping cis.net
ipconfig /all
```

### 4.2 Join the Domain

1. Right-click **This PC** → **Properties** → **Rename this PC (advanced)**
2. Click **Change** → select **Domain**
3. Enter: `cis.net` → **OK**
4. Another prompt will come up
5. Credentials: `ambe@cis.net` + password (-----)
6. Click **OK** on welcome message → **Restart**

### 4.3 Log In with a Domain Account

1. Click **Other user** on login screen
2. Enter: `CIS\ambe` 
3. Enter password → **Enter**

### 4.4 Verify Domain Membership

```cmd
systeminfo | findstr /i cis.net
gpresult /r
```


## Quick Reference Commands

### Active Directory & DNS

```powershell
dsa.msc                   # Active Directory Users and Computers
dnsmgmt.msc               # DNS Manager
gpmc.msc                  # Group Policy Management Console
dhcpmgmt.msc              # DHCP Manager
dcdiag /v                 # Diagnose DC health
netdom query fsmo         # Check FSMO role holders
repadmin /replsummary     # Check AD replication status
nslookup corp.local       # Test DNS resolution
gpupdate /force           # Force Group Policy update
gpresult /r               # Show applied GPOs

```
---

## Screenshots

Before domain computer was configured to DHCP, it was first configured static to enable connectivity between the domain controller and domain computer

<img width="1920" height="1080" alt="Screenshot 2026-05-20 104304" src="https://github.com/user-attachments/assets/7ba47827-e175-4b41-89ad-f6fb52e19261" />

Server Manager and IP Adrress

<img width="1920" height="1080" alt="Screenshot 2026-05-20 140009" src="https://github.com/user-attachments/assets/8e8e8eb5-1875-4fcd-bd60-c1e45d819d16" />

Creating User

<img width="1920" height="1080" alt="Screenshot 2026-05-20 105428" src="https://github.com/user-attachments/assets/53235df8-bc3a-4a77-b2c3-74fe8dae7dc0" />

Joining Computer to Domain

<img width="1920" height="1080" alt="Screenshot 2026-05-20 110807" src="https://github.com/user-attachments/assets/68ea92c4-5f0c-4182-a711-e38910c95402" />

Configuring DHCP

<img width="1920" height="1080" alt="Screenshot 2026-04-11 212300" src="https://github.com/user-attachments/assets/06afe2e6-c3d9-43b1-a7a5-1a018387aca3" />

DHCP Address Leases

<img width="1920" height="1080" alt="Screenshot 2026-05-20 150010" src="https://github.com/user-attachments/assets/7a5488d1-cee4-4d90-b5be-72cf8876ee7d" />


Group Policy Object (Worksation) with Organizational Units (Users org U and Computer-OU)

<img width="1920" height="1080" alt="Screenshot 2026-05-20 142034" src="https://github.com/user-attachments/assets/fb1bd60c-5e6e-449f-9445-0a62f125bf21" />

Verifying DNS

<img width="1920" height="1080" alt="Screenshot 2026-05-20 150910" src="https://github.com/user-attachments/assets/5349839e-102c-4529-8b94-cccbd45c1ce4" />

---

## Lessons Learned
- Windows Server hosting DNS must have a fixed, static IP address so clients never lose connection to name resolution
- Installing active directory domain services role handles the bulk of complex DNS configuration automatically
- Active directory cannot function without DNS
- Group Policy doed not apply immediately after changes, running gpupdate /force on the client and rebooting is required to see the changes take effect

  ---
  
## What Next
- Implement a backup strategy using windows server backup
- Configure Remote Desktop Services (RDS) for remote access
- Set up  SIEM tool (Splunk) for log monitoring and alerting
- Implement multi-factor authentication (MFA) withmicrosoft authentication









