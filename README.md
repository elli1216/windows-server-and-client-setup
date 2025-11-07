# Windows Server 2012 R2 and Windows 10 Client – Full Configuration Guide

This documentation provides a step-by-step guide to setting up a <b>Windows Server 2012 R2</b> domain environment with a <b>Windows 10 client</b>, including:

- Folder Redirection  
- Backup and Restore  
- Remote Desktop Configuration  
- Wallpaper Policy Deployment  
- Google Chrome Enterprise Deployment  

---

## 1. Environment Setup

### 1.1. Virtual Machine Requirements
| Component | Specification |
|------------|---------------|
| Server OS | <b>Windows Server 2012 R2 (with GUI)</b> |
| Client OS | <b>Windows 10</b> |
| Virtualization | <b>Oracle VirtualBox or VMware</b> |
| Network | <b>Internal Network</b> (static IPs for server, DHCP or static for client) |

---

## 2. Windows Server 2012 R2 Installation

```console
1. Boot the VM with the Windows Server 2012 R2 ISO.
2. Select language and click Install Now.
3. Enter the product key and accept the license agreement.
4. Choose Server with a GUI.
5. Select Custom: Install Windows only (Advanced).
6. Create and format partitions as instructed.
7. Set Administrator password: Css@1234.
8. After installation, log in using Administrator and the password above.
````

---

## 3. Initial Server Configuration

```console
1. Open Network and Sharing Center → Ethernet → Properties.
2. Select Internet Protocol Version 4 (TCP/IPv4) → Properties.
3. Assign a static IP (e.g., 192.168.10.1) and subnet mask 255.255.255.0.
4. Set the DNS server to itself (192.168.10.1).
5. Turn off Windows Firewall for Domain, Private, and Public networks.
6. Rename computer to SERVER, then restart.
```

---

## 4. Add Roles and Features

```console
1. Open Server Manager → Add Roles and Features.
2. Select the following:
   - Active Directory Domain Services
   - DHCP Server
   - DNS Server
   - File Server Resource Manager
3. Allow automatic restarts when prompted.
4. Wait for installation to complete.
```

---

## 5. Promote Server to Domain Controller

```console
1. In Server Manager, click the flag (notification icon).
2. Select Promote this server to a domain controller.
3. Choose Add a new forest.
4. Enter Root Domain Name: e.g. JDVP.org.
5. Set Directory Services Restore Mode (DSRM) password: Css@1234.
6. Continue through default settings → click Install.
7. Wait for automatic restart.
```

---

## 6. Configure DHCP and DNS

```console
1. After reboot, open Server Manager.
2. Click the notification flag → Complete DHCP Configuration.
3. Click Next → Commit → Close.
4. Verify DNS by ensuring server IP is listed as DNS in IPv4 properties.
```

---

## 7. File Server and Folder Redirection Setup

### 7.1. Create User Accounts

```console
1. Open Tools → Active Directory Users and Computers.
2. Expand JDVP.org.
3. Right-click domain → New → User.
   - Example user: Juan
   - Password: user-specific
   - Uncheck "User must change password at next logon"
   - Check "Password never expires"
4. Repeat to create a second user (e.g., Michael).
```

---

### 7.2. Create and Share User Folder

```console
1. Open Drive D: → Create folder User.
2. Right-click folder → Properties → Sharing → Advanced Sharing.
3. Check Share this folder.
4. Click Permissions → Remove Everyone → Add Domain Users.
5. Grant Full Control, click Apply, then OK.
6. Under Caching, select:
   “No files or programs from the shared folder are available offline.”
```

---

### 7.3. Configure Folder Redirection via Group Policy

```console
1. Open Group Policy Management (Tools → GPMC).
2. Right-click domain JDVP.org → Create a GPO in this domain.
   Name it: CSS Department or any name you want.
3. Right-click CSS Department → Edit.
4. Navigate to:
   User Configuration → Policies → Windows Settings → Folder Redirection.
5. Right-click Desktop → Properties.
   - Setting: Basic – Redirect everyone’s folder to the same location.
   - Target folder location: \\SERVER\User
   - Uncheck “Grant the user exclusive rights”.
   - Check “Also apply redirection policy to Windows 10”.
6. Repeat for Documents.
7. Click Apply → OK.
```

---

## 8. Windows 10 Client Setup

```console
1. Install Windows 10 normally.
2. Open Network & Internet Settings → Network and Sharing Center.
3. Set IP configuration:
   - Select “Obtain an IP address automatically”.
   - Set DNS to server IP (e.g., 192.168.10.1).
4. Rename computer to CLIENT.
5. Join domain:
   - Right-click This PC → Properties → Change Settings → Change.
   - Choose Domain: JDVP.org.
   - Enter Administrator credentials (Administrator / Css@1234).
6. Restart the system.
```

---

### Verify Folder Redirection

```console
1. Log in as a domain user (e.g., Juan).
2. Create a folder and files on Desktop.
3. On the server, open D:\User\Juan\Desktop.
4. Verify that files are visible.
```

---

## 9. Configure Remote Desktop

### On Server

```console
1. Search “Allow remote access to your computer”.
2. Enable “Allow remote connections to this computer”.
3. Click Select Users → Add.
4. Enter Domain Users → Check Names → OK.
```

### On Client

```console
1. Perform the same configuration on the Windows 10 machine.
2. Allow remote connections and add Domain Users.
3. Use Remote Desktop Connection to connect:
   - From client: connect to server IP.
   - From server: connect to client IP.
```

---

## 10. Backup and Restore Configuration

### On Server

```console
1. Open Drive D: → Create folder Backup Folder.
2. Right-click → Properties → Advanced Sharing → Share this folder.
3. Remove Everyone → Add Domain Users → Full Control.
4. Under Caching, select “No files or programs available offline”.
```

---

### On Client

```console
1. Create a folder in Drive D: named Backup Files.
2. Add sample files inside.
3. Open Control Panel → Backup and Restore (Windows 7).
4. Select “Set up backup”.
5. Choose “Save on network”.
6. Enter network path: \\SERVER\Backup Folder.
7. Enter Administrator credentials → Next.
8. Choose “Let me choose” → select Drive D.
9. Run backup and wait for completion.
10. To restore, open Backup and Restore → “Restore my files”.
```

---

## 11. Wallpaper Policy Deployment

```console
1. On Server, prepare a shared folder with a wallpaper image (e.g., \\SERVER\Shared\wallpaper.jpg).
2. Open Group Policy Management → Edit CSS Department. (or your name of your group)
3. Navigate to:
   User Configuration → Administrative Templates → Desktop → Desktop.
4. Double-click “Desktop Wallpaper”.
   - Enable it.
   - Set Wallpaper Name: \\SERVER\Shared\wallpaper.jpg
   - Wallpaper Style: Fill.
5. Apply and update policy on client:
   gpupdate /force
```

---

## 12. Google Chrome Enterprise Deployment Policy

```console
1. On your host desktop machine, download the latest Google Chrome Enterprise package:
   https://chromeenterprise.google/browser/download/
2. Extract the downloaded file. Inside, locate the GoogleChromeStandaloneEnterprise64.msi installer.
3. Copy or drag the installer file into the Windows Server 2012 R2 virtual machine.
4. On the server, open or create a shared software distribution folder:
   Example: D:\Software\ChromeInstaller\
   Place the Chrome .msi file there.
5. Open Group Policy Management → Edit CSS Department GPO (or your name of group).
6. Navigate to:
   Computer Configuration → Policies → Software Settings → Software Installation.
7. Right-click Software Installation → New → Package.
8. In the dialog, enter the full UNC path to the shared MSI file:
   \\SERVER\Software\ChromeInstaller\GoogleChromeStandaloneEnterprise64.msi
9. Select “Assigned” as the deployment method and click OK.
10. Close the Group Policy Editor and run on the client:
    gpupdate /force
11. Restart the Windows 10 client. Chrome will install automatically at startup.
```

---

## 13. Verification and Maintenance

| Task                      | Verification                                                    |
| ------------------------- | --------------------------------------------------------------- |
| <b>Folder Redirection</b> | Files created on client appear in server shared directory.      |
| <b>Remote Desktop</b>     | Users can access each machine via RDP using domain credentials. |
| <b>Backup</b>             | Backup completes successfully and restoration restores files.   |
| <b>Wallpaper</b>          | All users display the configured wallpaper.                     |
| <b>Chrome Deployment</b>  | Chrome installs automatically via Group Policy.                 |

---

## 14. Completion

You now have a fully functional <b>Windows Server 2012 R2</b> domain environment with a <b>Windows 10 client</b> configured for:

* Centralized storage via Folder Redirection
* Network Backup and Restore
* Remote Desktop access
* Wallpaper Policy deployment
* Google Chrome Enterprise deployment

---

<b>Author:</b> Darl Ellison Floresca <b>
Test Environment:</b> Windows Server 2012 R2 & Windows 10
