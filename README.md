# MS-OnPrem
This repository serves as a dedicated showcase for practical projects and solutions implemented using **Microsoft on-premises technologies**. It aims to demonstrate hands-on expertise in setting up, configuring, and managing traditional IT infrastructure components, often with an eye towards understanding foundational concepts that bridge to modern cloud solutions.

---

## About This Repository

While the IT landscape is rapidly shifting towards cloud-centric models, a strong understanding of on-premises environments remains fundamentally important. Many organizations operate complex hybrid infrastructures, and foundational knowledge of core services like Active Directory, SQL Server, SharePoint, and various virtualization technologies is invaluable.

This repository will contain detailed guides and configurations for projects focused on:

* **Microsoft SharePoint Server:** Installation, configuration, and administration scenarios.
* **Microsoft SQL Server:** Database setup, configuration for enterprise applications.
* **Active Directory Domain Services (AD DS):** Domain setup, user management, and group policies.
* **Virtualization Technologies:** Utilizing platforms like Hyper-V and VMware to create isolated lab and production-like environments.
* **Networking & System Administration:** Practical configurations for on-premises server roles and infrastructure.

Each project will aim to replicate real-world challenges or common deployment scenarios, providing step-by-step instructions, technical explanations, and insights gained during implementation.

---

## Projects

Here's a list of on-premises projects currently documented and available in this repository:

### 1. Single-Server SharePoint Subscription Edition Farm on Hyper-V

* **Problem/Business Scenario:** Many organizations, especially small to medium-sized businesses (SMBs) or internal development/testing teams, require a robust, self-managed collaboration platform like SharePoint. This project addresses the common need to establish a fully functional, domain-integrated SharePoint environment in an on-premises virtualized setting, such as on a powerful personal workstation. It's crucial for understanding the foundational components and interdependencies of a SharePoint farm.

* **Solution Overview:** This project outlines the comprehensive process of building a self-contained, single-server SharePoint Subscription Edition farm. The architecture involves setting up two virtual machines within **Hyper-V**: one dedicated **Domain Controller** (running Active Directory Domain Services) and another server hosting both **SQL Server** (as SharePoint's database backend) and **SharePoint Server Subscription Edition** itself. This setup mimics a common small-to-medium enterprise on-premises deployment, providing a complete lab environment for learning and experimentation.

* **Key Technologies Demonstrated:**
    * **Hyper-V:** Microsoft's native hypervisor for efficient virtual machine creation and management on a Windows host.
    * **Windows Server (e.g., Windows Server 2022):** The operating system for both the Domain Controller and the combined SQL/SharePoint server.
    * **Active Directory Domain Services (AD DS):** Establishing a new Active Directory domain, configuring DNS, and creating necessary service accounts for SharePoint.
    * **SQL Server (e.g., SQL Server 2019):** Installation and configuration of the database instance and specific settings required for SharePoint.
    * **SharePoint Server Subscription Edition:** Step-by-step installation and initial farm creation, including prerequisite installation.

* **What You'll Find:**
    A comprehensive, step-by-step guide (located in the `./SharePoint-SE-HyperV-Farm/` directory) that covers:
    * **Hyper-V Setup:** Instructions for enabling Hyper-V and creating virtual machines tailored for server workloads.
    * **Operating System Installation:** Windows Server installation and initial network configuration for domain integration.
    * **Active Directory Domain Services (AD DS) Configuration:** Promoting a server to a Domain Controller and setting up the domain.
    * **SQL Server Installation & Configuration:** Detailed steps for installing SQL Server and preparing it for SharePoint.
    * **SharePoint Server Subscription Edition Deployment:** Prerequisites installation, SharePoint installation, and farm configuration wizard steps.
    * **Post-Installation Tasks:** Essential configurations after the farm is up and running.
    * **Troubleshooting Tips:** Common issues and resolutions encountered during deployment.
    * **Detailed Explanations:** Insights into *why* each step is performed and its significance in an on-premises enterprise environment, including architectural diagrams and flow.

* **Access the Full Project Guide:** [Single-Server SharePoint Subscription Edition Farm on Hyper-V Guide](./SharePoint-SE-HyperV-Farm/README.md)

---

## How Projects Are Documented

Each project within this repository will typically follow a consistent structure to ensure clarity, reusability, and ease of understanding for anyone wishing to follow along or replicate the setup:

* **Dedicated Project Folder:** Each project will reside in its own named directory (e.g., `Project-Name-Short`).
* **`README.md` (within project folder):** This central file will contain the detailed step-by-step guide, architectural diagrams, code snippets, and thorough explanations.
* **Problem Statement / Business Scenario:** A clear definition of the real-world challenge or business need being addressed by the project.
* **Architecture & Design:** An overview of the components involved, their interaction, and the design choices made for the solution.
* **Implementation Steps:** Granular, command-by-command or click-by-click instructions for setting up and configuring the solution.
* **Validation & Testing:** How to verify the successful implementation and functionality of the deployed solution.
* **Key Learnings & Considerations:** Insights, best practices, potential challenges encountered, and alternative approaches.
* **Screenshots & Diagrams:** Visual aids to enhance understanding of complex configurations and flows.

---

## Getting Started / Contributions

Feel free to explore the projects and leverage them for your own learning and development. If you have questions, suggestions for improvements, or would like to contribute similar on-premises project guides, please feel free to open an issue or submit a pull request. Your contributions are welcome!

---
```
