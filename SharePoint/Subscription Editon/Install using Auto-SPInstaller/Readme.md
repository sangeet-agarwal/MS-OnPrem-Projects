# Guide to Automated SharePoint SE Installation with AutoSPInstaller

This guide details the process of performing a fully automated, configured installation of SharePoint Server Subscription Edition on a single, pre-existing server using the AutoSPInstaller tool. This powerful script suite replaces the manual process of running the prerequisite installer, setup executable, and the SharePoint Products Configuration Wizard.   

## Part 1: Pre-Automation Server and Account Preparation

Before running the automation script, several critical preparation steps must be completed. These ensure the script has the necessary permissions and resources to execute successfully.

### 1.1. Active Directory Service Accounts

A successful automated SharePoint installation depends on pre-configured service accounts that adhere to the principle of least privilege. The AutoSPInstaller script is designed to use specific accounts for different components.   

In Active Directory Users and Computers, create the following user accounts. For a lab environment, it is recommended to set their passwords to never expire to avoid service interruptions.

**Table 1: Required Service Accounts for AutoSPInstaller**

| Account Name     | Description/Purpose                                                                                      | Key Permissions Required                                                                                 |
|------------------|-----------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| `sp_install`      | The primary setup account used to run the AutoSPInstaller script.                                         | Must be a member of the local Administrators group on the SharePoint server. Requires `dbcreator` & `securityadmin` roles in SQL Server.   |
| `sp_farm`         | The SharePoint Farm Account. It runs the SharePoint Timer Service and acts as the identity for the Central Administration application pool. | A standard domain user. Permissions are granted to databases automatically during setup.   |
| `sp_services`     | A general-purpose account used as the identity for most service application pools (e.g., Managed Metadata, State Service).   | A standard domain user. |
| `sp_search`       | Runs the Search service application and acts as the default content access account for crawling content.   | A standard domain user. |
| `sp_userprofile`  | Used by the User Profile Synchronization service to import profiles from Active Directory.               | A standard domain user. Requires "Replicating Directory Changes" permission on the domain.   |

To grant the `sp_userprofile` account the required **"Replicating Directory Changes"** permission:

1. On your Domain Controller, open **Active Directory Users and Computers**.
2. Right-click the domain and select **Delegate Control...**.
3. Add the `sp_userprofile` account to the wizard.
4. Select **Create a custom task to delegate**.
5. Select **This folder, existing objects in this folder, and creation of new objects in this folder**.
6. In the permissions list, check the box for **Replicating Directory Changes** and complete the wizard.

### 1.2. SQL Server Configuration

A default SQL Server installation is not optimized for SharePoint. The following configurations must be applied using SQL Server Management Studio (SSMS) to ensure stability and performance. Log in to SSMS with an account that has **sysadmin** privileges.

#### Grant Setup Account Permissions

The `sp_install` account needs specific permissions on the SQL Server instance:

- In **Object Explorer**, expand **Security > Logins**.
- Add the `LAB\sp_install` account as a new login.
- In the login's properties, go to the **Server Roles** page and check the boxes for `dbcreator` and `securityadmin`.

#### Set Max Degree of Parallelism (MAXDOP)

This is a mandatory performance tuning setting for SharePoint workloads:

- In **Object Explorer**, right-click the server instance and select **Properties**.
- Navigate to the **Advanced** page.
- Set the value for **Max Degree of Parallelism** to `1`.

#### Configure Max Server Memory

This prevents SQL Server from consuming all available RAM:

- In the same **Server Properties** window, navigate to the **Memory** page.
- Set the **Maximum server memory (in MB)** to a value less than the total server RAM.  
  _Example_: For a server with 24 GB of RAM, a value of `20480` (20 GB) is a reasonable starting point.

---

## Part 2: The AutoSPInstaller Process

This section covers the core automated installation, from preparing the source files to executing the script.

### 2.1. Preparing the Installation Source

A successful automated installation begins with a well-structured and complete source directory. The best practice is to create an offline installer package that contains the SharePoint binaries and all required prerequisites.

#### Download AutoSPInstaller and AutoSPSourceBuilder:

- Download the latest version of **AutoSPInstaller** as a ZIP file from its official GitHub repository:  
  [github.com/brianlala/AutoSPInstaller](https://github.com/brianlala/AutoSPInstaller)

- Install **AutoSPSourceBuilder** from the PowerShell Gallery:

```powershell
Install-Module -Name AutoSPSourceBuilder -Force
```

#### Create the Folder Structure:

- On your SharePoint server, create a root installation folder, e.g., `C:\SPInstall`.
- Inside this folder, create a subfolder named `SP`.

#### Copy SharePoint Binaries and Run AutoSPSourceBuilder:

- Mount the **SharePoint Server Subscription Edition ISO**. Copy its contents to `C:\SPInstall\SP\SharePoint`.
- Launch an elevated PowerShell console and run:

```powershell
AutoSPSourceBuilder.ps1 -SourcePath "C:\SPInstall\SP\SharePoint" -Destination "C:\SPInstall\SP"
```

#### Copy AutoSPInstaller Scripts:

- From the extracted AutoSPInstaller ZIP file, copy the `Automation` folder into `C:\SPInstall\SP`.

---

### 2.2. Crafting the AutoSPInstallerInput.xml Configuration File

The `AutoSPInstallerInput.xml` file is the declarative blueprint for the entire SharePoint farm.

Generate it using the GUI at [autospinstaller.com](https://autospinstaller.com).

#### Generate the Base XML:

1. Navigate to autospinstaller.com and click **Build a Farm**.
2. **Install Tab**: Enable Auto-Logon, enter credentials for `sp_install`, enable Offline Install, and enter the product key.
3. **Farm Tab**: Enter a secure passphrase. Provide `sp_farm` credentials. Set the Database Server name. Configure the Central Admin port and assign `sp_farm` to the app pool.
4. **Servers Tab**: Add a single server with its hostname.
5. **MinRoles Tab**: Select the **Single-Server Farm** role.
6. **Service Applications Tabs**: Assign correct service accounts (from Part 1).
7. **Web Applications & Site Collections Tabs**: Define your web app and root site collection.

#### Download and Perform the Critical Manual Edit:

- Go to **Review & Download**, copy the XML, and save it as:  
  `C:\SPInstall\SP\Automation\AutoSPInstallerInput.xml`

**Important**: Manually change:

```xml
<Provision SPVersion="2019" ...>
```

to

```xml
<Provision SPVersion="SE" ...>
```

Save and close the file.

---

### 2.3. Executing the Automated Installation

1. Log in as the `sp_install` user.
2. Open an elevated PowerShell or Command Prompt.
3. Navigate to the Automation folder:

```cmd
cd C:\SPInstall\SP\Automation
```

4. Launch the installer:

```cmd
.\AutoSPInstallerLaunch.bat
```

The script will now run unattended, performing all necessary steps including installing prerequisites, binaries, creating the farm, and provisioning all configured services.

---

## Part 3: Post-Installation Verification

After the script completes, verify functionality.

### Access Central Administration:

- Open a browser and go to the Central Admin URL (e.g., `http://yourservername:2022`).

### Validate the Portal Site:

1. On your Domain Controller, open **DNS Manager**.
2. Create a new **A (Host)** record for your portal (e.g., `portal -> [SharePoint Server IP]`).
3. On the SharePoint server, open a browser and visit:  
   `http://portal.yourdomain.local`

---

## Sources and Related Content

- [AutoSPInstaller GitHub](https://github.com/brianlala/AutoSPInstaller)
- [AutoSPSourceBuilder](https://www.powershellgallery.com/packages/AutoSPSourceBuilder)
- [AutoSPInstaller GUI](https://autospinstaller.com)
