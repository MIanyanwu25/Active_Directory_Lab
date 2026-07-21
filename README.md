# Active_Directory_Lab# Lab 1 — Active Directory
### Windows Server 2025 · Azure Free Account · Identity & Access Management

![Azure](https://img.shields.io/badge/Azure-Free_Tier-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![Windows Server](https://img.shields.io/badge/Windows_Server-2025-0078D4?style=flat&logo=windows&logoColor=white)
![PowerShell](https://img.shields.io/badge/PowerShell-Scripted-5391FE?style=flat&logo=powershell&logoColor=white)
![Cost](https://img.shields.io/badge/Cost-%240-brightgreen?style=flat)

---

## Overview

| Field | Value |
|---|---|
| **Certification Alignment** | CompTIA Network+ · Security+ · Azure Administrator |
| **Tools Used** | Windows Server 2025 Evaluation · Azure Free Account |
| **Time to Complete** | 3–5 hours across multiple sessions |
| **Estimated Cost** | $0 — fully covered by free tiers and evaluation licences |
| **Career Relevance** | IT Support · Sysadmin · Cloud Engineer · Security Analyst |

Active Directory is the identity backbone of nearly every enterprise Windows environment. This lab builds a fully functional AD environment from scratch — and every concept directly transfers to Microsoft Entra ID (formerly Azure AD) in the cloud.

---

## The Business Problem This Lab Solves

Every organization that runs Windows infrastructure relies on Active Directory to answer one fundamental question: **who is allowed to do what?**

When a new employee joins, IT creates their account in AD and adds them to the right groups. Their access to email, shared drives, printers, and applications is granted automatically based on group membership. When they leave, IT disables one account and every door closes simultaneously.

This is not legacy technology. Hybrid environments sync on-premises AD identities to Entra ID in the cloud. This lab is foundational knowledge for anyone targeting a cloud or infrastructure role.

---

## Career Relevance by Role

| Role | How This Lab Applies |
|---|---|
| **IT Support / Help Desk** | Password resets, account unlocks, group membership changes — the top three ticket types in any enterprise |
| **Sysadmin** | Designing OU structure, deploying GPOs, managing domain-joined machines at scale |
| **Cloud Engineer** | Entra ID uses the same concepts: users, groups, roles, conditional access. On-prem AD knowledge transfers directly |
| **Security Analyst** | AD is the most targeted system in ransomware attacks. Understanding how it works is the foundation of defending it |

---

## What You'll Learn

| Skill | Real-World Application |
|---|---|
| Promote a Windows Server to Domain Controller | The first step in every enterprise Windows environment |
| Create organizational Units (OUs) | Apply different policies to different departments from one place |
| Create users, groups, and group memberships | Every access decision in an enterprise is group-based |
| Configure Group Policy Objects (GPOs) | Enforce settings across every machine in the domain centrally |
| Join a machine to the domain | Connect a workstation as a managed, policy-enforced resource |
| Configure role-based access with security groups | Apply least privilege — users only get what their job requires |
| Reset passwords and manage account lifecycle | The most frequent real-world task for IT support |

---

## Prerequisites

- An Azure free account — [create one here](https://azure.microsoft.com/free) **or** a local machine with 8GB+ RAM for VirtualBox
- Remote Desktop client (built into Windows; download [Microsoft Remote Desktop](https://apps.apple.com/us/app/microsoft-remote-desktop/id1295203466) for Mac)
- Basic familiarity with Windows Server (helpful but not required)

---

## Environment Setup

### Option A — Azure (Recommended)

Running in Azure means no local hardware requirements. The VM runs in Microsoft's data centre and costs $0 within the free tier.

1. Go to [azure.microsoft.com/free](https://azure.microsoft.com/free) and create a free account
2. Sign in to [portal.azure.com](https://portal.azure.com)
3. Search for **Virtual machines** and click **Create**
4. Use the settings below:

| Setting | Value | Why |
|---|---|---|
| Region | East US | Cheapest region, most available VM sizes under free tier |
| Image | Windows Server 2025 Datacenter — Gen2 | Latest server OS, includes free 180-day evaluation licence |
| Size | Standard_B2s (2 vCPU, 4GB RAM) | Smallest size that runs AD comfortably |
| Authentication | Password | Set a strong password — you'll use this to RDP in |
| Public inbound ports | Allow RDP (3389) | Required to connect from your local machine |
| OS disk | Standard SSD | Good performance, included in free tier storage |

> ⚠️ **Cost tip:** Stop the VM (don't delete it) when you're not using it. A B2s VM costs roughly $0.05/hour. Stopping pauses compute billing so your $200 free credit lasts much longer.

**Fix clipboard before connecting:**
1. Open the Remote Desktop application on your local machine
2. Enter the VM's public IP address → click **Show Options**
3. Click the **Local Resources** tab → check **Clipboard**
4. Click **Connect**

> 💡 If connecting through the Azure portal browser console, download the RDP file instead (Connect → Download RDP File) and open it with the native Remote Desktop app. This is the recommended approach for all lab work.

### Option B — VirtualBox (Local)

- Download [VirtualBox](https://www.virtualbox.org/) — free, no account required
- Download the [Windows Server 2025 Evaluation ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025) from Microsoft
- Create a VM: 4GB RAM minimum, 60GB disk, Windows Server 2019/2022 as the type
- Mount the ISO and follow the installation wizard — select **Datacenter with Desktop Experience**

> Minimum host specs: 8GB RAM, 60GB free disk space, quad-core CPU with virtualisation enabled in BIOS.

---

## Lab Steps

### Step 1 — Install Active Directory Domain Services

RDP into your VM. Open Server Manager — it opens automatically on login.

**GUI:** Manage → Add Roles and Features → Server Roles → check **Active Directory Domain Services** → Add Features → Install

**PowerShell:**
```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

**Install Group Policy Management Console (GPMC) now — don't skip this:**
```powershell
Install-WindowsFeature -Name GPMC
```

> ⚠️ Install GPMC before you need it. If it's not installed, Group Policy Management won't appear in Server Manager and you'll hit a wall in Step 5.

---

### Step 2 — Promote the Server to a Domain Controller

Installing the AD DS role does not create a domain. Promotion is the step that creates your forest, your domain, and makes this server the authoritative DNS and identity server.

**GUI:**
1. Click the yellow warning flag in Server Manager → **Promote this server to a domain controller**
2. Select **Add a new forest**
3. Set Root domain name to: `lab.local`
4. Set a DSRM password and click through to **Install**
5. The server will restart automatically when complete

**PowerShell:**
```powershell
Import-Module ADDSDeployment

Install-ADDSForest `
  -DomainName 'lab.local' `
  -DomainNetBiosName 'LAB' `
  -InstallDns:$true `
  -SafeModeAdministratorPassword (ConvertTo-SecureString 'YourDSRMPassword!' -AsPlainText -Force) `
  -Force:$true
```

---

### Step 3 — Build the organizational Structure

Open **Active Directory Users and Computers (ADUC)** from the Tools menu in Server Manager.

**Create organizational Units:**
```powershell
New-ADOrganizationalUnit -Name "IT"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Finance"   -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "HR"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Sales"     -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Computers" -Path "DC=lab,DC=local"
```

**Create Security Groups:**
```powershell
New-ADGroup -Name "IT_Admins"     -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
New-ADGroup -Name "Finance_Users" -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
New-ADGroup -Name "HR_Users"      -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=lab,DC=local"
New-ADGroup -Name "Sales_Users"   -GroupScope Global -GroupCategory Security -Path "OU=Sales,DC=lab,DC=local"
```

**Create Users and Assign to Groups:**

> ⚠️ Run the entire block below at once — do not run line by line. The `$password` variable must be defined before the `New-ADUser` commands or the script will fail.

```powershell
# Step 1 — define the password variable first
$password = ConvertTo-SecureString "Welcome@2026!" -AsPlainText -Force

# Step 2 — create all 4 users
New-ADUser -Name "alice.chen" -GivenName "Alice" -Surname "Chen" `
  -SamAccountName "alice.chen" -UserPrincipalName "alice.chen@lab.local" `
  -Path "OU=IT,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "bob.patel" -GivenName "Bob" -Surname "Patel" `
  -SamAccountName "bob.patel" -UserPrincipalName "bob.patel@lab.local" `
  -Path "OU=Finance,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "carol.jones" -GivenName "Carol" -Surname "Jones" `
  -SamAccountName "carol.jones" -UserPrincipalName "carol.jones@lab.local" `
  -Path "OU=HR,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "david.smith" -GivenName "David" -Surname "Smith" `
  -SamAccountName "david.smith" -UserPrincipalName "david.smith@lab.local" `
  -Path "OU=Sales,DC=lab,DC=local" -AccountPassword $password -Enabled $true

# Step 3 — add each user to their department group
Add-ADGroupMember -Identity "IT_Admins"     -Members "alice.chen"
Add-ADGroupMember -Identity "Finance_Users" -Members "bob.patel"
Add-ADGroupMember -Identity "HR_Users"      -Members "carol.jones"
Add-ADGroupMember -Identity "Sales_Users"   -Members "david.smith"
```

---

### Step 4 — Configure Group Policy

Open **Group Policy Management** from the Tools menu in Server Manager.

1. Expand **Forest: lab.local → Domains → lab.local**
2. Right-click the **IT OU** → Create a GPO in this domain and link it here
3. Name it: `IT Security Policy`
4. Right-click the new GPO → **Edit**
5. Configure the following settings:

| Policy Path | Setting | Value |
|---|---|---|
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Minimum password length | 12 |
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Password must meet complexity requirements | Enabled |
| Computer Config → Windows Settings → Security → Local Policies → Security Options | Interactive logon: Machine inactivity limit | 900 seconds |
| Computer Config → Administrative Templates → System → Removable Storage Access | All removable storage classes: Deny all access | Enabled |

> 💡 Test your GPO: join a second VM to lab.local, move its computer account into the IT OU, run `gpupdate /force`, then log in as `alice.chen` and verify the screen lock policy applies.

---

### Step 5 — Common Help Desk Tasks

**Reset a password:**
```powershell
Set-ADAccountPassword -Identity "bob.patel" -Reset `
  -NewPassword (ConvertTo-SecureString "NewPass@2026!" -AsPlainText -Force)
Set-ADUser -Identity "bob.patel" -ChangePasswordAtLogon $true
```

**Unlock a locked account:**
```powershell
Unlock-ADAccount -Identity "carol.jones"
```

**Disable an account (offboarding):**
```powershell
# Disable the account — preserves history and group memberships for audit
Disable-ADAccount -Identity "david.smith"

# Find all disabled accounts
Search-ADAccount -AccountDisabled | Select-Object Name, SamAccountName
```

**Audit inactive accounts:**
```powershell
# Find accounts that haven't logged in for 90 days
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} `
  -Properties LastLogonDate | Select-Object Name, LastLogonDate

# Check group membership for a specific user
Get-ADPrincipalGroupMembership -Identity "alice.chen" | Select-Object Name
```

---

## Verification

Run these commands to confirm everything is working before you wrap up:

| Check | Command | Expected Result |
|---|---|---|
| Domain controller is running | `Get-ADDomainController` | Returns DC info including forest lab.local |
| OUs exist | `Get-ADOrganizationalUnit -Filter *` | Lists all 5 OUs you created |
| Users exist and are enabled | `Get-ADUser -Filter {Enabled -eq $true}` | Lists your 4 test accounts |
| Group memberships correct | `Get-ADGroupMember -Identity IT_Admins` | Returns alice.chen |
| GPO is linked | `Get-GPInheritance -Target 'OU=IT,DC=lab,DC=local'` | Shows IT Security Policy as linked |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| PowerShell prompts for `Name:` when creating users | Run the entire user creation block at once — `$password` must be defined first |
| Cannot copy and paste into the VM | Open RDP client → Show Options → Local Resources → check Clipboard. Or download the RDP file from Azure portal and use the native Remote Desktop app |
| Promotion fails: DNS conflict | Set the NIC's preferred DNS to `127.0.0.1` before promoting |
| Cannot RDP after domain join | Log in as `LAB\Administrator`, not just `Administrator` |
| GPO not applying | Run `gpupdate /force` on the target machine, then `gpresult /r` to see applied policies |
| User cannot log in after creation | Confirm the account is Enabled and `ChangePasswordAtLogon` is not blocking login |
| AD Users and Computers not showing | Run `dsa.msc` from the Run dialog, or run `Add-WindowsFeature RSAT-ADDS` |

---

## Key Concepts Reference

| Term | What It Is |
|---|---|
| **Domain Controller (DC)** | The server that runs Active Directory and handles all authentication decisions |
| **Forest** | The top-level container of your entire AD structure — represents the organization |
| **Domain** | A boundary inside the forest with a name (e.g., `lab.local`) — everything inside is managed together |
| **organizational Unit (OU)** | A folder in AD used to organise users, computers, and groups — and link Group Policy |
| **Security Group** | A container of user accounts used to grant access to resources at scale |
| **Group Policy Object (GPO)** | A collection of settings automatically applied to users and computers inside an OU |
| **DSRM** | Directory Services Restore Mode — emergency recovery access; write the password down |

---

## Lab Series

This is **Lab 1** of an ongoing hands-on Azure and infrastructure lab series.

| Lab | Topic | Status |
|---|---|---|
| Lab 1 | Active Directory | ✅ Complete |
| Lab 2 | Coming soon | 🔄 In progress |

---

## Connect

If this lab was useful, follow along — more labs covering Azure networking, Infrastructure as Code (Bicep/Terraform), and Azure DevOps pipelines are on the way.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/michaelianyanwu)
