<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Video Demonstration</h2>

- ### [YouTube: How to Deploy on-premises Active Directory within Azure Compute](https://youtu.be/8iCBDJq4PYc)

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (22H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Creating Azure Resources
- Creating Virtual Machines and Testing Connectivity
- Install & Configure AD DS
- Create AD Structure in ADUC
- Joining clients to Domain forest
- Bulk User Creation & Testing
- Group Policy Editing
- DNS Management & Configuration
- File Sharing & Permissions

<h2>Deployment and Configuration Steps</h2>

![1](https://github.com/user-attachments/assets/90a9ef13-972a-4a94-aafc-61453c8bb641)
<p>
In Azure, create a resource group along with a virtual network and a subnet. Create two virtual machines, one for the domain controller (I'm naming it as dc-1) running Windows Server 2022 and the other as a client virtual machine running Windows 10. 
</p>
<br />

![2](https://github.com/user-attachments/assets/e656e54f-2224-432b-a0e6-45fbae27342e)
<p>
Once the virtual machines are deployed, head over to the domain controller and go to the network settings under Networking. Click the network interface controller, and head over to the IP settings. Change the ipconfig1 to become static. Go back to the domain controller and copy it's private ip address. Go to your client virtual machine and over to DNS settings inside the network interface controller add the ip address of the domain controller. Restart the client vm.
</p>
<br />

![3](https://github.com/user-attachments/assets/f6a1c4cb-b7fa-43fc-9742-cc95c78f04c2)
<p>
Log onto the domain controller via it's public ip address with Remote Desktop. On the top right, click Manage and add Roles and Features. Under Server Roles, select Active Directory Domain Services and go next to install. Once, it's done installing promote the server as a domain controller. Click add a new forest and setup a domain name (in this example its mydomain.com). Add a DSRM password and click continue until it installs. The computer should be prompted to restart. 
</p>
<br />

![4](https://github.com/user-attachments/assets/0e94076c-6c96-404e-addf-0e571ef56b03)
<p>
Log in back to the domain controller as your username@domain.com. An example is shown above of what it should look like. 
</p>
<br />


![5](https://github.com/user-attachments/assets/23d3910e-fdd0-4bce-b645-c0f0e0d2a700)
<p>
Go to your search bar and look up Active Directory Users and Computers (AUDC). Navigate to your domain, create an Organization Unit (OU) by right-clicking. Create two named _EMPLOYEES and _ADMINS. Under _EMPLOYEES, create a user. Right click on them hit properties, hit the Member of tab and click add. Add them to the Domain Admins. Logout of the current user in the DC and login as that user. 
</p>
<br />

[ezgif-3fdd56ef5c628b.webm](https://github.com/user-attachments/assets/3f1301d2-2679-4262-aa99-4a3a40085e2b)
<p>
Log into the client vm. Go to settings -> System -> System Properties -> click Change, toggle Domain and type in your domain. The computer should prompt you to sign in. Sign in using your user's credentials created before. You should get a prompt that you joined the domain and the computer to be asked to restart now. Note: If you don't see this and you get an error, restart the virtual machine as it's likely the DNS settings didn't get applied. 
</p>
<p>
Restart then, go to the domain controller and head over to ADUC, note that in the computers folder that the client computer is there. Log in the client virtual machine as the user you created before. Head back to settings -> System -> Advanced system settings -> Click Remote tab -> Select Users and add Domain Users to the list.
</p>
<br />

[7.webm](https://github.com/user-attachments/assets/8b6b0c2a-aaec-4915-800f-c2ec2592f85e)
<p>
Back to the domain controller, open up Powershell ISE as an admin. Create a new file and paste contents of this Powershell script below. Run the script and observe all the accounts created in the ADUC. Choose a user and log into the client machine as them. Notice that it works.
</p>
<br />

[Script](https://github.com/blavoir/configure-ad/blob/main/Generate-Names-Create-Users.ps1)

![8](https://github.com/user-attachments/assets/442e347a-6a05-41bd-9744-189c624a746a)
<p>
Back to the Domain Controller, search up gpmc.msc on the taskbar. Navigate to the domain name by the down arrows and find Group Policy Objects. Right click and create a new GPO name. Right click again and hit edit. Navigate to Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Account Policies -> Account Lockout Policy. 
</p>
<p>
Configure account lockout threshold by giving a number of invalid login attempts and Apply. Link the GPO to the domain by right clicking it or dragging up to the domain name. Right click on the right and select Enforced.
</p>
<br />


<p>
<img src="https://i.imgur.com/xqb48sv.png" height="80%" width="80%" alt="Force GP update"/>
<img src="https://i.imgur.com/WnDzVuI.png" height="80%" width="80%" alt="Error message"/>
<p>
Force an update by going to the client virtual machine and within Command Prompt Admin typing gpupdate /force. Sign out of the account and attempt to login with a account incorrectly. Notice the error. 
</p>
<br />

![11](https://github.com/user-attachments/assets/fad181a4-2324-4490-ba4a-9fcf78438f01)
<p>
Unlock it by going to the domain controller, navigating to ADUC, finding the locked user, clicking the Account tab and checking the Unlock account option. Now log in back correctly and notice it works. This is an illustration of a user locked out of their account. 
</p>
<br />



[12.webm](https://github.com/user-attachments/assets/a6bf0afb-620d-47c5-8e47-cec85651dc8d)
<p>
On the client machine, open up Command Prompt and attempt to ping mainhost. Notice that it fails. Go to the domain controller, and open up DNS Manager. Navigate to the Forward Lockup Zones and create a A-record for mainhost. In the line of IP address, point to the domain controller private IP address. Now go back and ping mainhost in the client machine and observe it works. You can see it works and is stored in the DNS cache by typing ipconfig /displaydns.
</p>
<p>
Note: DNS works by checking its dns cache first, then the host file and then the internet. You can test this by changing the A-record to a different IP address and then pinging mainhost again. It pings the old address as it's still in the cache and you would need to do ipconfig /flushdns with permissions to clear it. Then, it should update and use the right address.
</p>
<br />


![13](https://github.com/user-attachments/assets/fc059613-b551-4c26-9cea-9b5c4c8873f1)
<p>
In the domain controller and in DNS, create a CNAME record, name it search and point it to google.com. Then, in the client machine ping search. Note it works. You can do nslookup search and observe it shows google as well as their ip addresses.
</p>
<br />


![14](https://github.com/user-attachments/assets/f20d5074-b038-4d0d-b58e-cd514d961446)
<p>
In the domain controller, go to the file explorer and create 4 folders in the C drive: read-access, write-access, no-access and accounting. Right click read-access, click properties, navigate to Sharing, click share and add domain users with permissions of only read. Do the same with the others folders except accounting: write-access with domain users -> read/write and no-access domain admins -> read/write. 
</p>
<p>

Go to the client computer and navigate to the shared folders by typing \\domain controller name\. Try to access all the folders created before and notice the differences. Read-access allows you to view but, can't create files. Write-access allows you to view and create. No-access doesn't allow you view it at all. 
</p>
<br />



![15](https://github.com/user-attachments/assets/e79ecfc3-7e9a-4e12-bb37-537797bd03fe)

<p>
Go back to the domain controller and open up ADUC and create another Organization unit named _GROUPS. Create a security group named ACCOUNTANTS and apply it to one of the users before by right clicking, click Member of and adding accountants in the list. Back to the file explorer, share the folder to the accountants group with read/write permissions. Log in to that user or sign out and log back in, navigate back to network directory and notice the accountants folder is there. You can then access it. 
</p>
<br />

