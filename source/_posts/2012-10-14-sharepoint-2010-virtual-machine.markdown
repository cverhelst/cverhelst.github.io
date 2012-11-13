---
layout: post
title: "SharePoint 2010 - Virtual Machine"
date: 2012-10-14 11:45
comments: true
categories:
- sharepoint
- guide
- virtual
- server
---

# Overview

This guide will outline all the steps needed to intall a SharePoint 2010 Server development environment on a Virtual Machine. 

The particular approach used will leave you with a single VM hosting both _SQL Server 2008 R2_ and _SharePoint 2010 Server_. This VM also acts as a _domain controller_, which requires some additional steps during the installation.

During the installation, several accounts will be used:

* moss_admin (_Installation Account_)
* moss_farm (_SharePoint Farm Administrator_)
* moss_sql (_Account for connecting to the SQL database_)
* moss_service (_Account for running services_)

Ideally, you'll want to use a seperate account for every service, as detailed here: [SharePoint 2010 Accounts](http://www.ericharlan.com/Moss_SharePoint_2007_Blog/sharepoint-2010-service-account-reference-guide-a184.html)

## Sources

* [Installation and deployment for SharePoint Server 2010](http://technet.microsoft.com/en-us/sharepoint/ee518643.aspx)
  * [Initial deployment administrative and service accounts - SharePoint Server 2010](http://technet.microsoft.com/library/ee662513.aspx)
* [Setting up a SharePoint 2010 Development Environment](http://sharepointtaproom.com/2010/06/28/setting-up-a-sharepoint-2010-development-environment/)

## Components used

The following software will be used while setting up the VM in this guide:

* Virtualbox 4.2
* Windows Server 2008 R2
* SQL Server 2008 R2
* SharePoint Server 2010 - Server
  * Service Pack 1
  * Latest CU
  
Additional software for SharePoint Development:

* Visual Studio 2001
  * SP1
  * CKSDEV
* SharePoint Designer
  
# Virtual Machine

I'm using VirtualBox to manage my Virtual machines and I use .VHD virtual machine formats ( they are the Hyper-V format and allow for native boot ).

It is recommended to create a dynamically expanding VM with about 80GB maximum space. This will save disk space but will allow your VM to grow if necessary.
Fixed size VMs are more performant though.

My physical machine has 8GB of RAM and 4 processing threads. I assigned 5GB of RAM and 3 processing threads to my VM which is also running on an external SSD (e-sata).
  
# Windows Server 2008 R2

## Installation

Boot up the VM and install Windows Server 2008 R2 on it
Once you've activated Windows and installed any updates, it is a good idea to take a Snapshot of your VM.

Snapshot: _SPS2010-Base_

## Domain Controller

### Configuration

Set the name of the machine through _System Properties_

Make your Windows Server 2008 R2 a __Domain Controller__.

Enable the _Active Directory Domain Services_ server role
Run _dcpromo_ from Command Prompt and create a single domain (it's possible that after enabling the server role that you'll have this option immediately, so you don't have to run _dcpromo_ afterwards).

Open up __Active Directory Administrative Center__
Create a new forest

### Accounts

After upgrading to a Domain Controller, you are currently logged in with the Administrator account.

Now you need to create the following _Domain User_ accounts to be used later:

* moss_admin
    * Add to the _Administrators_ group
    * Optionally add to the _Domain Administrators_ group so you can make changes to the domain without having to change user
* moss_farm
* moss_sql
* moss_service

Set all of these accounts password to never expire.

Now log in with moss_admin and continue the installation with that user.

## General Configuration

### Overview

* Enable
  * Features
    * .NET Framework 3.5
    * _Windows PowerShell ISE (Optional)_
    * _Desktop Experience (Optional)_
        * Enables some Windows 7 Features on your server which is helpful if you want to use it as your main OS.
    
  * Roles
    * Application Server
    * Web Server (IIS)

* Disable
  * Shutdown Event Tracker
  * Windows Firewall
    * Disable on all profiles
  * IE Enhanced Security
  * User Account Control
  * IIS Loopback Check

### Detail

#### Features

Enable the following features in _Server Manager_

* .NET Framework 3.5
* _Windows PowerShell ISE (Optional)_
* _Desktop Experience (Optional)_
    * Enables some Windows 7 Features on your server which is helpful if you want to use it as your main OS.
  
#### Roles

Enable the following roles in _Server Manager_

* Application Server
* Web Server (IIS)
    * Leave the default settings
    
#### Disable Settings

Disable __Shutdown Event Tracker__:

* Group Policy Management > Forest > Domains > _yourDomain_ > Default Domain Policy -> Right Click -> "Edit"
  Computer Configuration > Policies > Administrative Templates > System
  Setting pane at the right => _Display Shutdown Event Tracker_ -> Right Click -> "Edit" -> Disable
  
Disable __Windows Firewall__:

* Control Panel > System and Security > Windows Firewall > _Turn Windows Firewall On or Off_

Disable on all profiles
  
Disable __IE Enhanced Security__:

* Server Manager > _Security Information_ header > Configure IE ESC

Turn both off.

Disable __User Account Control__:

* Control Panel > System and Security > Change User Account Control Settings

Turn it to the lowest setting

Disable __IIS Loopback Check__:

This allow you to use _alternate access mappings_ for your SharePoint Web Applications (different urls, which you'll also have to add to your hosts file).

PowerShell Code:

    New-ItemProperty HKLM:\System\CurrentControlSet\Control\Lsa -Name "DisableLoopbackCheck" -Value "1" -PropertyType dword

This code sets the registry setting appropriately.

#### Additional Configuration

Configure __Administrator password to never expire__:

* Active Directory Users and Computers > Find the __Administrator__ user -> Right Click -> "Edit"
  Account Tab > _Password Never Expires_
  
Enable SharePoint deployment on a _Domain Controller_:

PowerShell Code:
  
    $acl = Get-Acl HKLM:\System\CurrentControlSet\Control\ComputerName 
    $person = [System.Security.Principal.NTAccount]"Users" 
    $access = [System.Security.AccessControl.RegistryRights]::FullControl 
    $inheritance = [System.Security.AccessControl.InheritanceFlags]"ContainerInherit, ObjectInherit" 
    $propagation = [System.Security.AccessControl.PropagationFlags]::None 
    $type = [System.Security.AccessControl.AccessControlType]::Allow 
    $rule = New-Object System.Security.AccessControl.RegistryAccessRule($person, $access, $inheritance, $propagation, $type) 
    $acl.AddAccessRule($rule) 
    Set-Acl HKLM:\System\CurrentControlSet\Control\ComputerName $acl

Take another snapshot.

Snapshot: _SPS2010-Init_    
    
# Software

Since the last two steps are the installation of SQL Server 2008 R2 and SharePoint 2010, you might want to do these differently in the future. To save you a lot of hassle reinstalling programs like Visual Studio 2010 etc. it is a good idea to install all your extra software at this point in time.

Take another snapshot afterwards.

Snapshot: _SPS2010-PreSQL_
    
# SQL Server 2008 R2

## Installation

Choose the first option _SQL Server Feature Installation_

Select _all_ features.

Leave the default instance name _MSSQLSERVER_

Service Accounts:

Click __Use the same account for all SQL Server Services__.

Use moss_sql and change the startup type of all the services to automatic

Account Provisioning: Leave windows authentication mode and provide the current user ( moss_admin ) as the SQL Server Administrator

Reporting Services Configuration - Install but do not configure

# SharePoint 2010 - Server

## Pre-Installation

Configure your moss_admin account to have the appropriate permissions on the SQL database:

Using SQL Server Management Studio, connect to the database _(local)_

In SSMS > _DBInstance_ > Security > Logins > moss_admin -> Right Click -> "Properties"

In Server Roles, make sure the following are enabled:

* securityadmin
* dbcreator

## Installation

Install the prerequisites manually or just run the prerequisites installer included with the SharePoint 2010 installation media.

Normally, if you updated your SQL Server 2008 R2, this should be limited to:

* MSChart
* Microsoft Geneva Framework
* SQL Server 2008 ASADOMD10
* Windows Identity Foundation (?)

Good time for another Snapshot:

Snapshot: _SPS2010-PreSP_

Because we're on a _Domain Controller_, SharePoint assumes we'll want a standalone installation (which we don't). So to circumvent this, run the installer like so from the command line:

    setup.exe /config Files\SetupFarm\Config.xml

When the installation is completed don't run the _SharePoint Products Configuration Wizard_ yet but install any language pack you might want.

Then run the _SharePoint Products Configuration Wizard_:

Database => ComputerName
Farm account => moss_farm 

port 40000
NTLM


Let's first install the language packs (if any) and upgrade to SP1 + latest CU. Doing the updates before the configuration seems to make them process faster. Make sure to run the installation of the language packs before any upgrade or you'll have to download separate upgrade packages for the language packs specifically later on, while in the other case they will get upgraded as well.

[SharePoint 2010 Cumulative Updates and Service Packs](http://technet.microsoft.com/en-us/sharepoint/ff800847.aspx)

Be aware that for any CU after _August 2011_ that we have since passed, it is no longer necessary to first upgrade foundation and then server, upgrading server alone is now sufficient.

__TODO: This needs a better explanation__

This means that the updates that need to be installed are the following (in this order):
* _Any Language Packs you may want to install_
* SP1 for __SharePoint Foundation__
* SP1 for __SharePoint Server__
* SP1 for each Language pack individually
* Latest CU for __SharePoint Server__


# Appendix 1: Visual Studio 2010 - Installation for SharePoint 2010

During installation, you'll have the option to do either a _full install_ or a _custom install_, choose __custom install__.

From the list of components, select only these:

* _Microsoft Visual Studio 2010 Professional_
  * Visual C#
  * Visual Web Developer
  * Graphics Library
* Microsoft Office Developer Tools(x64)
* Dotfuscator Software Services - Community
* Microsoft SharePoint Developer Tools

__TODO: Check if the bottom 3 are really required__