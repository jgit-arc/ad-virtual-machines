<p align="center">
<img src="https://i.imgur.com/M3rj4Ua.png" />
</p>

# Virtual Machines with Mircosoft Azure

Active Directory is used to centrally manage multiple user accounts in a single place (passwords, permissions, etc.). It is commonly used to manage resources on a large scale. This walkthrough will utilize Active Directory within a Windows virtual machine using Microsoft Azure. This will allow us to simulate creating and working with multiple users. Note: The focus will be on Windows Active Directory, not Azure Entra ID (which is the Azure version of Active Directory). This is not a comprehensive Azure guide and assumes basic Azure familiarity.

---

<h2>Environments and Technologies Used</h2>

- Microsoft Azure
- Azure Subscription
- Remote Desktop

---

## 1. Setup Domain Controller in Azure

<p>
<img src="https://i.imgur.com/43JRoir.png"/>
</p>

Two virtual machines will be used: one to run Windows Server (DC-1), which will act as the domain controller, and a second that will act as the client (client-1). Both will have virtual network interface cards. By default, the VMs will point to Azure's specific DNS server. In order for client-1 to join the domain, we want the DNS to be set to the domain controller (which will act as a DNS server).

Log in to Azure at portal.azure.com and sign in to your Azure account. We will use resource groups to manage the virtual machines. Go to Resource Groups. Click Create. Select the subscription you want to use. Name the resource group ad-lab. Select your local region. Click Review + Create. Click Create.

Go to Virtual Networks in Azure. Click Create. Select the subscription you want to use. Select the recently created ad-lab resource group. Enter AD-Vnet for the network name. Place the network in the same region you selected for the resource group. Click Review + Create. Click Create. Next, we will create two virtual machines.

Go to Virtual Machines. Click Create. Select the subscription you want to use. Select the recently created ad-lab resource group. Name the VM DC-1. Place the VM in the same region you selected for the virtual network. Under Image, select Windows Server 2022. Under Size, select a VM with at least 2 vCPUs. Enter a username and password. For the sake of the walkthrough, I am using 'defaultuser'. If you use something else, do not forget it! Under Licensing, check the box referring to licensing and the confirmation box. Click the Networking tab. Under Virtual network, select the recently created virtual network. Click Review + Create. Click Create.

Go to Virtual Machines. Click Create. Select the recently created ad-lab resource group. Name the VM client-1. Place the VM in the same region you selected for the resource group. Under Image, select Windows 10 Pro. Under Size, select a VM with at least 2 vCPUs. Enter a username and password. For the sake of the walkthrough, I am using 'defaultuser'. If you use something else, do not forget it! Under Licensing, check the box referring to licensing and the confirmation box. Click Networking. Under Virtual network, select the recently created virtual network. Click Review + Create. Click Create.

<p>
<img src="https://i.imgur.com/t7p7DBa.png"/>
</p>

We must set the Domain Controller's (DC-1) NIC private IP address to static. Go back to Virtual Machines and select DC-1. Select Networking. Select Network Settings. Click Network Interface/IP Configuration. Click ipconfig1. Under Private IP Address Settings, change it to Static and click Save.

Return to Virtual Machines. Select DC-1 and copy its public IP address. Open Remote Desktop and connect to DC-1. To verify that you are on the correct machine, Windows Server should open automatically as soon as the VM starts. If it does not, go to Start and enter About. Next to Edition, it should read Windows Server 2022. Click Start, then click Run. Enter wf.msc and click Windows Defender Firewall Properties. Set the firewall state to Off in the Domain Profile, Private Profile, and Public Profile. Click Apply and OK.

---

## 2. Setup Client-1 in Azure

<p>
<img src="https://i.imgur.com/cy10qxl.png" />
</p>

Return to Virtual Machines. Select DC-1 and copy its private IP address. Select client-1. Go to Network Settings. Click Network Interface/IP Configuration. Click DNS servers. Click Custom. Paste the private IP address from DC-1. Click Save.

Go back to Virtual Machines. Select the checkbox next to client-1 and click Restart. Click client-1 and copy its public IP address. Open Remote Desktop. Add a separate PC (we'll need to use client-1 and DC-1 simultaneously) and connect to client-1.

<p>
<img src="https://i.imgur.com/GiITZwN.png"/>
</p>

Return to Virtual Machines. Select DC-1 and copy its private IP address. In client-1's virtual machine, open PowerShell. Enter ping <DC-1's private IP address>. The output should look identical to the screenshot above. If not, the VMs may be on different virtual networks or DC-1's Windows firewall may not have been turned off.

In PowerShell, enter ipconfig /all. Next to DNS Servers, you should see DC-1's private IP address, indicating that everything is working.

---

## 3. Join Client-1 to domain


<p>
<img src="ttps://i.imgur.com/2rDpqWs.png"/>
<img src="https://i.imgur.com/YLJXcI3.png"/>
<img src="https://i.imgur.com/qm7ejGz.png"/>
</p>

We will now install Active Directory on the domain controller, create an admin, and join client-1 to the domain. This will allow users (which we will create shortly) on the domain to log in to client-1. If you're not already connected, remote desktop into DC-1. Click Start and enter Server Manager.
Click Add roles and features. Click Next until you get to Select Destination Server. There should only be a single server (DC-1). Click Next. In Select Server Roles, check Active Directory Domain Services and click Add Features. Click Next until you get to Confirm Installation Selections. Check Restart the Destination Server Automatically if required and click Yes. Click Install. A successful installation should look like the screenshot above. Click Close.


<p>
<img src="https://i.imgur.com/BtEHKHr.png"/>
<img src="https://i.imgur.com/IknVVoG.png"/>
</p>

DC-1 must be promoted to an actual domain controller (within the virtual machine itself, not Azure). From the Server Manager homepage (screenshot above), click the yellow flag at the top right. Click Promote this Server to a Domain Controller. Click Add a New Forest. Enter a root domain name. I chose mydomain.com for the walkthrough. Click Next.
On the Domain Controller Options screen, leave all settings at their default values and enter a simple password (we will not use this) for the Directory Services Restore Mode Password. Click Next until you reach DNS Options. Uncheck Create DNS Delegation. Click Next through the Prerequisites Check, and then click Install. The VM will be restarted automatically. 


## 4. Create domain admin user

Now that the DC-1 VM is a domain controller, users must specify which domain they want to log in to. There are two different contexts: accounts local to client-1 (local to that specific computer) or a domain account. To log back into DC-1, change the username to reflect the domain account. It should look like this:

Username: mydomain.com<yourusername> (note the appropriate backslash)

<p>
<img src="https://i.imgur.com/NVw8Zcq.png"/>
</p>

After you've signed in, the Group Policy will update. This is expected. Once the process is finished, click Start and select Windows Administrative Tools. Open Active Directory Users and Computers. Click mydomain.com, then click Users. To the right, you will see the account you used to sign in (the primary domain admin account). Right-click mydomain.com, select New, and click Organizational Unit. Enter _EMPLOYEES and click OK. Repeat the same process to create another organizational unit. Enter _ADMINS and click OK.

Note: Make sure the names are entered exactly (same capitalization, same underscores) as shown for the sake of following the walkthrough.

We will now create a new user. Right-click the _ADMINS folder, select New, and click User. Enter the following fields:

- First name: Jane
- Last Name: Doe
- User logon name: jane_admin


<p>
<img src="https://i.imgur.com/gM8Nc1j.png"/>
<img src="https://i.imgur.com/WLQJZNv.png"/>
</p>


Click Next. Enter a password you will remember. For the sake of the walkthrough, deselect "User must change password at next logon" and check "Password never expires." Click Next, then click Finish.

The jane_admin account needs to be added to the Domain Admins security group before it has administrator privileges. Right-click the Jane Doe account and select Properties. Click the "Member Of" tab. Click Add. In the "Select Groups" box, enter Domain Admins and click Check Names. Click OK, then click Apply. The account is now a domain admin and can create users, reset passwords, etc.

Log out and sign back in as mydomain.com\jane_admin. This is the account you will use to administer changes for the remainder of the walkthrough.

## 5. Join client-1 to the domain (mydomain.com)

<p>
<img src="https://i.imgur.com/taHeUKV.png"/>
<img src="https://i.imgur.com/qXEoa7m.png"/>
</p>

Sign back in to DC-1 as the domain account we just created (mydomain.com\jane_admin). Leave this VM open, as we will need to alternate between the two. Log into client-1 using your original account (defaultuser).

On client-1, right-click Start, click System, and then click "Rename this PC (advanced)." Under the Computer Name tab, click Change. Under "Member of," select Domain and enter mydomain.com. Click OK.

Because we already changed the DNS settings to use DC-1's private address, the VM is able to detect the domain controller for the mydomain.com domain. Enter the Jane Admin account username and password. You will be prompted to restart the VM.

We will now verify that client-1 is part of the domain.

<p>
<img src="https://i.imgur.com/2BKjsOa.png"/>
</p>

Return to DC-1. Click Start and open Active Directory Users and Computers. Click mydomain.com, then click Computers. You will see client-1. Right-click mydomain.com, select New, and click Organizational Unit. Enter _CLIENTS and click OK.

Click the Computers folder and drag client-1 into the _CLIENTS folder.


## 6. Setup remote desktop for non-admin users on client-1

<p>
<img src="https://i.imgur.com/925vcT8.png"/>
</p>


Log in to client-1 as mydomain.com\jane_admin. Right-click Start and click System. Click Remote Desktop. Under User Accounts, click Select Users that Can Remotely Access This PC. Click Add. Enter Domain Admins, click Check Names, and click OK twice.


## 7. Create a bunch of additional users and test login with a created user


<p>
<img src="https://i.imgur.com/HeoQzz2.png"/>
<img src="https://i.imgur.com/eNMIWUw.png"/>
</p>

Log in to DC-1 as your domain admin (e.g., jane_admin). Click Start, enter PowerShell. Right-click PowerShell and select Run as Administrator. Click the New File icon in the top left. Open <a href="https://traff.co/0vv9hqIp">this link</a>, copy the text, and paste it into PowerShell. Click Save As, name the file create-users, and save it anywhere on the VM.

Note: This script generates a large list of random users for us to use as accounts. The specifics of PowerShell or the script itself are beyond the scope of this walkthrough. It simply needs to be executed to achieve the desired result. However, notice that on line 43 of the script, there is a reference to _EMPLOYEES. That is the same _EMPLOYEES folder we created earlier.

Click the Run Script button. This will take a while to complete, but you can still check Active Directory to see the users that have been created. In Active Directory, right-click _EMPLOYEES and select Refresh. As we established earlier, all of these profiles are able to log in as domain users. We can test this. Pick a random profile (make note of the name if necessary). Log out of client-1 (keep DC-1 open) if it is still open. Sign in as the profile you selected (be sure to use the mydomain.com context) and use the password "Password1" (created by the script). Once the account successfully signs in, log back out.

(Optional) - You do not have to let the script continue. Once you've tested one or two accounts to verify that they are able to sign in, you can manually stop the script in PowerShell.

## 8. Account Lockouts

<p>
<img src="https://i.imgur.com/Qu17I20.png"/>
<img src="https://i.imgur.com/7dTpfAc.png"/>
</p>

We want to be able to manage accounts when they have been locked out or when users have forgotten their passwords. This must be configured in the Group Policy Management Console. In DC-1, click Start. Click Run. Enter gpmc.msc and click OK. Right-click Default Domain Policy and click Edit. Expand Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Account Lockout Policy. There are three primary settings we can configure:

Account Lockout Duration
Account Lockout Threshold
Reset Account Lockout Counter After



<p>
<img src="https://i.imgur.com/DID0SLf.png"/>
<img src="https://i.imgur.com/548SvjO.png"/>
</p>

Click Account Lockout Duration. Click Define this Policy Setting. Set the time to 30 minutes. Click Apply and OK. This will automatically adjust the other options: the lockout threshold is 5 attempts, and the reset counter is 10 minutes. We need to force these changes on client-1. Sign into client-1 as the domain account Jane Admin. Click Start. Enter CMD and run CMD as an administrator. In the command prompt, type gpupdate /force and press Enter. This will update the policy. Now, when someone fails a login after a certain number of attempts (set to 5), the account will be locked. In the CMD, enter gpresult /r. Under Applied Group Policy Objects, you will see that the Default Domain Policy is active. Log out of client-1.


<p>
<img src="https://i.imgur.com/er2KkzP.png"/>
</p>


Try to log into client-1 with a random user (I recommend using the same user from earlier), but this time, enter the wrong password. Repeat this failed login 10 times. We want to purposely lock the account. Once Remote Desktop displays the lockout message, go back to DC-1, open Active Directory Users and Computers, and find the account you attempted to sign into. Double-click the account, select the Account tab, and you will see "Unlock Account. This account is currently locked out on this Active Directory Domain Controller." Check this box to unlock the account. Click Apply and OK. Log in to the random user account, this time using the correct password.


## 8. Enabling/Disabling Accounts

<p>
<img src="https://i.imgur.com/c5kxLHS.png"/>
</p>


Part of managing accounts includes enabling them (e.g., for new hires) or disabling them (e.g., if the account was compromised). In DC-1, in Active Directory Users and Computers, right-click a random user and select Disable Account. Notice the down arrow next to the icon, indicating the account has been disabled. Right-click the account again and select Enable Account to re-enable the account.

## 9. Observing Logs


<p>
<img src="https://i.imgur.com/7Xwm9bW.png"/>
<img src="https://i.imgur.com/NnDbnFN.png"/>
</p>

In DC-1, click Start. Open Event Viewer. Expand Windows Logs. Right-click Security, click Find, and enter the name of the client-1 account you used for the failed login attempts. This will bring up all the logs (not just the failed logins) related to that account. You can select each individual log to get a description of the circumstances surrounding the event.

Note: The following walkthroughs will use these virtual machines and configurations. You can turn them off, but do NOT delete them if you intend to continue.
