---
title: How to Upgrade FactoryTalk Historian v5 to v8
date: 2023-01-16 00:00:00 -0800
categories: [SCADA, Historian]
tags: [rockwell, factorytalk, historian, vantagepoint, directory, howto]     # TAG names should always be lowercase
---

Recently I was tasked with helping a customer upgrade their Factorytalk Historian SE ver. 5.01 to 8.00 during their Christmas shutdown. Here’s a document putting together all the helpful documentation and information in one place.

I’ll be breaking this process down into the following steps:
1. Plan
2. Backup
3. Test
4. Execute

# Plan

## Architecture

A good place to start in the upgrade process is the current architecture of the Historian. Historian supports various architectures from standalone single-server installations to distributed installations where components are separated on different servers. Below is an excerpt from the “FactoryTalk Historian SE Installation and Configuration Guide” for version 5.01.

![historian-architecture](/assets/2023-01-16-How-To-Upgrade-FT-Historian/historian-architecture.png)
*This diagram shows Historian on a single server with FT Directory on a separate server.*

![historian-architecture2](/assets/2023-01-16-How-To-Upgrade-FT-Historian/historian-architecture2.png)
*This diagram shows the Historian can be distributed in different ways.*

### Single Server vs. Distributed - Which architecture is for me?

For smaller systems, a standalone deployment is simpler and quicker to install. As a system becomes larger, distributed systems can help with administrative tasks like isolating issues/troubleshooting, upgrading, and migrating.

## Order

When upgrading Historian, it is important to note that the upgrade process has to be done in steps/phases depending on the starting and ending version. For example, our customer had to go from 5.01 to 7.00 to 8.00. 

“Appendix E: Upgrading FactoryTalk Historian SE” in the “FactoryTalk Historian SE Installation and Configuration Guide” for version 8.00 is a useful reference document to understand some of the nuances of upgrading.

Another useful resource was <a href="https://www.youtube.com/watch?v=2PqUDiwy9TM" target="_blank" rel="noopener noreferrer">this video</a> showing an upgrade from 5.01 to 6 on a different machine.

<strong>Protip: Compare each Historian version with Windows and SQL version requirements.</strong>

![historian-migration-path](/assets/2023-01-16-How-To-Upgrade-FT-Historian/historian-migration-path.png)
*Example of planning for customer Historian, FT Directory, and Vantagepoint Upgrade/Migration.*

# Backup

Once the architecture and upgrade order have been determined, we need to gather/create all the backup files before our Testing step.

Depending on your specific architecture/project, different backup files may be needed. In our scenario, we needed to backup Historian, Vantagepoint, and FT Directory. Within Historian and Vantagepoint installations, there are MSSQL databases that need to be backed up.

## Backup Historian

1. Backup Historian with SMT tool. <a href="https://rockwellautomation.custhelp.com/app/answers/answer_view/a_id/1134130" target="_blank" rel="noopener noreferrer">QA63345 (TechConnect Req.)</a>

![backup-historian](/assets/2023-01-16-How-To-Upgrade-FT-Historian/backup-historian.png)

2. <a href="https://youtu.be/rcJdDYbhpZU?t=112" target="_blank" rel="noopener noreferrer">Backup PIFD MSSQL database</a>.

3. Backup Interface .bat files. <a href="https://rockwellautomation.custhelp.com/app/answers/answer_view/a_id/601857" target="_blank" rel="noopener noreferrer">QA57155 (TechConnect Req.)</a>

![backup-interface](/assets/2023-01-16-How-To-Upgrade-FT-Historian/backup-interface.png)

## Backup Vantagepoint

Follow the extensive guide available on TechConnect. <a href="https://rockwellautomation.custhelp.com/app/answers/answer_view/a_id/62884/" target="_blank" rel="noopener noreferrer">QA8466 (TechConnect Req.)</a>

## Backup FT Directory

To backup the FactoryTalk Directory, open FactoryTalk Administration Console, Select Network or Local depending on your architecture.

![backup-ftdirectory](/assets/2023-01-16-How-To-Upgrade-FT-Historian/backup-ftdirectory.png)

Next, Right-Click the Hostname of your FT Directory and select ‘Backup…’

![backup-ftdirectory2](/assets/2023-01-16-How-To-Upgrade-FT-Historian/backup-ftdirectory2.png)

# Test

Once the backup files have been collected, a test environment can be set up to practice any administrator tasks such as restore, migration, upgrading etc. in a safe place without affecting your production environment. Here we can refine our process and document any issues we run into to make the real transition as smooth as possible.

I have a homelab setup with an Intel NUC running VMware ESXi. I can spin up VMs for different servers to test configuration, migration, etc.

The general steps are below:
- Recreate the production environment in a lab environment.
- Practice backup/restore onto a new intermediate environment.
- Create a new FT directory VM.
- Restore FT directory configuration.
- Point all servers to the new FT directory VM.	
- Create a new Asset Framework database server with Historian v7.01-compatible MSSQL version (2014 SP3), restore the db, and point Historian to the new db.
- Upgrade Historian in place from v5 to v7.01.
  - If having issues with installation, try running the PISDK Setup located on DVD\Redist\PISDK\Setup.exe (ran into a bug with a newer Intel chip and Win 2012 R2 for v7).
- Upgrade SQL db compatibility to 2014. (See Historian v7 Release Notes)
- Run AF Server upgrade from Historian installer and point to AF Server.
- Verify Historian installation with SMT and MDB to AF sync.
- Backup Historian.
- Upgrade Historian in place from v7.01 to v8.
  - If installing Historian v8 on Win 2012 R2, install Windows Update KB2999226 beforehand. (See Historian v8 Release Notes)
- Run AF Server upgrade from Historian installer and point to AF Server.
- Verify Historian installation with SMT and MDB to AF sync.
- Create a new Asset Framework database server with SQL 2016 (Client requirement), backup and restore AF Server here and point Historian to the new db.
- Backup/restore Historian from Migration VM to Test Production VM.
- Verify/Configure Historian interface in FT Directory.
- Install Vantagepoint on a new Win 2019 VM.
  - Install IIS services.
  - Install Microsoft Excel 32-bit if Excel reports required.
  - Install all Vantagepoint components.
- Restore Vantagepoint MSSQL db’s onto the new VM.
- Configure Vantagepoint to look at the new Historian VM.

# Execute

Great! Now that testing is complete, use your notes and references to perform the migration/upgrade for your system.

Once your system has been upgraded and tested, don’t forget to install the latest patch rollup from Rockwell Automation’s PCDC website.

# Notes/Tips

FactoryTalk Historian, Vantagepoint, and FT Directory heavily involve Windows systems administration. Rely on your IT team for granting admin access and brush up on your Windows admin skills.

Some of these skills include:

## Windows

- Active Directory
- Group Policy
- IIS Administration
- Workgroup vs. Domain
- Time Sync between servers
- Running services under Windows service accounts

## SQL Server

- SQL Server Management Studio (SSMS)
- SQL Administration (Permissions, remote access, backup/restore)

Additional resources that were helpful were the following Rockwell Techconnect Knowledgebase articles. Unfortunately, many require a Techconnect subscription.

![knowledgebase](/assets/2023-01-16-How-To-Upgrade-FT-Historian/knowledgebase.png)

And if you have a Rockwell Support contract, definitely leverage reaching out to them for any issues or questions.

Lastly, FactoryTalk Historian is in many ways OSIsoft PI, with some interfacing with FT Directory/Activation Manager. OSIsoft has many free resources available on their website, forum, and YouTube that are helpful for general understanding and thorough tutorials.
