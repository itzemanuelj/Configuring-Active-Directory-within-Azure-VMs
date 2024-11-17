<p align="center">
<img src=https://i.imgur.com/3iwtTkN.jpeg" height="40%" width="60%" alt="Logo"/>
</p>

# Configuring Active Directory within Azure VMs**


## Prerequisites and Installation
***
This tutorial outlines the prerequisites and installation steps for Configuring Active Directory within Azure VMs


## Environments and Technologies Used
***
- **Remote Desktop** 
- **PowerShell** 
- **Active Directory Domain Services**


## Operating Systems Used
***
- **Windows Server 2022**
- **Windows 10 Pro** 

## List of Prerequisites
***
- **Azure Subscription and Virtual Network**
- **Windows Server VM****
- **Static IP and DNS Configuration**


## Installation Steps
***
## Step 1: Set Up Virtual Machines in Azure
----

1. **Create DC-1**: Deploy a Windows Server 2022 VM in Azure.- If you need help creating your virtual machines, please see my tutorial [here](https://github.com/itzemanuelj/Creating-a-Virtual-Machine-Using-Azure)
2. Place it in a new Virtual Network .
3. In this tutorial we will name this Virtual Network VN1
4. **Create Client-1**: Deploy a Windows 10 VM in the same VNET as DC-1.
&nbsp;

![1](https://i.imgur.com/vZAElND.png)
![2](https://i.imgur.com/z4et6Mt.png)
![2](https://i.imgur.com/l6Mpwgn.png)

## Step 2: Configure DC-1 (Domain Controller)
----
### 1. Assign a Static Private IP:
- Go to the ****Azure portal****, open ****DC-1’s Networking settings****, and configure the private IP to be static.

### 2. Enable ICMP (Ping) on DC-1:
- Open a PowerShell window on DC-1 and run the following command:

  ```powershell
  New-NetFirewallRule -DisplayName "ICMPv4" -Protocol ICMPv4 -Direction Inbound -Action Allow
  ```
&nbsp;

![1](https://i.imgur.com/72kU3hm.png)
![2](https://i.imgur.com/8tT2uJA.png)


### 3: Install Active Directory Domain Services:
1. Log into DC-1 and open the ****Server Manager****.
2. Click ****Add Roles and Features****:
   - Choose ****Active Directory Domain Services****.
3. After installation, click the notification to ****Promote this server to a domain controller****:
   - Set up a new forest (in this tutorial we will name our domain controller `mydomain.com`) and restart DC-1.
&nbsp;

![1](https://i.imgur.com/mqir0sB.png)
![2](https://i.imgur.com/J5iSDZ5.png)

### 4. Log Back In:
- Log into DC-1 
&nbsp;

![2](https://i.imgur.com/7UkNnMf.png)



## Step 3: Configure Client-1
----

### Set the Client to Use DC-1 as DNS Server:
1. Open the ****Azure portal**** and navigate to ****Client-1’s Networking settings****.
2. Change the ****DNS server**** setting to the private IP address of ****DC-1****.
3. Restart ****Client-1****.
&nbsp;

![1]()
![2]()

### Join Client-1 to the Domain:
1. Log into ****Client-1**** and go to ****Settings > System > About****.
2. Under ****Rename this PC (advanced)****, click ****Change****:
   - Select ****Domain**** and enter `mydomain.com`.
3. Provide credentials for `mydomain.com\azureuser`.
4. Restart ****Client-1**** to apply the changes.
&nbsp;

![1]()
![2]()

# Step 4: Create Organizational Units and Users
---
## Log Into DC-1:
1. Open **Active Directory Users and Computers**.
2. Right-click your domain (e.g., `mydomain.com`) and create two Organizational Units (OUs):
   - **_EMPLOYEES**
   - **_ADMINS**

## Add a User:
1. Inside **_ADMINS**, right-click and create a new user:
   - **Name**: Jane Doe
   - **Username**: Jane_admin
2. Add **Jane_admin** to the **Domain Admins** security group.
&nbsp;

![1]()
![2]() 

# Step 5: Test Domain Functionality
----

## Remote Desktop for Regular Users:
1. Log into **Client-1** as `mydomain.com\JaneDoe`.
2. Open **System Properties > Remote Desktop**:
   - Allow access for **Domain Users**.
janedoe@mydomain.com
## Create Bulk Users with PowerShell:
1. On **DC-1**, run the following script in PowerShell to create test users:

   ```powershell
   for ($i=1; $i -le 1000; $i++) {
       New-ADUser -Name "User$i" -SamAccountName "user$i" -UserPrincipalName "user$i@mydomain.com" `
       -Path "OU=_EMPLOYEES,DC=mydomain,DC=com" `
       -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
       -Enabled $true
   }
   ```

2. Pick one user (e.g., `user500`) and test logging into **Client-1** using Remote Desktop.
&nbsp;

![1]()
![2]() 

# Conclusion

>Congratulations! 🎉 You’ve successfully deployed an on-premises-like Active Directory environment in Azure.  
---

- **DC-1** is managing the domain.  
- **Client-1** is a domain-joined machine.  
- We've set up OUs, created users, and verified Remote Desktop functionality.
