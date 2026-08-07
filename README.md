# Active Directory Setup and Configuration in Azure Microsoft Windows Server

## **Deploy and configure Active Directory Domain Services (AD DS) on a Windows Server virtual machine hosted in Microsoft Azure.**

![Platform](https://img.shields.io/badge/platform-Azure-0078D4?logo=microsoftazure&logoColor=white)
![OS](https://img.shields.io/badge/OS-Windows%20Server%202025-00A4EF)

---

## Overview

## This guide provides step-by-step instructions to deploy and configure Active Directory Domain Services (AD DS) on a Windows Server virtual machine hosted in **Microsoft Azure**. 

## Watch Me Build This Lab Here!

<!-- Replace with your own screen recording link (Loom, YouTube, etc.) once you have one -->
- *[Add a walkthrough video link here]*

---

## Prerequisites

- Azure account set up with appropriate permissions
- Existing or new Resource Group
- Network Security Group (NSG) allowing RDP (port 3389)
- Windows Server 2025 Datacenter

---

## Architecture

The lab consists of a single Domain Controller running AD DS + DNS for the `lab.local` forest, an organisational unit structure mapped to departments, role-based security groups, and a Group Policy Object enforcing baseline security settings across the IT OU. A second VM domain-joins to validate that policy is actually being pushed and enforced.

<!-- ![Create VM](your-screenshot-url-here) --> <img width="199" height="150" alt="azure-ad-lab-architecture" src="https://github.com/user-attachments/assets/d5f4e436-a398-4f5f-a22c-1fdbbde8eff3" />
<svg viewBox="0 0 1140 860" xmlns="http://www.w3.org/2000/svg" role="img">
<title>Azure Active Directory Lab Architecture</title>
<desc>Diagram of an Azure resource group hosting a domain controller VM, connected admin workstation and GitHub repo, and the resulting Active Directory OU structure for lab.local.</desc>

<defs>
<marker id="arrowTeal" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
<path d="M2 1L8 5L2 9" fill="none" stroke="#0F6E56" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
</marker>
<marker id="arrowPink" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
<path d="M2 1L8 5L2 9" fill="none" stroke="#993556" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
</marker>
</defs>

<rect x="8" y="8" width="1124" height="844" rx="16" fill="#FAFAF8" stroke="#E3E1D8" stroke-width="1"/>

<text x="570" y="42" text-anchor="middle" font-family="sans-serif" font-size="22" font-weight="700" fill="#26215C">Active Directory Lab 1 — Azure Architecture</text>
<text x="570" y="64" text-anchor="middle" font-family="sans-serif" font-size="13" fill="#534AB7">Domain: lab.local · OS: Windows Server 2025 Datacenter · Region: East US 2</text>

<rect x="40" y="85" width="1060" height="475" rx="14" fill="#EEEDFE" stroke="#534AB7" stroke-width="2" stroke-dasharray="6,4"/>
<text x="58" y="110" font-family="sans-serif" font-size="14" font-weight="700" fill="#3C3489">☁ Microsoft Azure</text>

<rect x="60" y="122" width="1020" height="418" rx="10" fill="#F1EFE8" fill-opacity="0.6" stroke="#5F5E5A" stroke-width="1.5" stroke-dasharray="5,3"/>
<text x="76" y="145" font-family="sans-serif" font-size="12.5" font-weight="700" fill="#444441">Resource Group: Lab1-RG</text>

<rect x="80" y="160" width="635" height="345" rx="8" fill="#E1F5EE" fill-opacity="0.5" stroke="#0F6E56" stroke-width="1.5" stroke-dasharray="4,3"/>
<text x="96" y="182" font-family="sans-serif" font-size="12" font-weight="700" fill="#085041">Virtual Network: Lab1-VM-vnet / default</text>

<rect x="100" y="196" width="230" height="58" rx="6" fill="#FAECE7" stroke="#993C1D" stroke-width="1.5"/>
<text x="112" y="217" font-family="sans-serif" font-size="11.5" font-weight="700" fill="#712B13">🛡 NSG: Lab1-VM-nsg</text>
<text x="112" y="236" font-family="sans-serif" font-size="10.5" fill="#993C1D">RDP TCP 3389 allowed</text>

<rect x="100" y="270" width="595" height="210" rx="8" fill="#FFFFFF" stroke="#0F6E56" stroke-width="2"/>
<path d="M108 270 H687 A8 8 0 0 1 695 278 V306 H100 V278 A8 8 0 0 1 108 270 Z" fill="#0F6E56"/>
<text x="397" y="293" text-anchor="middle" font-family="sans-serif" font-size="13" font-weight="700" fill="#FFFFFF">🖥 Virtual Machine: Lab1-VM</text>

<text x="118" y="322" font-family="sans-serif" font-size="12" font-weight="700" fill="#085041">Role:</text>
<text x="232" y="322" font-family="sans-serif" font-size="12" fill="#444441">Active Directory Domain Controller</text>
<text x="118" y="339" font-family="sans-serif" font-size="12" font-weight="700" fill="#085041">OS:</text>
<text x="232" y="339" font-family="sans-serif" font-size="12" fill="#444441">Windows Server 2025 Datacenter</text>
<text x="118" y="356" font-family="sans-serif" font-size="12" font-weight="700" fill="#085041">Size:</text>
<text x="232" y="356" font-family="sans-serif" font-size="12" fill="#444441">Standard_B2ls_v2 (2 vCPU / 4GB RAM)</text>
<text x="118" y="373" font-family="sans-serif" font-size="12" font-weight="700" fill="#085041">Public IP:</text>
<text x="232" y="373" font-family="sans-serif" font-size="12" fill="#444441">52.247.113.186</text>
<text x="118" y="390" font-family="sans-serif" font-size="12" font-weight="700" fill="#085041">Private IP:</text>
<text x="232" y="390" font-family="sans-serif" font-size="12" fill="#444441">10.0.0.4</text>
<text x="118" y="407" font-family="sans-serif" font-size="12" font-weight="700" fill="#085041">NIC:</text>
<text x="232" y="407" font-family="sans-serif" font-size="12" fill="#444441">lab1-vm667 (primary)</text>
<text x="118" y="424" font-family="sans-serif" font-size="12" font-weight="700" fill="#085041">Services:</text>
<text x="232" y="424" font-family="sans-serif" font-size="12" fill="#444441">AD DS · DNS · GPMC</text>
<text x="118" y="441" font-family="sans-serif" font-size="12" font-weight="700" fill="#085041">Access:</text>
<text x="232" y="441" font-family="sans-serif" font-size="12" fill="#444441">RDP (port 3389)</text>
<text x="118" y="458" font-family="sans-serif" font-size="12" font-weight="700" fill="#085041">Automation:</text>
<text x="232" y="458" font-family="sans-serif" font-size="12" fill="#444441">deploy-ad.ps1 (PowerShell)</text>

<rect x="755" y="160" width="315" height="85" rx="8" fill="#FAEEDA" stroke="#854F0B" stroke-width="1.5"/>
<text x="770" y="183" font-family="sans-serif" font-size="12" font-weight="700" fill="#633806">📄 Subscription</text>
<text x="770" y="203" font-family="sans-serif" font-size="11" fill="#854F0B">SGL Azure 2</text>
<text x="770" y="221" font-family="sans-serif" font-size="10" fill="#8A8878">Created: 8/3/2026 12:34 AM UTC</text>

<rect x="755" y="260" width="315" height="100" rx="8" fill="#FBEAF0" stroke="#993556" stroke-width="1.5"/>
<text x="770" y="283" font-family="sans-serif" font-size="12" font-weight="700" fill="#72243E">💻 Admin Workstation</text>
<text x="770" y="303" font-family="sans-serif" font-size="11" fill="#993556">Remote Desktop (RDP)</text>
<text x="770" y="321" font-family="monospace" font-size="11" fill="#993556">→ 52.247.113.186:3389</text>

<rect x="755" y="375" width="315" height="85" rx="8" fill="#F1EFE8" stroke="#5F5E5A" stroke-width="1.5"/>
<text x="770" y="398" font-family="sans-serif" font-size="12" font-weight="700" fill="#2C2C2A">⚙ GitHub Repository</text>
<text x="770" y="417" font-family="sans-serif" font-size="9" fill="#5F5E5A">shirleylandondata/azure-active-directory-domain-lab</text>
<text x="770" y="432" font-family="sans-serif" font-size="9.5" fill="#888780">README.md · Infrastructure docs</text>

<line x1="755" y1="300" x2="700" y2="333" stroke="#993556" stroke-width="1.5" stroke-dasharray="5,4" marker-end="url(#arrowPink)"/>

<rect x="40" y="580" width="1060" height="250" rx="14" fill="#E1F5EE" fill-opacity="0.5" stroke="#0F6E56" stroke-width="2" stroke-dasharray="6,4"/>
<text x="58" y="605" font-family="sans-serif" font-size="14" font-weight="700" fill="#085041">🗄 Active Directory — lab.local</text>

<line x1="397" y1="480" x2="570" y2="615" stroke="#0F6E56" stroke-width="1.5" stroke-dasharray="5,4" marker-end="url(#arrowTeal)"/>
<rect x="440" y="536" width="90" height="20" fill="#FAFAF8"/>
<text x="485" y="551" text-anchor="middle" font-family="sans-serif" font-size="10.5" fill="#0F6E56">Hosts AD DS</text>

<rect x="490" y="620" width="160" height="42" rx="6" fill="#0F6E56"/>
<text x="570" y="646" text-anchor="middle" font-family="sans-serif" font-size="12.5" font-weight="700" fill="#FFFFFF">DC=lab,DC=local</text>

<line x1="570" y1="662" x2="570" y2="675" stroke="#0F6E56" stroke-width="2"/>
<line x1="185" y1="675" x2="956" y2="675" stroke="#0F6E56" stroke-width="2"/>
<line x1="185" y1="675" x2="185" y2="690" stroke="#0F6E56" stroke-width="2"/>
<line x1="442" y1="675" x2="442" y2="690" stroke="#0F6E56" stroke-width="2"/>
<line x1="699" y1="675" x2="699" y2="690" stroke="#0F6E56" stroke-width="2"/>
<line x1="956" y1="675" x2="956" y2="690" stroke="#0F6E56" stroke-width="2"/>

<rect x="70" y="690" width="230" height="130" rx="8" fill="#EEEDFE" stroke="#534AB7" stroke-width="1.5"/>
<path d="M78 690 H292 A8 8 0 0 1 300 698 V718 H70 V698 A8 8 0 0 1 78 690 Z" fill="#534AB7"/>
<text x="185" y="708" text-anchor="middle" font-family="sans-serif" font-size="11.5" font-weight="700" fill="#FFFFFF">OU=IT</text>
<text x="82" y="736" font-family="sans-serif" font-size="11" fill="#3C3489">👤 alice.chen</text>
<text x="82" y="753" font-family="sans-serif" font-size="11" fill="#3C3489">→ IT_Admins</text>
<text x="82" y="773" font-family="sans-serif" font-size="10.5" fill="#3C3489">🛡 GPO: IT Security Policy</text>
<text x="82" y="791" font-family="sans-serif" font-size="10" font-style="italic" fill="#085041">Screensaver lock enforced</text>

<rect x="327" y="690" width="230" height="130" rx="8" fill="#FAEEDA" stroke="#854F0B" stroke-width="1.5"/>
<path d="M335 690 H549 A8 8 0 0 1 557 698 V718 H327 V698 A8 8 0 0 1 335 690 Z" fill="#854F0B"/>
<text x="442" y="708" text-anchor="middle" font-family="sans-serif" font-size="11.5" font-weight="700" fill="#FFFFFF">OU=Finance</text>
<text x="339" y="736" font-family="sans-serif" font-size="11" fill="#633806">👤 bob.patel</text>
<text x="339" y="753" font-family="sans-serif" font-size="11" fill="#633806">→ Finance_Users</text>

<rect x="584" y="690" width="230" height="130" rx="8" fill="#FAECE7" stroke="#993C1D" stroke-width="1.5"/>
<path d="M592 690 H806 A8 8 0 0 1 814 698 V718 H584 V698 A8 8 0 0 1 592 690 Z" fill="#993C1D"/>
<text x="699" y="708" text-anchor="middle" font-family="sans-serif" font-size="11.5" font-weight="700" fill="#FFFFFF">OU=HR</text>
<text x="596" y="736" font-family="sans-serif" font-size="11" fill="#712B13">👤 carol.jones</text>
<text x="596" y="753" font-family="sans-serif" font-size="11" fill="#712B13">→ HR_Users</text>

<rect x="841" y="690" width="230" height="130" rx="8" fill="#FBEAF0" stroke="#993556" stroke-width="1.5"/>
<path d="M849 690 H1063 A8 8 0 0 1 1071 698 V718 H841 V698 A8 8 0 0 1 849 690 Z" fill="#993556"/>
<text x="956" y="708" text-anchor="middle" font-family="sans-serif" font-size="11.5" font-weight="700" fill="#FFFFFF">OU=Sales</text>
<text x="853" y="736" font-family="sans-serif" font-size="11" fill="#72243E">👤 david.smith</text>
<text x="853" y="753" font-family="sans-serif" font-size="11" fill="#72243E">→ Sales_Users</text>

</svg>

---

## Steps

### 1. Launch the Windows Server Virtual Machine

1. Sign in to the **Azure Portal** (portal.azure.com).
2. Go to **Virtual machines → Create → Azure virtual machine**.

<!-- ![Create VM](your-screenshot-url-here) --> <img width="1061" height="782" alt="VM_lab1 1" src="https://github.com/user-attachments/assets/b324c670-3415-478f-b83d-5cc72713bdd4" />

3. Choose the **Windows Server 2025 Datacenter (Gen2)** image.
4. Select a VM size — `Standard_B2s` (2 vCPU / 4GB RAM) is enough for this lab and fits within free-tier credit.

<!-- ![Select image and size](your-screenshot-url-here) --> <img width="1062" height="832" alt="VM_lab1 2" src="https://github.com/user-attachments/assets/3c06a9c3-2f00-42b9-904f-26c8bdbba27b" />

<!-- ![Choose VM size](your-screenshot-url-here) -->

5. Configure **networking**:
   - Place the VM in your target Virtual Network (VNet) and subnet.
   - Leave **Public inbound ports** open for RDP if connecting over the internet.
6. Set **Authentication type** to Password and record a strong admin password.
7. Configure the **Network Security Group (NSG)** to allow:
   - TCP 3389 (RDP) from your IP

<!-- ![Configure NSG](your-screenshot-url-here) --> <img width="1063" height="843" alt="VM Lab1 3" src="https://github.com/user-attachments/assets/469866ca-4f1f-4de5-bef8-e6330c458981" />


8. Click **Review + Create**, then **Create**.

<!-- ![VM created](your-screenshot-url-here) --> <img width="1457" height="428" alt="VM_Lab1 4" src="https://github.com/user-attachments/assets/e95efc5c-c685-4b27-a7e6-75c838741652" />


> Stop the VM (don't delete it) between sessions — a `Standard_B2s` VM costs roughly $0.05/hour, and stopping it pauses compute billing while preserving your configuration.

---

### 2. Connect to the Instance via RDP

1. Once the VM is running, note its **public IP address** from the Overview page.
2. In the Azure Portal, click **Connect → RDP → Download RDP File**.
   
<!-- ![Connect via RDP](your-screenshot-url-here) --> <img width="1276" height="655" alt="VM_Lab1 5" src="https://github.com/user-attachments/assets/28c244da-aa18-4ca1-a3d8-15b6c33f591a" />
3. Open the downloaded `.rdp` file with the native Remote Desktop app (not the browser console — clipboard sharing is limited there).
<!-- ![Connect via RDP](your-screenshot-url-here) --> <img width="542" height="592" alt="Screenshot 2026-08-06 234258" src="https://github.com/user-attachments/assets/cec121b0-3b3c-4efe-bdbd-2b7a1e12193e" />

4. Before connecting, click **Show Options → Local Resources tab** and check **Clipboard** so you can paste commands into the VM.
5. Connect using the admin username and password you set at VM creation.

<!-- ![Enable clipboard sharing](your-screenshot-url-here) --> <img width="543" height="597" alt="Screenshot 2026-08-06 234212" src="https://github.com/user-attachments/assets/f73eef91-5fa3-496b-a972-be215cb3c99f" />


---

### 3. Install Active Directory Domain Services (AD DS)

1. In **Server Manager**, select **Manage → Add Roles and Features**.
2. Proceed through the wizard:
   - Select **Role-based or feature-based installation**.
   - Choose your server.
   - Check **Active Directory Domain Services**.
   - Add required management tools and install.

<!-- ![Add Roles and Features wizard](your-screenshot-url-here) --> <img width="1417" height="778" alt="Screenshot 2026-08-06 235519" src="https://github.com/user-attachments/assets/d1664fb2-bb87-4132-accb-dca8be747a46" />

<!-- ![Select AD DS role](your-screenshot-url-here) --> <img width="982" height="705" alt="Screenshot 2026-08-06 235820" src="https://github.com/user-attachments/assets/2ac9f1f3-dbb9-47e9-b305-4e31f97b0059" />


Or install via PowerShell:

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
Install-WindowsFeature -Name GPMC   # Group Policy Management Console — needed later, install now
```

---

### 4. Promote the Server to a Domain Controller

1. After installation, click the **flag icon** in Server Manager.
2. Select **Promote this server to a domain controller**.
3. Choose:
   - **Add a new forest**
   - Root domain name: `lab.local`
4. Set a **Directory Services Restore Mode (DSRM)** password.
5. Accept the default DNS and NetBIOS options, complete the wizard, and let the server restart.

<!-- ![Promote to domain controller](your-screenshot-url-here) -->
<!-- ![Set root domain name](your-screenshot-url-here) -->
<!-- ![Installation progress](your-screenshot-url-here) -->

Or promote via PowerShell:

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

### 5. Build Out the Directory Structure

1. Log in again after reboot.
2. Open **Active Directory Users and Computers (ADUC)** from the Tools menu.
3. Create Organizational Units (OUs) for each department.
4. Create role-based security groups inside each OU.
5. Create test user accounts and add them to the appropriate group.

<!-- ![Active Directory Users and Computers](your-screenshot-url-here) -->
<!-- ![Creating an OU](your-screenshot-url-here) --> <img width="981" height="687" alt="Screenshot 2026-08-07 000631" src="https://github.com/user-attachments/assets/b4755f01-cf30-4625-a18a-21e5e97ef1d7" />

<!-- ![Creating a new user](your-screenshot-url-here) --> <img width="552" height="509" alt="Screenshot 2026-08-07 001421" src="https://github.com/user-attachments/assets/5b946f92-d9b4-4dde-8617-3b2fc3754dfc" />


```powershell
# Organizational Units
New-ADOrganizationalUnit -Name "IT"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Finance"   -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "HR"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Sales"     -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Computers" -Path "DC=lab,DC=local"

# Security groups
New-ADGroup -Name "IT_Admins"     -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
New-ADGroup -Name "Finance_Users" -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
New-ADGroup -Name "HR_Users"      -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=lab,DC=local"
New-ADGroup -Name "Sales_Users"   -GroupScope Global -GroupCategory Security -Path "OU=Sales,DC=lab,DC=local"

# Users
$password = ConvertTo-SecureString "Welcome@2026!" -AsPlainText -Force

New-ADUser -Name "alice.chen" -GivenName "Alice" -Surname "Chen" `
  -SamAccountName "alice.chen" -UserPrincipalName "alice.chen@lab.local" `
  -Path "OU=IT,DC=lab,DC=local" -AccountPassword $password -Enabled $true

Add-ADGroupMember -Identity "IT_Admins" -Members "alice.chen"
# ...repeated per department
```

---

### 6. Configure and Link a Group Policy Object (GPO)

1. Open **Group Policy Management** from the Tools menu.
2. Right-click the **IT** OU → **Create a GPO in this domain and link it here**.
3. Name it `IT Security Policy` and edit it to configure:

| Setting | Value | Purpose |
|---|---|---|
| Minimum password length | 12 | Enforce strong passwords |
| Password complexity | Enabled | Require mixed case, number, symbol |
| Machine inactivity limit | 900 sec | Auto-lock idle sessions |
| Removable storage access | Deny all | Block USB-based data exfiltration |

<!-- ![Group Policy Management console](your-screenshot-url-here) -->
<!-- ![GPO settings](your-screenshot-url-here) -->

4. **Verify it actually works:** join a second VM to `lab.local`, move its computer object into the `IT` OU, run `gpupdate /force`, and confirm the screen-lock policy applies on login.

---

## Common Help Desk Tasks Practiced

```powershell
# Password reset with forced change on next login
Set-ADAccountPassword -Identity "bob.patel" -Reset -NewPassword (ConvertTo-SecureString "NewPass@2026!" -AsPlainText -Force)
Set-ADUser -Identity "bob.patel" -ChangePasswordAtLogon $true

# Unlock a locked account
Unlock-ADAccount -Identity "carol.jones"

# Offboarding — disable, don't delete (preserves audit history)
Disable-ADAccount -Identity "david.smith"

# Audit — accounts inactive for 90+ days
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} -Properties LastLogonDate
```

---

## Verification

| Check | Command | Expected result |
|---|---|---|
| DC is running | `Get-ADDomainController` | Returns forest `lab.local` |
| OUs exist | `Get-ADOrganizationalUnit -Filter *` | Lists all 5 OUs |
| Users exist and enabled | `Get-ADUser -Filter {Enabled -eq $true}` | Lists test accounts |
| Group membership correct | `Get-ADGroupMember -Identity IT_Admins` | Returns `alice.chen` |
| GPO linked | `Get-GPInheritance -Target 'OU=IT,DC=lab,DC=local'` | Shows `IT Security Policy` |

---

## Troubleshooting

| Problem | Root cause / fix |
|---|---|
| **AD Organizational Unit path mismatch** (`DC=lab` vs `DC=Lab1VM`) | The domain's actual distinguished-name components didn't match what was assumed when writing OU paths. Confirmed the real domain DN with `Get-ADDomain \| Select DistinguishedName` and corrected every `-Path` argument to match exactly — PowerShell will not fuzzy-match a DN |
| **Built-in `Computers` container conflict** | Windows auto-creates a default `Computers` container (not a true OU) at domain creation, which cannot have GPOs linked to it. Machine objects landing there silently ignored the GPO. Fix was moving computer objects into the purpose-built `Computers` OU, or redirecting the default computer container with `redircmp` |
| **Password complexity preventing account enablement** | New accounts failed to enable because the chosen password didn't meet the domain's complexity policy (upper/lower/number/symbol, minimum length). Fixed by generating passwords that satisfied the policy before calling `-Enabled $true`, rather than enabling first and troubleshooting the failure after |
| **DNS configuration preventing domain join** | Client machines pointed at a public or ISP-provided DNS server instead of the Domain Controller, so they couldn't resolve `lab.local` or locate the domain via SRV records. Fixed by setting the client NIC's preferred DNS server to the DC's static IP before attempting the join |
| **Remote Desktop authorization for domain users** | Domain accounts couldn't RDP into member servers by default — only local Administrators group members can, out of the box. Fixed by adding the relevant domain group to the target machine's **Remote Desktop Users** local group |

---

## Lessons Learned

- **Active Directory relies on DNS for domain discovery.** A domain join isn't really an AD problem until DNS is ruled out first — clients locate the DC via DNS SRV records, and nothing else works if that resolution fails.
- **Group Policy only applies when objects are in the correct OU.** Membership in the right *security group* is not the same as location in the right *OU* — GPOs link to OUs, not groups, and a misplaced computer object will never receive the policy no matter how the groups are configured.
- **PowerShell distinguished names must match the domain exactly.** There's no fuzzy matching on `-Path` — the DN has to mirror the actual domain structure character-for-character, which means verifying it rather than assuming it.
- **Built-in Active Directory containers differ from Organizational Units.** The default `Users` and `Computers` containers look like OUs in the GUI but functionally aren't — they can't have GPOs linked to them, which trips up anyone assuming they can.
- **Domain-joined clients validate GPO deployment.** A GPO that looks correctly configured and linked in Group Policy Management isn't actually proven until a real client pulls it down and the setting takes effect — configuration and verification are two different steps.

---

## Cleanup (Optional)

If testing only, stop or deallocate the VM to avoid ongoing charges:

```powershell
# From Azure CLI, to deallocate and stop billing
az vm deallocate --resource-group <rg-name> --name <vm-name>
```

To remove it entirely once you're done with the lab:

```powershell
az vm delete --resource-group <rg-name> --name <vm-name>
```

---

## Future Improvements

- Deploy a second Domain Controller for redundancy and to practice AD replication
- Configure DHCP for automatic client addressing
- Implement Group Policy Preferences for more granular, item-level configuration
- Configure roaming user profiles
- Add Azure VPN connectivity for hybrid on-prem/cloud access
- Integrate Microsoft Entra ID (Azure AD) for hybrid identity sync

---

## Technologies

- Microsoft Azure
- Windows Server 2025
- Active Directory Domain Services
- DNS
- PowerShell
- Remote Desktop Protocol
- Group Policy Management
- Azure Virtual Machines

---

## Skills Demonstrated

- Active Directory Administration
- Windows Server Administration
- Azure Infrastructure
- Virtual Networking
- PowerShell Automation
- Identity and Access Management
- DNS Configuration
- Group Policy Management
- Remote Desktop Services
- Virtual Machine Deployment
- Domain Administration

---
