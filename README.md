<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

1) Prepare Active Directory Infrastructure in Azure
    - Setup Domain Controller VM (Windows Server 2022) named “dc-1”
    - Setup Client VM (Windows 10) named “client-1”
2) Deploy Active Directory
    - Create a Domain Admin user within the domain
    - Join client-1 to your domain (mydomain.com)
3) Create users using PowerShell
    - Setup Remote Desktop for non-administrative users on client-1
    - Create a bunch of additional users and attempt to log into client-1 with one of the users


<h2>Deployment and Configuration Steps</h2>

<h4>1. Prepare Active Directory Infrastructure in Azure</h4>
<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/1.png"/>
</p>
<p>
Create two VMs in Azure ensuring that they are in the SAME VIRTUAL NETWORK and SUBNET MASK (see networking tab when creating VM).

Name one dc-1 (Domain Controller) and set the image to Windows Server 2022.

Name second VM client-1 and set the image to Windows 10.
</p>
<br />

<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/2.png"/>
</p>
<p>
Click into dc-1 -> Network Settings -> Network Interface. We will configure dc-1 to have a static IP address.

Navigate to ipconfig1 -> select Static
</p>
<br />

<h4>2. Deploy Active Directory</h4>
<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/3.png"/>
</p>
<p>
Using dc-1's public IP address, remote desktop into dc-1. Navigate to the start menu -> run -> search for wf.msc

This will open up the firewall settings for DC-1. Go to Windows Defender Firewall Properties and turn off the firewall for each tab. Be sure to click apply when finished.
</p>
<br />

<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/4.png"/>
</p>
<p>
Go to Client-1's Network Settings -> Network Interface -> DNS settings. Enter in DC-1's private IP address (this can be found in DC-1's network settings or overview tab). Be sure to click save.
</p>
<br />

<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/5.png"/>
</p>
<p>
Once the settings are saved navigate back to the virtual machines homepage and give Client-1 a restart by checking its box and clicking "Restart"
When Client-1 has restarted, remote desktop into Client-1 using its public IP address.
</p>
<br />

<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/6.png"/>
</p>
<p>
Within Client-1 open up PowerShell and ping DC-1 to ensure that both VMs are on the same virtual network and that DC-1's firewall is disabled.

Next, type "ipconfig /all and look to make sure the DNS server of Client-1 is set to DC-1's private IP address.
</p>
<br />

<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/7.png"/>
</p>
<p>
Log in to DC-1 and in server manager click "Add role and features". From here add a feature to install Active Directory Domain Services. Click through next to install AD DS.

Once AD DS is installed, go to the top right flag in the server manager and promote the server to become a domain controller. Add a new forest named mydomain.com and continue through the setup wizard. (Be sure to uncheck DNS delegation as you progress through the setup

Once the setup is finished, the VM will reboot.
</p>
<br />

<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/8.png"/>
</p>
<p>
Remote desktop back into DC-1 but this time with a domain specific user. Enter "mydomain.com\labuser" (mydomain.com\username)

This will log in as a specific user within the domain.
</p>
<br />

<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/9.png"/>
</p>
Open Active Directory Users & Computers from within the start menu. By right clicking on mydomain, create a new Organizational Unit (OU).

Create one named _EMPLOYEES and create another named _ADMINS

Once both OUs are created, within _ADMINS right click to create a new user. We will name this user Jane Admin and configure her login credentials.

Something like jane.admin and a password we can remember will suffice.
<p>

</p>
<br />

<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/10.png"/>
</p>
<p>
Once she is created, right click on her to access Properties -> Members of, and add her to "Domain Admins". Be sure to click "Check Names" and apply when finished.

Log out of DC-1 and log back in as “mydomain.com\jane.admin”

We can use jane.admin as the admin account from now on
</p>
<br />

<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/11.png"/>
</p>
<p>
Login to Client-1 as the original local admin (labuser)

From the start menu -> System -> Rename this PC Advanced -> Change, and enter mydomain.com

This will prompt a login screen where we can login as mydomain.com\jane.admin and join Client-1 to the domain controller DC-1
</p>
<br />

<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/12.png"/>
</p>
<p>
We can verify the join was successful by going back into DC-1 and Active Directory Users and Computers (ADUS) and finding Client-1 within the Computer tab.

Create a new OU named _CLIENTS and drag Client-1 into it.
</p>
<br />

<h4>3. Create users using PowerShell</h4>

<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/13.png"/>
</p>
<p>
Log into Client-1 as Jane.Admin -> System -> Remote Desktop -> Select users that can remotely access this PC -> Add -> Domain Users -> Check names -> OK

This will allow any user under the Domain Users group in the domain controller to be able to log into Client-1. We will create users in the User Creation lab.
</p>
<br />

<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/14.png"/>
</p>
<p>
Within DC-1 as Jane.Admin, open PowerShell ISE as ADMINISTRATOR and paste the following script--> https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/AD_PS
  
Be sure to edit the password you want the users to have at the top of the script text.
</p>
<br />

<p>
<img src="https://github.com/BrianRivera22/configure_AD/blob/main/AD%20Implementation%20%26%20PowerShell%20User%20Creation/15.png"/>
</p>
<p>
Check within ADUC and under _EMPLOYEES to see all the created users. Select any user and log into Client-1 to test access

All the created users should have access because we gave domain users access to remote desktop in Client-1

<b>This concludes this walkthrough! Check out Group Policy Objects next!</b>
</p>
<br />
