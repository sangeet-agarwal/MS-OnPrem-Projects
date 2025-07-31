# A Comprehensive Guide to Deploying a Zero-Cost, Two-Tier SharePoint Server Subscription Edition Trial Farm on a Local Workstation

## Introduction

This document provides an exhaustive, expert-level guide for constructing a temporary, two-server SharePoint Server Subscription Edition farm. The entire process is designed to be completed on a single high-performance host machine at zero software cost, leveraging official evaluation editions of all required Microsoft products. This guide is intended for technically proficient individuals, such as IT professionals, systems engineers, and developers, who seek to establish a functional trial or development environment for SharePoint Server Subscription Edition.

The target architecture consists of a host personal computer running a Type-1 hypervisor, which will contain two distinct virtual machines (VMs). The first VM will be configured as an Active Directory Domain Controller (DC), providing the essential identity and authentication services for the farm. The second VM will be a consolidated application and database server, co-locating the SharePoint Server Subscription Edition and Microsoft SQL Server 2022 roles. This two-tier topology represents a common and realistic small-scale deployment model.

It is assumed that the user possesses a foundational understanding of Windows Server administration, basic IP networking principles, and general virtualization concepts. The instructions presume the availability of a host computer running a 64-bit edition of Windows 11 Pro or Enterprise, equipped with an AMD Ryzen 7 9800X3D processor and 64 GB of RAM, or a system with comparable performance characteristics. The report will guide the user through every necessary step, from host system preparation and software acquisition to the point where the environment is fully configured and ready for the final SharePoint Products Configuration Wizard.

---

## Section 1: Foundational Architecture and Host System Preparation

The successful deployment of a virtualized SharePoint farm begins with the correct preparation of the host system and a strategic selection of the underlying virtualization technology. This section details the process of configuring the host machine, choosing the optimal hypervisor, and acquiring the necessary software installation media.

### 1.1 Strategic Analysis of Virtualization Platforms

The selection of a hypervisor is a critical architectural decision. For the purpose of creating a local, Windows-centric server lab, Microsoft Hyper-V is the superior choice.

Hyper-V is a Type-1, or "bare-metal," hypervisor. When the Hyper-V role is enabled on Windows 11 Pro, the virtualization stack loads first, and the host Windows 11 operating system itself runs in a special parent partition on top of the hypervisor. This architecture provides more direct and efficient access to hardware for guest virtual machines, which can lead to better performance for I/O-intensive server workloads like SQL Server and SharePoint.

Furthermore, Hyper-V's native integration into the Windows operating system provides a more cohesive management experience when working exclusively with Microsoft server products. A critical consideration when working with hypervisors is that they require exclusive access to the processor's hardware virtualization extensions (Intel VT-x or AMD-V). Consequently, it is not recommended to have both Hyper-V and other virtualization applications like VMware Workstation or VirtualBox installed and active simultaneously.

### 1.2 Host System Configuration: Enabling the Hyper-V Role

Before enabling the Hyper-V role, it is essential to verify that hardware-assisted virtualization is enabled in the host system's UEFI/BIOS firmware. This feature is often labeled as "Intel Virtualization Technology (VT-x)," "AMD-V," or "SVM Mode" and is a mandatory prerequisite.

The most efficient method for enabling the Hyper-V role on a Windows 11 Pro system is via Windows PowerShell. Launch Windows PowerShell with administrative privileges and execute the following command:

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

The system will install the necessary components and prompt for a restart to complete the process. This restart is mandatory.

### 1.3 Acquiring Software Assets

To proceed with the farm construction at zero cost, download the official evaluation or trial versions of all required software components directly from Microsoft. These versions are fully functional for a 180-day period.

* **Windows Server 2022 (180-day Evaluation):**
    * Download Link: `https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022`
* **SQL Server 2022 (180-day Evaluation):**
    * Download Link: `https://www.microsoft.com/en-us/evalcenter/download-sql-server-2022`
* **SharePoint Server Subscription Edition (180-day Trial):**
    * Download Link: `https://www.microsoft.com/en-us/download/details.aspx?id=103378`
    * Trial Product Key (Enterprise): `VW2FM-FN9FT-H22J4-WV9GT-H8VKF`

Store all downloaded ISO files in a known location on the host machine.

---

## Section 2: Engineering the Virtual Infrastructure

This phase involves architecting and provisioning the virtual machines that will form the SharePoint farm.

### 2.1 Virtual Machine Resource Allocation Strategy

A critical configuration parameter for any SharePoint virtualization is the memory allocation method. Microsoft explicitly states that using dynamic memory features is **not supported** for servers running SharePoint Server. All virtual machines hosting a SharePoint role must be configured with a fixed, static amount of memory.

**Table 1: Virtual Machine Resource Allocation Plan**

| VM Role                      | vCPUs   | Static RAM | VHDX (C: Drive) | VHDX (Data Drive) | Notes                                        |
| :--------------------------- | :------ | :--------- | :-------------- | :---------------- | :------------------------------------------- |
| **DC01** (Domain Controller) | 2 Cores | 4 GB       | 80 GB           | N/A               | Gen 2, vTPM Enabled.                         |
| **SPSQL01** (SP + SQL)       | 8 Cores | 24 GB      | 80 GB           | 150 GB            | Gen 2, vTPM Enabled, **Static Memory Only**. |

### 2.2 Configuring the Virtual Network Fabric

To ensure a stable and isolated environment, create an **Internal Virtual Switch** within Hyper-V Manager.

1.  Open **Hyper-V Manager**.
2.  In the **Actions** pane, select **Virtual Switch Manager...**.
3.  Select **Internal** and click **Create Virtual Switch**.
4.  Name the new switch (e.g., `SP-Internal-Net`) and click **OK**.

### 2.3 Provisioning the Virtual Machines

Create two **Generation 2** VMs using the "New Virtual Machine" wizard in Hyper-V Manager, applying the resources from Table 1.

**Key Steps for VM Creation:**

1.  **Specify Generation:** Select **Generation 2**.
2.  **Assign Memory:** Enter the static RAM value. For `SPSQL01`, **uncheck** the box "Use Dynamic Memory for this virtual machine".
3.  **Configure Networking:** Connect the VM to the `SP-Internal-Net` switch.
4.  **Installation Options:** Select "Install an operating system from a bootable image file" and choose the Windows Server 2022 ISO.

**Post-Creation Security Configuration:**

For each VM, go to its **Settings > Security** and ensure that **Enable Secure Boot** and **Enable Trusted Platform Module** are both checked.

---

## Section 3: Constructing the Identity Foundation: The Domain Controller (DC01)

A properly configured Active Directory domain is a prerequisite for any SharePoint Server farm.

### 3.1 Windows Server 2022 OS Installation and Configuration

1.  Install **Windows Server 2022 (Desktop Experience)** on the `DC01` VM.
2.  Rename the server to `DC01` and restart.
3.  Assign a static IP address:
    * IP Address: `192.168.100.10`
    * Subnet Mask: `255.255.255.0`
    * Preferred DNS Server: `127.0.0.1` (points to itself)

### 3.2 Installation and Promotion of Active Directory

1.  In Server Manager, use the **Add Roles and Features** wizard to install **Active Directory Domain Services**. Accept the prompt to add the **DNS Server** role as well.
2.  After installation, click the notification flag in Server Manager and select **Promote this server to a domain controller**.
3.  Select **Add a new forest** and provide a root domain name (e.g., `sptrial.local`).
4.  Follow the wizard, setting a DSRM password and accepting defaults. The server will restart.

### 3.3 Provisioning SharePoint Service Accounts

In **Active Directory Users and Computers** on `DC01`, create a new Organizational Unit (OU) named `SharePoint Service Accounts`. Inside this OU, create the following user accounts with strong passwords set to never expire:

* `sql_service`
* `sp_farm`
* `sp_serviceapps`

---

## Section 4: Assembling the Application Tier: The SharePoint & SQL Server (SPSQL01)

This section details the configuration of the second virtual machine, `SPSQL01`.

### 4.1 Server Deployment and Domain Integration

1.  Install **Windows Server 2022 (Desktop Experience)** on the `SPSQL01` VM.
2.  Rename the server to `SPSQL01`.
3.  Configure a static IP address. **This is critical.**
    * IP Address: `192.168.100.20`
    * Subnet Mask: `255.255.255.0`
    * Preferred DNS Server: `192.168.100.10` (points to the DC)
4.  Join the server to the `sptrial.local` domain and restart.

### 4.2 SQL Server 2022 Deployment and Hardening

**Installation:**

1.  Log into `SPSQL01` as a domain admin. Mount the SQL Server ISO and run setup.
2.  Install the **Database Engine Services** as a **Default instance**.
3.  On the Service Accounts tab, set both services to run as `sptrial\sql_service`.
4.  For Database Engine Configuration, choose **Mixed Mode** authentication. Add the current user (domain admin) and the `sp_farm` account as SQL administrators.
5.  Complete the installation, then install **SQL Server Management Studio (SSMS)** from the link in the installation center.

**Post-Install Configuration (Crucial):**

1.  Launch **SSMS** and connect to the server.
2.  In server **Properties > Advanced**, set **Max Degree of Parallelism** to `1`.
3.  In server **Properties > Memory**, set **Maximum server memory** to `8192` MB.
4.  In **Security > Logins**, give the `sp_farm` account the `dbcreator` and `securityadmin` server roles.

The following table summarizes these critical settings.

**Table 2: SQL Server Configuration for SharePoint Environments**

| Configuration Setting      | Recommended Value                   | Justification                                        |
| :------------------------- | :---------------------------------- | :--------------------------------------------------- |
| Service Account            | `sptrial\sql_service`               | Adheres to the principle of least privilege.         |
| Authentication Mode        | Mixed Mode                          | Provides flexibility.                                |
| SQL Administrators         | Domain Admin, `sptrial\sp_farm`     | Grants necessary access for management and setup.    |
| Max Degree of Parallelism  | `1`                                 | A strict SharePoint requirement to prevent query issues. |
| Maximum Server Memory      | `8192` MB                           | Prevents SQL from starving SharePoint of RAM.        |
| Farm Account Server Roles  | `dbcreator`, `securityadmin`        | Allows the setup wizard to create and manage databases. |

### 4.3 SharePoint Server Prerequisite Installation

The final step is to prepare the `SPSQL01` server.

1.  Mount the SharePoint Server Subscription Edition ISO file on the `SPSQL01` VM.
2.  From the root of the mounted drive, execute `prerequisiteinstaller.exe`.
3.  The tool will analyze, download, and install all required components. This may require one or more automatic server reboots.

---

## Conclusion: Farm Readiness and Path Forward

The virtual infrastructure is now fully configured. Before proceeding with the SharePoint installation, perform a final verification.

**Final Pre-flight Checklist:**

* **`DC01` VM Status:** Confirm `DC01` is running.
* **`SPSQL01` VM Status:** Confirm `SPSQL01` is running and joined to the domain.
* **Network Connectivity:** From `SPSQL01`, successfully `ping dc01.sptrial.local`.
* **Service Accounts:** Confirm the three service accounts exist in AD.
* **SQL Server Configuration:** Confirm MAXDOP, memory limits, and `sp_farm` permissions in SSMS.
* **Prerequisites:** Ensure the prerequisite installer has completed successfully on `SPSQL01`.

With all checklist items confirmed, the environment is fully prepared. Your next step is to log into `SPSQL01`, run `setup.exe` from the SharePoint ISO, and follow the SharePoint Products Configuration Wizard to create your new farm.
