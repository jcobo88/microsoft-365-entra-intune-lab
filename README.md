# Microsoft 365 / Entra ID / Intune Administration Lab

## Overview

I built this lab to practice the type of Microsoft cloud administration and endpoint support work commonly performed by help desk, desktop support, and junior systems administrators.

The environment uses a Microsoft 365 Business Premium tenant, Microsoft Entra ID, Microsoft Intune, and a Windows 11 Pro virtual machine. The project covers identity administration, device enrollment, configuration and compliance policies, application deployment, Conditional Access, Windows Update management, BitLocker, Windows LAPS, troubleshooting, and remote device administration.

The lab was built as a cloud-first Microsoft environment rather than as an extension of my separate Active Directory lab.

---

## Environment

| Component | Configuration |
|---|---|
| Tenant | Cobo Technologies |
| Licensing | Microsoft 365 Business Premium |
| Identity | Microsoft Entra ID |
| Endpoint Management | Microsoft Intune |
| Pilot Endpoint | WIN11-INTUNE01 |
| Endpoint OS | Windows 11 Pro |
| Virtualization | Oracle VirtualBox |
| Pilot User | Alex Rivera |
| Pilot Device Group | DG-Windows-Pilot |
| Conditional Access Pilot Group | SG-CA-MFA-Pilot |

---

## Architecture

```text
Microsoft 365 Business Premium
            |
            v
     Microsoft Entra ID
     |              |
     |              +--> Cloud users / groups / MFA
     |
     +--> Conditional Access
            |
            v
       Microsoft Intune
     |       |        |
     |       |        +--> Application deployment
     |       +----------> Compliance / security
     +------------------> Windows configuration
            |
            v
      WIN11-INTUNE01
      Windows 11 Pro
```

---

## Identity and Licensing

I created a small fictional organization, **Cobo Technologies**, to simulate common Microsoft 365 administration tasks.

The identity environment included:

- Cloud-only Microsoft Entra users across multiple departments.
- Assigned security groups for IT, HR, and Sales.
- Bulk user creation using a CSV template.
- Microsoft 365 Business Premium license assignment.
- A dedicated Conditional Access pilot group.
- A separate user account for help-desk troubleshooting scenarios.

This allowed me to practice the relationship between identity, group membership, licensing, authentication, and access.

---

## Microsoft Entra Join and Intune Enrollment

I created a Windows 11 Pro virtual machine named:

```text
WIN11-INTUNE01
```

The device was joined directly to Microsoft Entra ID and automatically enrolled into Microsoft Intune.

I verified the device registration state with:

```powershell
dsregcmd /status
```

The device reported:

```text
AzureAdJoined : YES
DomainJoined  : NO
DeviceAuthStatus : SUCCESS
```

I then verified the device in Microsoft Intune as:

```text
Managed by: Intune
Ownership: Corporate
Compliance: Compliant
Primary user: Alex Rivera
```

---

## Device Configuration

I created a Settings Catalog policy for Microsoft Edge and deployed it to a pilot device group.

The policy configured:

- Office.com as the Edge homepage.
- The Home button on the toolbar.
- Device-level mandatory policy enforcement.

I first verified that no policies were present in:

```text
edge://policy
```

After syncing the device, the policies appeared with:

```text
Source: Platform
Applies To: Device
Level: Mandatory
Status: OK
```

This demonstrated both configuration deployment and endpoint-side verification.

---

## Compliance Policy and Troubleshooting

I created a Windows compliance policy requiring Microsoft Defender Firewall.

The test sequence was:

```text
Device initially compliant
        |
        v
Disable active firewall profile
        |
        v
Intune reports Not compliant
        |
        v
Firewall identified as failed setting
        |
        v
Restore firewall
        |
        v
Device returns to Compliant
```

This demonstrated the difference between a configuration policy and a compliance policy.

A configuration policy changes settings.

A compliance policy evaluates whether the endpoint meets organizational requirements.

---

## Microsoft Store Application Deployment

I deployed **Company Portal** as a required Microsoft Store application.

The application was assigned to:

```text
DG-Windows-Pilot
```

Intune later reported:

```text
Installation status: Installed
```

This demonstrated required application deployment through Microsoft Intune.

---

## Win32 Application Deployment

I packaged and deployed 7-Zip as a Win32 application.

The original installer was:

```text
7z2603-x64.msi
```

I packaged it using the Microsoft Win32 Content Prep Tool:

```text
7z2603-x64.msi
        |
        v
IntuneWinAppUtil.exe
        |
        v
7z2603-x64.intunewin
```

The silent install command was:

```cmd
msiexec /i "7z2603-x64.msi" /qn /norestart
```

The silent uninstall command was:

```cmd
msiexec /x "{23170F69-40C1-2702-2603-000001000000}" /qn /norestart
```

The deployment included:

- x64 architecture requirements.
- Windows operating system requirements.
- System installation context.
- MSI product-code detection.
- Silent install and uninstall commands.
- Required assignment to the pilot device group.

Intune later reported:

```text
WIN11-INTUNE01
Status: Installed
```

This demonstrated the complete Win32 application lifecycle from packaging through detection and deployment.

---

## Conditional Access and MFA

I created a Conditional Access pilot policy:

```text
CA-Pilot-Require-MFA
```

The policy targeted:

```text
SG-CA-MFA-Pilot
```

and required Microsoft's built-in multifactor authentication strength.

The policy was initially configured as:

```text
Report-only
```

before enforcement.

I validated the targeting using the Conditional Access **What If** tool and later confirmed the policy against a real OfficeHome sign-in.

The sign-in logs showed:

```text
Report-only: Success
```

This demonstrated how Conditional Access can be tested before enforcement to reduce the risk of accidentally blocking users.

---

## Device Compliance and Conditional Access

I created another Conditional Access policy:

```text
CA-Pilot-Require-Compliant-Windows-Device
```

The policy required Windows devices to be marked compliant by Intune.

I tested the policy under three conditions.

### Healthy Device

```text
Intune:
Compliant

Conditional Access:
Report-only: Success
```

### Firewall Disabled

I intentionally disabled the active Microsoft Defender Firewall profile.

Intune detected:

```text
Firewall:
Not compliant
```

A new OfficeHome sign-in then produced:

```text
CA-Pilot-Require-Compliant-Windows-Device
Report-only: Failure
```

The Entra sign-in record showed:

```text
Managed: Yes
Compliant: No
Join Type: Azure AD joined
```

### Firewall Restored

After turning Microsoft Defender Firewall back on and syncing the endpoint:

```text
Intune:
Compliant
```

The next Conditional Access evaluation returned:

```text
Report-only: Success
```

This demonstrated the full relationship between:

```text
Endpoint security state
        |
        v
Intune compliance
        |
        v
Microsoft Entra Conditional Access
        |
        v
Cloud resource access decision
```

---

## Windows Update Management

I created an Intune Update Ring:

```text
WIN-Update-Ring-Pilot
```

The policy included:

```text
Microsoft product updates: Allow
Windows drivers: Allow
Quality update deferral: 0 days
Feature update deferral: 0 days

Active hours:
8:00 AM - 5:00 PM

Quality update deadline:
2 days

Feature update deadline:
7 days

Grace period:
1 day
```

The policy was assigned to:

```text
DG-Windows-Pilot
```

I verified the settings directly on the endpoint under:

```text
Windows Update
→ Advanced options
→ Configured update policies
```

Windows displayed the settings as:

```text
Type: Mobile Device Management
```

This confirmed the Intune Windows Update policy reached the endpoint successfully.

---

## BitLocker Management

Before creating the Intune BitLocker policy, I verified the VM's security prerequisites using PowerShell.

```powershell
Get-Tpm | Select-Object TpmPresent,TpmReady,TpmEnabled,TpmActivated
```

The TPM reported:

```text
TpmPresent   : True
TpmReady     : True
TpmEnabled   : True
TpmActivated : True
```

I also verified:

```powershell
Confirm-SecureBootUEFI
```

```text
True
```

and:

```powershell
Get-ComputerInfo | Select-Object BiosFirmwareType
```

```text
BiosFirmwareType : Uefi
```

Windows Recovery Environment was also enabled.

BitLocker status showed:

```text
VolumeStatus         : FullyEncrypted
ProtectionStatus     : On
EncryptionPercentage : 100
EncryptionMethod     : XtsAes128
```

I created:

```text
WIN-BitLocker-Pilot
```

The policy configured:

- Required device encryption.
- TPM-based startup protection.
- No startup PIN or USB startup key.
- BitLocker recovery-password management.
- Recovery-password rotation.
- Central recovery-information storage.

Intune later reported:

```text
Succeeded: 1
Errors: 0
Conflicts: 0
```

I also verified that a BitLocker recovery-key record existed for the operating system drive without exposing the actual recovery password.

---

## Windows LAPS

I enabled Microsoft Entra Windows LAPS and created:

```text
WIN-LAPS-Pilot
```

Windows Local Administrator Password Solution automatically created and managed:

```text
Cobo-LAPSAdmin
```

The LAPS policy configured:

```text
Backup directory:
Microsoft Entra ID only

Password age:
30 days

Password length:
20 characters

Password complexity:
Uppercase + lowercase + numbers + special characters

Post-authentication reset delay:
8 hours

Automatic account management:
Enabled
```

I verified the account locally with:

```powershell
Get-LocalUser | Select-Object Name,Enabled,Description
```

Windows reported:

```text
Cobo-LAPSAdmin
Enabled: True

This account is currently being automatically managed
by your corporate administrator.
```

In Intune I verified that the LAPS password record existed and that password rotation timestamps were present without exposing the actual password.

---

## Help Desk Identity Troubleshooting

I created a separate employee identity:

```text
Elena Marquez
Procurement Coordinator
Operations
```

The account was assigned Microsoft 365 Business Premium and configured with MFA.

I first verified a healthy OfficeHome sign-in.

Then I simulated a help-desk incident by administratively blocking the user's sign-in.

The user received:

```text
Your account has been locked.
Contact your support person to unlock it, then try again.
```

Instead of assuming the password was incorrect, I investigated the Microsoft Entra sign-in logs.

The logs showed:

```text
Status:
Failure

Sign-in error code:
50057

Failure reason:
The user account is disabled.
```

I then restored the user's ability to sign in.

A new OfficeHome sign-in showed:

```text
Status: Success
```

The troubleshooting workflow was:

```text
Healthy user
      |
      v
Block sign-in
      |
      v
User sees account locked
      |
      v
Check Entra sign-in logs
      |
      v
Error 50057
User account disabled
      |
      v
Restore sign-in
      |
      v
Verify successful authentication
```

---

## Remote Device Administration

I issued a remote restart command from Microsoft Intune against:

```text
WIN11-INTUNE01
```

Intune reported:

```text
Restart initiated.
Restart will occur when the device is notified.
```

The endpoint then displayed:

```text
You're about to be signed out

Your device administrator has scheduled a reboot
```

This demonstrated direct remote administration of an Intune-managed endpoint.

---

## Troubleshooting Highlights

| Scenario | Diagnosis | Resolution |
|---|---|---|
| Firewall compliance failure | Intune reported Firewall as Not compliant | Restored active Defender Firewall profile |
| Conditional Access device failure | Entra showed managed device as noncompliant | Restored endpoint compliance |
| Missing device identity | Sign-in had no Device ID and showed Managed: No | Retested using managed Edge profile |
| Blocked Microsoft 365 user | Entra error 50057 identified disabled account | Restored sign-in |
| Win32 application delay | Application installed before portal status updated | Verified endpoint and waited for reporting |
| LAPS account creation | Managed account was not initially visible | Synced device and verified automatic creation |

---

## Key Lessons

This project reinforced several important Microsoft administration concepts:

- Microsoft Entra join and Intune enrollment are related but separate.
- Creating an Entra user does not automatically assign Microsoft 365 services.
- Group membership and licensing serve different purposes.
- Configuration policies change settings while compliance policies evaluate state.
- Required application assignments force deployment.
- Win32 requirement rules determine application applicability.
- Win32 detection rules determine whether an application is installed.
- Conditional Access should be tested before enforcement.
- Report-only mode allows administrators to evaluate access policies safely.
- Device-based Conditional Access depends on valid device identity being present in the sign-in.
- Intune reporting can lag behind the actual endpoint state.
- BitLocker recovery passwords and LAPS credentials should be centrally managed but never exposed publicly.
- Sign-in logs provide far more useful troubleshooting information than generic user-facing error messages.

---

## Repository Structure

```text
microsoft-365-entra-intune-lab/
├── README.md
├── SCREENSHOTS.md
├── .gitignore
└── screenshots/
    ├── 01-microsoft-365-tenant-created.png
    ├── 02-entra-id-first-user-created.png
    ├── ...
    └── 58-intune-remote-restart-received.png
```

---

## Security and Repository Hygiene

This repository intentionally does not contain:

- User passwords.
- Temporary passwords.
- MFA QR codes.
- Authenticator secrets.
- MFA verification codes.
- BitLocker recovery passwords.
- LAPS-managed local administrator passwords.
- Bulk provisioning CSV files containing passwords.
- Third-party installers.
- `.intunewin` application packages.
- Unnecessary personal contact information.

Screenshots are used to provide configuration and troubleshooting evidence while keeping credentials and recovery secrets protected.

---

## Skills Demonstrated

- Microsoft 365 Administration
- Microsoft Entra ID
- Microsoft Intune
- Windows 11 Administration
- Identity and Access Management
- User and Group Administration
- Microsoft 365 Licensing
- Device Enrollment
- Endpoint Configuration
- Compliance Policies
- Conditional Access
- Multi-Factor Authentication
- Microsoft Store App Deployment
- Win32 Application Packaging
- MSI Deployment
- Windows Update Management
- BitLocker
- Windows LAPS
- Endpoint Security
- Sign-In Log Analysis
- Help Desk Troubleshooting
- Remote Endpoint Administration
- PowerShell

---

## Screenshot Evidence

The project includes a full screenshot evidence trail covering the lab from tenant creation through remote endpoint administration.

See:

[SCREENSHOTS.md](SCREENSHOTS.md)
