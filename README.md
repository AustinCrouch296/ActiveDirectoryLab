# ActiveDirectoryHomeLab
<img src="/images/main_diagram.png">

## Introduction
In this lab, I will explain the procedures necessary to set up and configure an Active Directory Domain Controller, a Windows 10 client machine, and create 1k+ user accounts using a PowerShell script, and create a simulated real-world network environment that can be used to explore Active Directory, Group Policy, and 

### Files necessary to follow along:
<a href="https://www.virtualbox.org/wiki/Downloads">Oracle VirtualBox</a>
<a href="https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019">Windows Server 2019 ISO</a>
<a href="https://www.microsoft.com/en-us/software-download/windows10">Windows 10 ISO</a>
<a href="https://github.com/joshmadakor1/AD_PS">PowerShell Create Accounts Script (GitHub)</a>

## Procedures
### Creating a Windows Server 2019 Virtual Machine
Download and open Oracle VM VirtualBox.

Click Machine > New, and add a name for your Domain Controller ("DC" or "DomainController").
Set the Type to "Microsoft Windows" and Version to "Windows 2019", and press Next.
(Note: I do not to link the ISO until I boot up the VM and am prompted to, because I've ran into issues doing so previously.)

For the following screens, these are the options I selected:
    Hardware:   Set the Base Memory to "2048 MB"
                Processors the to "1" 
    Virtual Hard Disk:
                Disk Space: "20.00 GB"

Select the Virtual Machine you have just created, and press Settings.
    Under General > Advanced
        Shared Clipboard:   Bidirectional
        Drag'n'Drop:        Bidirectional
        (This will allow you to copy and paste between your PC to the VM, which can be helpful.)
    Network > Adapter 1:
        Enable Network Adapter: True
        Attached to: NAT
    Network > Adapter 2:
        Enable Network Adapter: True
        Attached to: Internal Network
        Name: intnet
Press OK.

Select the Virtual Machine again, and press Start.
After it loads, VirtualBox will prompt you to link the Windows ISO, link the Server 2019 ISO and press "Mount and Retry Boot".

#### Configuring NICs (Internal/External Networks)

#### Configuring Active Directory Domain Services

#### Configuring Remote Access Server (RAS)/NAT

#### Configuring the DHCP Server

##### How to enable internet access from this virtual machine
*Note in a real production environment, it would NOT be a good idea to allow this*

#### Populating the domain with test accounts using PowerShell

### Creating a Windows 10 "Client" Virtual Machine

### Experimenting with Group Policy 

## Conclusion