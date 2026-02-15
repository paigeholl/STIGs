# Scan Templates w/ STIGs

STIGs (Security Technical Implementation Guides) are compliance checklists that define secure configuration standards for systems. They check whether specific security settings are properly configured. Unlike general vulnerabilities (which are individual software flaws or weaknesses), STIGs focus on hardening configurations to meet DoD security baselines. In this lab, we'll intentionally create security misconfigurations on a Windows VM, then use Tenable Nessus to scan it against DISA STIG standards to see how the scanner detects and flags non-compliant settings.

## Create Vulnerable VM

**WARNING: Never create these vulnerabilities in production environments.**

Create a Windows 11 Pro Virtual Machine if needed.  [Click here to see steps.](https://github.com/paigeholl/VulnVM/blob/main/README.md)

**Disable Windows Firewall**

- Log into the VM
- Disable Windows Firewall if not already done.
- See how to do that [here](https://github.com/paigeholl/UnvA-Scans/blob/main/README.md#disable-windows-firewall)

**Create Administrator Account**

- Search "Computer Management" in taskbar
- Open the app
- Select "Local Users and Groups" > "Users"
- Right-click in window
- Select "New User"

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_cqvJJQZjA3P91Tp2Lh.png width=400px>

- Username: Administrator
- Uncheck "User must change password at next logon"
- Check "Password never expires"
- Select "Create" > "Close"

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_ohBNROUcCxF5jysspI.png width=400px>

**Add Administrator to Administrators Group**

- Select "Groups"
- Double-click "Administrators"
- Select "Add…"
- Type "Administrator"
- Select "Check Names" > "OK" > "Apply" > "OK"

**Enable Guest Account with Blank Password**

- Double-click "Guest" user in Users folder
- Uncheck "Account is disabled"
- Select "Member Of" tab
- Select "Add…"
- Type "Administrators"
- Select "Check Names" > "OK" > "Apply" > "OK"
- Right-click "Guest" user
- Select "Set Password…" > "Proceed"
- Leave password fields blank
- Select "OK"

## Create Scan Template

- Log into Tenable
- Navigate to Scans > Create a Scan Template > Advanced Network Scan

**Basic Tab**

- Name: Name it anything, for example “Win-11-STIG-test”
- Check "Start the Remote Registry service during the scan"
- Check "Enable administrative shares during the scan"
- Check "Start the Server service during the scan"

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_G6gZigTjtseGCIHaZ5.jpg width=400px>

**Discovery Tab**

- Check "Ping the remote host"
- Check "Use fast network discovery"
- Network Port Scanners: Check "TCP"

**Assessment Tab**

- Check "Perform thorough tests"
- Uncheck "Only use credentials provided by the user"

**Credentials**

- Select "Add Credentials" > "Host" > "Windows"
- Enter Administrator username and password

**Compliance Tab**

- Select "Add Compliance Audits"
- Search "STIG"
- Under "Windows" select "DISA Microsoft Windows Server 2019 STIG"
- Select "Save"

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_finF5mw2ORCaknRIfz.jpg width=400px>

**Plugins Tab**

- Disable all plugins
- Enable only:
  - General
  - Policy Compliance
  - Settings
  - Windows
  - Windows: Microsoft Bulletins
  - Windows: User management
- Save the Scan Template

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_sgpXNyA4yYoQptGx1m.jpg width=400px>

## Run Scan with Template

- Navigate to Scans > Create Scan > User Defined

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_5HcGki4iyjbl4YHAMQ.jpg width=400px>

- Select the scan template created above
- Name: YourName - Win Server 2019 STIG
- Scanner Type: Internal Scanner
- Credentials: We don't need to touch this since we set these up in the previous steps but enter the credentials if you had to change them
- Launch the scan

## Observe the Results

Observe Vulns by Plugin

- During my scan, I found several medium‑severity issues in the Misc. category related to the libcurl library (good thing we checked Misc. to get that exposure😉). One of them affects how libcurl handles LDAPS (LDAP over TLS) requests when an application uses multiple threads.
  - The problem: changing TLS settings in one thread can unintentionally change them for all threads, which can weaken security across every connection.
  - Think of it like a group of kids sharing the same light switch but in different rooms. If one kid flips it, every room’s lights change, even the ones that weren’t supposed to.
  - The fix is simple: upgrade libcurl to version 8.18.0 or later.

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_sEIxurQrpC5yvScdhK.png width=400px>

**Audits Tab**

- Review STIG compliance results
- Passed = configurations meet STIG requirements
- Failed/Warning = configurations require remediation to meet STIG compliance
- Key failures identified:
  - "The built-in guest/Administrator account must be disabled"
  - "Administrator accounts must not be enumerated during elevation"
- STIG successfully detected the intentional misconfigurations we put in place earlier
- Some stand outs are 'The built-in guest/Administrator account must be disabled.' and 'Administrator accounts must not be enumerated during elevation'
    

<img src=https://ik.imagekit.io/typeai/tr:w-1200,c-at_max/img_OTz5zO2GpGAYxWNDxY.jpg width=400px>

•	Observe Remediations
  
•	NOTE: The VM became noticeably sluggish during the scan, which is why resource heavy scans are typically scheduled during off hours to avoid disrupting users.
  
•	To get more practice: Go back in to the VM and make the adjustments to the guest and admin accounts we created/modified and run the scan again to see the Audit results 'passed'
