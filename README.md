<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" height="40%" width="70%"alt="Microsoft Active Directory Logo"/>
</p>

<h1>Configuring Active Directory (On-Premises) Within Azure</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<!-- <h2>Video Demonstration</h2>

- ### [YouTube: How to Deploy on-premises Active Directory within Azure Compute](https://www.youtube.com) -->

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop Connection
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Create Resources
- Ensure Connectivity between the client and Domain Controller
- Install Active Directory
- Create an Admin and Normal User Account in AD
- Join Client-1 to your domain (mydomain.com)
- Setup Remote Desktop for non-administrative users on Client-1
- Create additional users and attempt to log into client-1 with one of the users

<h2>Deployment and Configuration Steps</h2>
<br />
<br />
<h3 align="center">Setup Resources in Azure</h3>
<br />
<p>
<p>
      Create the Resource Group where both of your VMs and your virtual net will exist:
</p>
<p>
     <img src="https://i.postimg.cc/43QknJPY/Create-Resource-Group.png" height="75%" width="100%" alt="resource group"/>
</p>
<p>
    Create the Virtual Network your VMs will connect to so they can communicate with each other. Be sure to place this in the Resource Group you created in the last step:
</p>
<p>
    <img src="https://i.postimg.cc/7Y2RfHYq/Create-Virtual-Network.png" height="75%" width="100%" alt="virtual network"/>
</p>
<p>
    Create the Windows Server 2022 Domain Controller VM named "DC-01". Place it in the Resource Group you made and attach it to the same virtual network you made. NOTE: Create a new public IP too. It'll save you time later:
</p>
<p>
   <img src="https://i.postimg.cc/mZHxFNmx/DCVM-1.png" height="75%" width="100%" alt="DC creation 1"/>
   <img src="https://i.postimg.cc/WzMfY8yh/DCVM-2.png" height="75%" width="100%" alt="DC creation 2"/>
</p>

</p>
<p>
  Create the Client VM (Windows 10) named “Client-1”. Use the same Resource Group and Vnet that was created in the previous step:
</p>
<p>
  <img src="https://i.postimg.cc/52YdH5DZ/Client-1-VM-1.png" height="75%" width="100%" alt="vm windows"/>
  <img src="https://i.postimg.cc/QMQz5fmn/Client-1-VM-2.png" height="75%" width="100%" alt="vm windows"/>
</p>
<p>
  Set Domain Controller’s (Windows Server 2022 VM) NIC Private IP address to be static. It should be set to dynamic by default; just change it to static:
</p>
<p>
  <img src="https://i.postimg.cc/BQPqQMJx/Static-Private-IP-DCVM.png" height="75%" width="100%" alt="static ip"/>
</p>

<br />
<br />
<h3 align="center">Ensure Connectivity between the client and Domain Controller</h3>
<br />
<p>
  Login to the Domain Controller (DC-01), video on that <a href="https://youtu.be/u0qzXQTW3yk?si=XBLIIh_5TY9Y8PeO" target="_blank">here</a>, and enable ICMPv4 on the local Windows firewall. This is a very important step. A connection cannot be made between the VMs if you skip this:
</p>
<p>
  <img src="https://i.postimg.cc/d3d0rtP9/Enable-ICMPv4-DCVM.png" height="75%" width="100%" alt="enable ICMPv4"/>
</p>
<p>
  Login to Client-1 with Remote Desktop Connection and ping DC-01’s private IP address with ping -t <ip address> (perpetual ping):
</p>
<p>
  <img src="https://i.postimg.cc/L5VCLzwV/Good-Connection-Between-VMs.png" height="75%" width="100%" alt="perpetual ping"/>
</p>
        
<br />
<br />
<h3 align="center">Install Active Directory</h3>
<br />
<p>
  Login to DC-01 and install Active Directory Domain Services from the add roles and features wizard:
</p>
<p>
  <img src="https://i.postimg.cc/250WhVjP/Install-Active-Directory.png" height="75%" width="100%" alt="active directory install"/>
</p>
<p>
  Promote as a Domain Controller. Click on "promote this server to a domain controller." I have arrows pointed to it in the picture below, and follow the setup wizard. Click add a new forest and name the domain whatever you want or just copy what I did. Remember what it is though:
</p>
<p>
  <img src="https://i.postimg.cc/mgpVBnmQ/Promote-To-Domain-Controller.png" height="75%" width="100%" alt="domain controller promotion"/>
</p>

<br />
<br />
<h3 align="center">Create an Admin and Normal User Account in AD</h3>
<br />
<p>
  After DC-01 restarts, log back into it to complete these steps. Click on tools in the top right. In Active Directory Users and Computers (ADUC), create an Organizational Unit (OU) called “_EMPLOYEES” and another one called "_ADMINS":
</p>
<p>
  <img src="https://i.postimg.cc/BbQNQgNq/Add-New-Organizational-Unit.png" height="75%" width="100%" alt="organizational unit"/>
  <img src="https://i.imgur.com/v02CBPI.png" height="75%" width="100%" alt="organizational unit"/>
</p>
<p>
  Create a new employee named “John Smith” with the username of “john_smith”. Take note of the password:
</p>
<p>
  <img src="https://i.postimg.cc/t4kPtMQy/Create-New-User.png" height="75%" width="100%" alt="admin creation"/>
</p>
<p>
  Add john_smith to the “Domain Admins” Security Group. Right-click their name and select properties, then follow the path in the image below:
</p>
<p>
  <img src="https://i.postimg.cc/pdQSZxVr/Add-User-To-Group.png" height="75%" width="100%" alt="security group"/>
</p>
<p>  
  Log out/close the Remote Desktop connection to DC-01 and log back in as “john_smith@mydomain.com.” Use john_smith as your admin account from now on. NOTE: The picture says john_admin because that's the user logon name I gave him, but john_smith works too:
</p>
<p>
  <img src="https://i.postimg.cc/Xqtw4fvv/Log-In-As-New-User.png" height="75%" width="100%" alt="admin login"/>
</p>
<br />
<br />
<h3 align="center">Join Client-1 to your domain (mydomain.com)</h3>
<br />
<p>
  From the Azure Portal, set Client-1’s DNS settings to the DC’s Private IP address. The path to DNS settings is up top in the picture. Save it and then reset Client-1 in the Azure Portal:
</p>
<p>
  <img src="https://i.postimg.cc/25X3rZj9/DNSSetup-On-Client-1.png" height="75%" width="100%" alt="client dns settings"/>
</p>
<p>
  Login to Client-1 (Remote Desktop) as the original local admin you made when you first created the VM and join it to the domain (the computer will restart):
</p>
<p>
  <img src="https://i.postimg.cc/XqFvKNSV/Client-1-To-Domain.png" height="75%" width="100%" alt="domain joining"/>
</p>
<p>
  Login to the Domain Controller (Remote Desktop) and verify Client-1 shows up in Active Directory Users and Computers (ADUC) inside the “Computers” container on the root of the domain.
</p>
<p>
  Create a new OU named “_CLIENTS” and drag Client-1 into there:
</p>
<p>
  <img src="https://i.postimg.cc/vmnMNYHk/New-OUand-Client-1-Added.png" height="75%" width="100%" alt="active directory client verification"/>
</p>
<br />
<br />
<h3 align="center">Setup Remote Desktop for non-administrative users on Client-1</h3>
<br />
<p>
  Log into Client-1 as jonh_smith@mydomain.com and open system properties.
</p>
<p>
  Click “Remote Desktop”.
</p>
<p>
  Allow “domain users” access to remote desktop.
</p>
<p>
  You can now log into Client-1 as a normal, non-administrative user now.
</p>
<p>
  Normally, you’d want to do this with a Group Policy that allows you to change MANY systems at once (maybe a future lab):
</p>
<p>
  <img src="https://i.postimg.cc/dtytkjGG/Set-Up-Remote-Desktop-Userson-Client-1.png" height="75%" width="100%" alt="remote desktop setup"/>
</p>
<br />
<br />
<h3 align="center">Create a bunch of additional users and attempt to log into client-1 with one of the users</h3>
<br />
<p>
  Login to DC-1 as john_smith
</p>
<p>
  Open PowerShell_ise as an administrator.
</p> 
<p>  
  Create a new File and paste the contents of this script (https://github.com/mikiahcarnes/azuread/blob/main/adscript.ps1) into it. I recommend changing the $NUMBER_OF_ACCOUNTS_TO_CREATE variable to something like 50 or 100; 1000 takes too long to create. After you do that, click the run button circled in the picture below and observe the users being created:
</p>
<p>
  <img src="https://i.postimg.cc/TYPGn8B8/Script-To-Add-Users.png" height="75%" width="100%" alt="create users script"/>
</p>
<p>
  When finished, open ADUC observe the accounts in the appropriate OU, and attempt to log into Client-1 with one of the accounts; the password is Password1 unless you change it in the script you ran:
</p>
<p>
  <img src="https://i.postimg.cc/tRx4YH0B/Generated-Users.png" height="75%" width="100%" alt="employee user accounts"/>
  <img src="https://i.postimg.cc/QMYpmKqg/Login-As-New-Created-User.png" height="75%" width="100%" alt="employee user selection"/>
</p>
<br />
<br />
<p>
  That's it. Now that we're done DON'T FORGET TO CLEAN UP YOUR AZURE ENVIRONMENT so that you don't incur unnecessary charges.
</p>
<p>
  Close your Remote Desktop connection, stop the virtual machines, delete the Resource Group(s) created at the beginning of this tutorial, and verify the Resource Group deletion.
</p>
