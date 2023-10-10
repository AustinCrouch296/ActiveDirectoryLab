# ActiveDirectoryHomeLab
<img src="/images/main_diagram.png">

## Introduction
In this lab, I will explain how to set up and configure an Active Directory Domain Controller, a Windows 10 client machine to connect to it, and create 1k+ user accounts using a PowerShell script, to create a simulated real-world network environment that we can use to learn more about Active Directory, Group Policies, and troubleshoot common network issues.

### Files necessary to follow along:
[Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)<br />
[Windows Server 2019 ISO](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)<br />
[Windows 10 ISO](https://www.microsoft.com/en-us/software-download/windows10)</a><br />
[PowerShell Create Accounts Script (GitHub)](https://github.com/joshmadakor1/AD_PS)</a>

# Procedures
## **Creating a Windows Server 2019 Virtual Machine**
Download and open Oracle VM VirtualBox.

Click Machine > New, and add a name for your Domain Controller ("DC" or "DomainController").<br />
Set the Type to "Microsoft Windows" and Version to "Windows 2019", and press Next.<br />
*(Note: I do not to link the ISO until I boot up the VM and am prompted to, because I've ran into issues doing so previously.)*<br />

For the following setup screens, these are the options I used:<br />
- Hardware:<br />
    - Base Memory: 2048 MB<br />
    - Processors: 1<br />
- Virtual Hard Disk:<br />
    - Disk Space: 20.00 GB<br />

Select the Virtual Machine you have just created, and press Settings:<br />
- General > Advanced<br />
    - Shared Clipboard: Bidirectional<br />
    - Drag'n'Drop: Bidirectional<br />
- Network > Adapter 1:<br />
    - Enable Network Adapter: True<br />
    - Attached to: NAT<br />
- Network > Adapter 2:<br />
    - Enable Network Adapter: True<br />
    - Attached to: Internal Network<br />
    - Name: intnet<br />

Select OK, click the Virtual Machine again, and select Start.<br />
The virtual machine will load and ask you to link an ISO file.<br />
Download and link the Server 2019 ISO and press "Mount and Retry Boot".<br />

**We now have the Virtual Machine setup, the next steps will take place inside of Windows!**

## **DC: Windows Setup Wizard**

Continue through the Windows Setup wizard.<br />
Select these are the options as they appear:<br />
- Select the operating system you want to install.<br />
    - Windows Server 2019 Standard Evaluation (Desktop Experience)<br />
- Which type of installation do you want?<br />
    - Custom: Install Windows only (advanced)<br />
    - Click next to following screen to select the default partition.<br />
- Customize Settings:<br />
    - Password: Password1<br />
    *(To keep the lab simple, I suggest using the same password throughout the entire lab.)*<br />

## **Configuring NICs (Internal/External Networks)**
Navigate to Control Panel > Network and Internet > Network and Sharing Center > Change Adapter Settings.<br />
Right click on the adapter named "Ethernet" > select Status.<br />
- Verify that it displays "IPv4 Connectivity: Internet", and close the window.<br />
- Right-click on "Ethernet" again, choose Rename > and set the name to "\_INTERNET\_".<br />
- Right-click on "Ethernet 2", choose Rename > and set the name to "\_INTNET\_".<br />
*(This will help make our networks more easily identifiable in the future.)*<br />

We will now change the IPv4 settings of the internal network to those in the diagram above.<br />
- Right-click on "\_INTNET\_" > select Properties.<br />
- Click Internet Protocol Version 4 (TCP/IPv4) > click Properties.<br />
    - General > Enable "Use the following IP Address:"<br />
    - IP address: 172.16.0.1<br />
    - Mask: 255.255.255.0<br />
    - Gateway: &lt;empty&gt;<br />
    - Preferred DNS server: 127.0.0.1 (Loopback address)<br />
    - Alternate DNS server: &lt;empty&gt;<br />

## **Renaming the PC**
Right-click on the Start Menu > select System.<br />
- Click the "Rename this PC" button.<br />
- Change the name of the device to "DC" for Domain Controller.<br />
- Choose the option to Restart now to apply the changes we have made so far.<br />

## **Configuring Active Directory Domain Services**
Now that we have configured our NICs, the next step is to setup our Domain (Active Directory Domain Server).<br />

Open Server Manager > select "Add roles and features". <br />
Click next until you get to the Select destination server/Server Selection tab.
 <br />
- Select DC, and click Next.<br />
- Select Active Directory Domain Services, and click Next.<br />
- Click Next through the rest of the settings, and click Install.<br />

On Server Manager there should now be a Notifications icon (a flag with a yellow caution symbol) in the top right.<br />
Select it, and select "Promote this server to a domain controller".<br />
This will open the Active Directory Domain Services Configuration Window, choose the following settings:<br />
- Deployment Configuration:<br />
    - Select the deployment option: Add a new forest.<br />
    - Root domain name: mydomain.com<br />
- Domain Controller Options<br />
    - Password & Confirm Password: Password1<br />
- Click Next through the rest of the settings, and click Install.<br />

### **Creating a dedicated domain admin user account**
Open Active Directory Users and Computers. <br />
Right-click on mydomain.com > select New > Organizational Unit<br />
- Name: "_ADMINS"<br />
    - Right-click "_ADMINS" group > New > User<br />
- Now set up the Admin account with your own information, in this format:<br />
    - First name: John<br />
    - Last name: Smith<br />
    - User logon name: j-smith@mydomain.com<br />
- Click Next.<br />
    - Password & Confirm Password: Password1<br />
    - Uncheck "User must change password at next logon"<br />
    - Check "Password never expires"<br />
- Click Next and Finish.<br />

To set your new User Account as a Domain Admin, select the user account, right-click > Properties.<br />
- Member Of > Add<br />
- In "Enter the object names to select (examples):<br />
    - Type "domain admins".<br />
    - Click Check Names<br />
    - Apply the settings.<br />

Now we can sign out of our account, and sign in with our new user account.<br />
On the Sign-in screen, select Other Users, and enter the credentials for your user. <br />
- For example:<br />
  - Username: j-smith<br />
  - Password: Password1<br />

## **Configuring Remote Access Server (RAS)/NAT**
We will set up a RAS/NAT to allow our client virtual machines to be on a private virtual network, but still be able to access the Internet through the Domain Controller.<br />

Open Server Manager > select "Add roles and features". <br />
Click next through the Wizard, choosing these options on the following tabs:<br />
- [Server Selection] Select DC<br />
- [Server Roles] Select Remote Access<br />
- [Role Services] Select DirectAccess and VPN (RAS)<br />
    - Select Routing<br />
- Click Next through the rest of the settings, and click Install.<br />

Navigate to Server Manager > Tools > Routing and Remote Access<br />
- Right-click on 'DC (local)' and select Configure and Enable Routing and Remote Access<br />
- Click next through the Wizard, choosing these options on the following tabs:<br />
    - [Configuration] Select Network address translation (NAT)<br />
    - [NAT Internet Connection] Select "Use this public interface to connect to the Internet"<br />
        - Select \_INTERNET\_.<br />
    - *(If these options are unavailable, try closing and reopening Routing and Remote Access).*<br />
- Click Next through the rest of the settings, and click Finish.<br />

Now there should be a green symbol next to 'DC (local)' in Routing and Remote Access, this means that we configured it correctly. Now client VMs should be able to once we set up DHCP for them!

## **Configuring the DHCP Server**
Open Server Manager > select "Add roles and features". <br />
- Click next through the Wizard, choosing these options on the following tabs:<br />
    - [Server Selection] Select DC.mydomain.com<br />
    - [Server Roles] Select DHCP Server<br />
    - Click Next through the rest of the settings, and click Install.<br />

- Navigate to Server Manager > Tools > DHCP<br />
  - Expand the dc.mydomain.com drop-down menu.<br />
  - Right-click on IPv4 and select New Scope...<br />

- Click next through the Wizard, choosing these options on the following tabs:<br />
    - [Scope Name] Name: 172.16.0.100-200<br />
    - [IP Address Range] 
        - Start IP Address: 172.16.0.100<br />
        - End IP Address: 172.16.0.200<br />
        - Length: 24<br />
        - Subnet mask: 255.255.255.0<br />
    - [Router (Default Gateway)] IP address: 172.16.0.1 (and **click Add**!)<br />
- Click Next through the rest of the settings, and click Finish.<br />
    - Right-click on dc.mydomain.com and select Authorize.<br />
    - Right-click on dc.mydomain.com and select Refresh.<br />
    - A green symbol should appear next to IPv4 and IPv6 to show that DHCP is now properly configured!

### **How to enable internet access from this virtual machine**
*(In a real production environment, you would not want to allow a domain server to access the Internet.)* <br />

- Inside of Server Manager:<br />
    - Select the Local Server tab on the left-hand side.<br />
    - Find the properties set for "IE Enhanced Security Configuration: On".<br />
    - Change the setting to Off for both Administrators and Users.<br />

## **Populating the domain with test accounts using PowerShell**
Download the PowerShell script from this repository onto the Server by copying this link and pasting it into Internet Explorer.<br />
Download and open the .zip file, choose the Compressed Folder Tools options tab and press Extract all.<br />
Browse to your Desktop folder and select Extract.<br />

The *names.txt* file contians a list of names that our PowerShell script will use to populate our Active Directory with users accounts.<br />
<br />
Open *names.txt*, add your own name to the file, and save it.<br />
Run Windows PowerShell ISE as Administrator.<br />
In PowerShell click File > Open and navigate to the AD_PS Folder/CREATE_USERS.ps1.<br />

- Type the command "Set-ExecutionPolicy Unrestricted" into the PowerShell Terminal, and press enter.<br />
    - Select "Yes to all" on the window that appears.<br />
- Type the command "cd C:\Users\j-smith\Desktop\ActiveDirectoryHomeLab".<br />
  - *(Replace "j-smith" with the username for the admin account that you created.)*<br />
- Press the Run Script button (or the F5 key) to start execute the script.<br />

**Now if you open Active Directory Users and Computers, expand mydomain.com, and click the _USERS folder, you can see all of the users inside!**<br />
*Note that the way we formatted these user accounts is different than how we set up the admin account:*<br />
- *j-smith (Administrator)*<br />
- *jsmith (User)*

## **Creating a Windows 10 "Client" Virtual Machine**
Created Windows 10 ISO using Media Creation Tool<br />
- Made clipboard & drag/drop bi-directional for ease of use<br />
- Assigned 20GB storage, 2048MB RAM, 2 CPUs<br />
- Changed Network from NAT to Internal Network, to use DC’s DHCP server. (and simulate a real corporate network)<br />
- Used ipconfig and ping mydomain.com to make sure connectivity was working correctly.<br />
- Checked hostname and change by going to Settings > Rename this PC (advanced) to both rename PC and join domain at the same time:<br />

In Oracle VM VirtualBox - Click Machine > New, and add a name for your Client VM ("CLIENT1").<br />
Set the Type to "Microsoft Windows" and Version to "Windows 10", and press Next.<br />

For the setup screens, use the same options as the Server VM:<br />
- Hardware:<br />
    - Base Memory: 2048 MB<br />
    - Processors: 1<br />
- Virtual Hard Disk:<br />
    - Disk Space: 20.00 GB<br />

Select the Virtual Machine, and press Settings:<br />
- General > Advanced<br />
    - Shared Clipboard: Bidirectional<br />
    - Drag'n'Drop: Bidirectional<br />
    (This will allow you to copy/paste between your PC and VM.)<br />

**For this CLIENT VM, we will only use an Internal network as internet access will be provided for via our DC's DHCP server.**<br />
- Network > Adapter 1:<br />
    - Enable Network Adapter: True<br />
    - Attached to: Internal Network<br />
    - Name: intnet<br />

Select OK, and start the Virtual Machine.<br />
The virtual machine will ask you to link an ISO file.<br />
Link the Windows 10 ISO that you have created, and press "Mount and Retry Boot".<br />

## **Client VM: Windows Setup Wizard**

Continuing through the Windows Setup wizard, these are the options you should select when prompted to:<br />

- Activate Windows<br />
    - "I don't have a product key".<br />

- Select the operating system you want to install.<br />
    - Windows 10 Pro **(IMPORTANT!)**<br />
- Which type of installation do you want?<br />
    - Custom: Install Windows only (advanced)<br />
    - Click next to following screen to select the default partition.<br />
- Let's connect you to a network<br />
    - "I don't have internet"<br />
    - Continue with limited setup<br />
- Who's going to use this PC?<br />
    - Name: user<br />
    - Password: Password1<br />
- Services<br />
    - For privacy settings, I like to disable everything, and skip customization.<br />

And there we go! Now if you open Command Prompt and type `ipconfig` then all of your internet settings should all be assigned from your Domain Controller. (As long as you have both VMs running at the same time!)<br />
You should have access to the internet via the Domain Controller, and you can verify this by running the command `ping google`, or `ping mydomain.com` to ping the DC and get a response.<br />

There is only one final step in joining the Domain Controller, and that is:<br />
- Open Settings > System > About > Related Settings > Rename this PC (advanced).<br />
- [Computer Name] Press the "Change..." button.<br />
    - Computer name: CLIENT1<br />
    - Member Of: Domain: mydomain.com<br />
    - A Windows Security tab will open, and you can enter the user account you created for yourself to give the PC access to the domain. (jsmith/Password1)<br />
    - Allow the PC to restart to join it to the domain.<br />

When the VM reaches the sign-in screen, you can now choose the "Other user" option and sign-in with any of the user accounts that we created earlier. A new profile will be built whenever a new user signs in.<br />

# *Conclusion*

We now have a simulated corporate network environment that we can use to experiment in!<br />
There are a ton of different possibilities to use this environment for experimenting and get hands-on experience in providing troubleshooting issues in the Windows Operating System, and learning about Active Directory, DNS, DHCP, Group Policy, and really anything else!<br />

Here are some examples of ways that I have used this lab, which I may go over in-depth in a Group Policy lab in the future!<br />
- Creating Group Policies objects to apply a computer and user configurations. Enforcing wallpapers, password account policies, deploying software, configuring Windows Firewall (to enable things like RDP), drive mapping policies, user folder redirection to keep user profiles saved on the DC itself.<br />

- Setting up Network Shares (NTFS & Share Permissions) to allow specific users/security groups to access files from the Domain Controller.<br />

- Learning how to set up Task Scheduler (for automated backups), creating scenarios of real-world issues and using Event Viewer logs and research to give yourself useful experience in solving issues that you will face in the real world in Information Technology. 
