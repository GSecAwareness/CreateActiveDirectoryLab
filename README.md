# CreateActiveDirectoryLab
We will create a mini corporate network with these instructions using Server 2019 and one Windows 10 client to verify access. Both VMs were created with Oracle. 

1)	Run through the setup of the VM for Server 2019
2)	Make sure to configure two network adapters on the VM
3)	One adapter for NAT and one adapter for internal

## Set up Addressing

1)	Click on connection
2)	Click on Network (connected)
3)	Change adapter settings
4)	Right click on Ethernet, Status, Details: to see ip address; label this connection appropriately
5)	Set IP address to Internal; Connection -> Network -> Change Adapter settings -> properties -> Ipv4 
6)	Set static Ipv4 address as 

## Rename PC

1)	Right-click Start Menu -> Settings -> Under About, Choose “Rename PC” and rename DC
2)	Restart

## Install AD

1)	Add roles and features
2)	Click next until Select Destination Server, click next
3)	Choose Active Directory Domain Services
4)	Click Next and then Install
5)	Once the role is installed, close the diagloue box 
6)	Click on the flag in the top right (yellow flag) and choose post deployment configuration
7)	Promote this server to a domain controller
8)	Add a Forest and name the Root domain name: “mydomain.com”, click next
9)	Under Domain Controller Options, enter a Password and confirm it
10)	Next past DNS, next, and then install -> sign out

## Create New Admin account 

1)	Start -> Windows Administrative Tools -> Users and Computers 
2)	Right-click and MyDomain and create a new Organizational Unit to put the Admin account in
3)	Name “Admins” and Uncheck “Protect container from deletion”
4)	Create a new user inside of the new Admins OU with password, uncheck expirations
5)	To make the account a domain admin, right-click and choose properties
6)	Choose Member of and Add  “Domain Admins” and Check the name to verify
7)	Sign out and log in with new admin account

## Install RAS and NAT – allow client to be in private network but access the internet through the Domain Controller

1)	Add Roles and Features, Next until choice “Remote Access”
2)	Next until “Routing” under Role Service, next and install
3)	Close the Window
4)	Go to Tools at the top right and choose Routing and Remote Access
5)	Right-click on DC (local) and choose Configure and Enable
6)	Choose NAT and finish; if not successful, just do over
7)	Choose the correct interface to connect to the internet (INTERNET) and Finish

## DHCP – assigning IP address to our hosts

1)	Add roles and select DHCP, finish
2)	Tools and select DHCP to create scope
3)	Right-click on Ipv4 and choose New Scope
4)	Name the Scope 172.16.0.100-200
5)	Fill in Scope and configure
6)	Add DC IP address into Default Gateway
7)	Add Domain Name (default) and IP address of DNS server (might have to change to DC address), no WINS server
8)	Authorize DHCP server and refresh

## Configure the DC to access the internet (not recommended)

1)	Click on Configure this local server
2)	Disable IE Enhanced Security Configuration
3)	Put Powershell  script and download and put on desktop
4)	Open Names in PS folder and add my name
5)	Open PS ISE and run as admin
6)	Open script from desktop folder
7)	Before Running the script, enable the execution of scripts 
8)	Set-ExecutionPolicy Unrestricted -> Yes to All
9)	Cd c:\users\a-mgraham\desktop\AD_PS-master to change to the correct directory
10)	Click play and it should run the script
11)	Once it completes, go to AD Users and Computers -> -USERS
12)	Find Users, Contacts, and Groups -> Search for your name

## Create Windows 10 Pro VM

1)	Go through Oracle Configuration to set up Win 10 image, named Client1
2)	Go to Advanced and enable Shared Clipboard amd Drag/Drop
3)	Go to Network and change Adapter to Internal Network (this will pull IP from DHCP on DC)
4)	Start VM and run through setup
5)	Create local account User
6)	Check ip config and verify its pulling an IP address from the default gateway from DC
7)	Right click on Start and choose settings
8)	
## Rename Windows PC and join domain

1)	Right-click on Start and choose System
2)	Choose Rename this PC (Advanced)
3)	Choose Rename this PC (Advanced), not the regular Rename this PC
4)	Click on Change (To rename this computer or change its domain or workgroup)
5)	Rename PC to Client1 and switch from Workgroup to Domain
6)	In the blank field, type mydomain.com and click OK
7)	Login with credentials and restart
8)	Login with credentials on Windows client1 (other user)

## Verify Client1 on DC

1)	Go to DC ->  Server Manager -> Tools -> DHCP
2)	Go to Scope -> Address Leases 
3)	Inside there should be one lease (the lease assigned to Client1)
## Verify Client1 in AD Users and Computers

1)	Click Start -> Windows Administrative Tools -> AD Users and Computers
2)	In mydomain.com -> go to the Computers container and verify Client1

## PowerShell  Script Explained

 
![getcontent](https://github.com/GSecAwareness/CreateActiveDirectoryLab/blob/main/Get-content%20from%20Names%20text%20file.PNG)

*PASSWORD_FOR_USERS: Password1* 		

This sets this as the password for all users we create

*USER_FIRST_LAST_LIST =  Get-Content .\names.txt*

This gets the first and last name from names.txt

*$password = ConvertTo-SecureString $PASSWORD_FOR_USERS –AsPLainText –Force*	

This takes the plain text password and converts it into the $password object

*New-ADOrganzaitonalUnit –Name _USERS –ProtectedFromAccidentalDeletion $false* 	

This creates a container called _USERS under Active Directory Users and Computers (mydomain.com) 	and unchecks “Protect container from Accidental Deletion”

![foreachloop](https://github.com/GSecAwareness/CreateActiveDirectoryLab/blob/main/for%20each%20loop.PNG)
   
*Foreach ($n in $USER_FIRST_LAST_LIST) {}*	

This creates a loop where $n is the current user being examined to receiving input; the loop runs for each name in the list 

*$first = $n.Split(“ “)[0].ToLower()* 	

This creates a split from the space; takes the first element (0) and stores it in the variable $first

*last - $n.Split(“ “)[1].ToLower()*

This splits from the space; takes the second element (1) and stores it in the variable $last

*username = “$($first.Substring(0,1))$($last)”.ToLower()*

This manipulates the first and last name to make one name; it takes the first character of the first name and attaches it to the last name 

Example: John Smith = jsmith

*Write-Host “Creating user: $($username)” –BackgroundColor Black –ForegroundColor Yellow*

This creates output to the screen “Creating user:" 
 
![createsuser](https://github.com/GSecAwareness/CreateActiveDirectoryLab/blob/main/Creates%20new%20user%20in%20AD.PNG)

This creates a new user and gives password “Password1”; password never expires; and put in the OU _USERS; it will pull the 1000 users and put them in the _USERS container
