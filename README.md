# ActiveDirectoryHomeLab
<img src="/images/main_diagram.png">

## Introduction
In this lab, I will explain the procedures necessary to set up and configure an Active Directory Domain Controller, a Windows 10 client machine, and create 1k+ user accounts using a PowerShell script, and create a simulated real-world network environment that can be used to explore Active Directory, Group Policy, and 

### Files necessary to follow along:
<a href="https://www.virtualbox.org/wiki/Downloads">Oracle VirtualBox</a><br />
<a href="https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019">Windows Server 2019 ISO</a><br />
<a href="https://www.microsoft.com/en-us/software-download/windows10">Windows 10 ISO</a><br />
<a href="https://github.com/joshmadakor1/AD_PS">PowerShell Create Accounts Script (GitHub)</a>

## Procedures
### Creating a Windows Server 2019 Virtual Machine
Download and open Oracle VM VirtualBox.

Click Machine > New, and add a name for your Domain Controller ("DC" or "DomainController").<br />
Set the Type to "Microsoft Windows" and Version to "Windows 2019", and press Next.<br />
(Note: I do not to link the ISO until I boot up the VM and am prompted to, because I've ran into issues doing so previously.)<br />

For the following setup screens, these are the options I used:<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Hardware:<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Base Memory: 2048 MB<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Processors: 1<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Virtual Hard Disk:<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Disk Space: 20.00 GB<br />

Select the Virtual Machine you have just created, and press Settings.<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;General > Advanced<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Shared Clipboard:&nbsp;Bidirectional<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Drag'n'Drop:&nbsp;Bidirectional<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(This will allow you to copy and paste between your PC to the VM, which can be helpful.)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Network > Adapter 1:<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Enable Network Adapter: True<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Attached to: NAT<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Network > Adapter 2:<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Enable Network Adapter: True<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Attached to: Internal Network<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Name: intnet<br />
Press OK.<br />

Select the Virtual Machine again, and press Start.<br />
Once it loads, VirtualBox will prompt you to link the Windows ISO, link the Server 2019 ISO and press "Mount and Retry Boot".<br />
From here, we will be working in Windows!

#### Configuring NICs (Internal/External Networks)
Open and navigate to the Control Panel > Network and Intenet > Network and Sharing Center > Change Adapter Settings<br />
Right click on "Ethernet" > select Status.<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;It should display "IPv4 Connectivity: Internet".<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Close the window, right-click on "Ethernet" again, choose Rename > and set the name to "_INTERNET_".<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;After that, right-click on "Ethernet 2", choose Rename > and set the name to "_INTNET_".<br />
This will help us more easily identify them.<br />

We will now change the IPv4 settings of the internal network to those in the diagram above.<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Right-click on "_INTNET_" > select Properties.<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Click Internet Protocol Version 4 (TCP/IPv4) > click Properties.<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;General > Enable "Use the following IP Address:"<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IP address: 172.16.0.1<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Mask: 255.255.255.0<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Gateway: &lt;empty&gt;<br />
        
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Preferred DNS server: 127.0.0.1<br />(Loopback address)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Alternate DNS server: &lt;empty&gt;<br />

#### Renaming the PC
Right-click on the Start Menu > select System.<br />
Click "Rename this PC".<br />
Change the name of the device to "DC" for Domain Controller.<br />
Click Restart now to apply the changes we have made so far.<br />

#### Configuring Active Directory Domain Services

#### Configuring Remote Access Server (RAS)/NAT

#### Configuring the DHCP Server

##### How to enable internet access from this virtual machine
*Note in a real production environment, it would NOT be a good idea to allow this*

#### Populating the domain with test accounts using PowerShell

### Creating a Windows 10 "Client" Virtual Machine

### Experimenting with Group Policy 

## Conclusion
