# `Active Directory`` `
### installing
1. tools -> computer management -> Local Users and Groups -> Administator -> Set Password
1. ncpa.cpl -> static ip (on your network range) -> default gateway -> (DNS and Active Directory are depend on each other) you should set it (DNS and active directory should be on the same service) _it's better to be on one system (if you've alreardy installed DNS service on another system , set your DNS server related to that system ip unless set your  system IP Adress as Preferred DNS Server ** it installs DNS server on its own**_
1. Add rules and features -> change hostname of your system ⇒ (this pc right click / settings + pause break -> rename this PC -> restart your computer (at least have 4 GB Ram to run windows server)
1. on server manager -> Add rules and features _don't forget to check your IP to avoid confliction or unwanted changes _
1. next -> add Active Directory Domain Services check -> next * 2 -> install
1. on the top of the window, a popup notif comes up and recommend to promote your server to a domain controller, so click on that and a window comes up named (Deployment Configuration) -> click on add a new forest(if you've not already set a domain) -> ** Make sure your Domain name be a proper name (avoid internet domain i.e .org || .com ...) you can name it related to your city or .local...** _ it's really hard to change it later_
7. next -> don't chane anything here and just set a recovery password to access NDTS(Database)  after problem Active Directory -> set it complex enough 7 digit **don't forget it!**
1. next -> your NetBIOS name set your domain name except your .domain
1. next -> don't change your default path, ** to set the policy of your client saves inside SYSVOL Directory (really important folder)  ->***** click install***
1. _when a server converts to controller  it makes the windows servcer slower_
1. it restarts automatically and remove local group and users and put them inside NTDS database
1. ** check something after your system has been rebooted:**
	1. computer management -> check is there local users and groups or not: if not it works.
	1. active directories and users _if executes and shows the users it works_
	1. execute run command and type your domain hostname or ip : two folder (NETLOGON and SYSVOL) should comes up : if yes it's ok if not don't progress and check the problem or reinstall Domain controller
	1. go to DNS section and check it executes or not (inside that, check the ip and domain name)

	## how to goin clients in 3 ways
	* login on domain server Administrator
	* ** check all of the client should be on the same IP range**
	* click on windows and pause button -> change settings -> click on change button
	* go to server manager and click on tools -> Active Directory Users and Computer -> click on users -> right click to assign new user and add it **but it's 10 default to join domain**
	* go to computer management and set password for administrator and rename it 'cause of security cases
	* _ inside VMware workstation we have snapshot to save the state to reverse back when faced a problem_

	**Command line**
		```powershell
		Add-Computer -DomainName "DomainName" -restart
		```
		1. Adding Computers Remotely to a Domain  -> _ _***you can use it to remotely add user to domain via windows server  by Powershell***
		```powershell
		Add-Computer -ComputerName "computername" -LocalCredential "computername"\"username" -DomainName "domainname" -Credential "domain"\"domainadminuser" -Restart
		```
### DNS

**Round Robin : **for example you want to offer 4 webservers to  a offer a website.
on your Administrator Properties on advanced tab enable round robin ⇒ **webservers IP are in the same range**
when i user request to dns server it set the website dns to first ip, after that second ip and ...

**Netmask Ordering**
suppose someone want to call your webserver whom has the same range of my server, so it redirect to that server .
suppose we have another webserver on different IP range which its information is available on our dns, it first validate which IP range does the requester have; if has external range it redirect the user to external webserver if has local, redirects it  to our local webserver. it prevents long loading to send requests ** the Netmask Ordering has priority option to change the webservers weight to set the request traffic on the webserver (local and externals).**

***Name Resolution Systems ***_⇒ name to ip convertersion process by DNS server _
**FQDN (FUlly Qualified Domain Naming / DNS Name ⇒ shows the authority of the the system i.e SRV1.google.com ⇒ com > google > SRV1 ⇒ **SRV1 is domain inside google and the google inside com domain. FQDN contains two segment -> Hostname - DNS Suffix ⇒ in above example : SRV1 is **Hostname** and next till end are **Suffix**. 

**Zone -> **DataBase of DNS Server. When a client search about hostname on network, it goes to the DNS server and the DNS Server first checks its cache. if it was existed  ok unless goes and seek inside its zones to find data about that hostname if it wasn't inside the zones neither it goes on forwarders or root hints (they are as the same as our DNS Server).
**Root Hint -> ** They are DNS Server which saves every existed domain names data.
**Forwarders -> **They are default not set and should set manually; when your DNS Server doesn't have the data about your search hostname, we assume that, some specific DNS Server which called Forwarders have that specific data about our hostname. * if the Forwardes haven't set yet or the data wasn't inside them , so our DNS Server ask about that data on Root Hint.
![](/home/cipher/.local/share/AppFlowy/data/images/539a476f-a2e7-4313-9b42-90014b870341.png)

after finding the answer, it stores the data inside its cache to make the process faster.
_when the hostname's ip changes, you should delete your DNS Server cache 'cause it has the hostname's previous ip so returns wrong answer._

### Active Directroy Types
1. Active Directory  Domain Service ( AD DS)
![](/home/cipher/.local/share/AppFlowy/data/images/8b63d11d-6893-4378-91c9-dae5a05b8e16.png)

===============================================
1. Active Directory Lightweight Directory Service ( AD LDS)
===============================================

1. Active Directory Certificate Service ( AD CS)
![](/home/cipher/.local/share/AppFlowy/data/images/6a9f3150-7a03-421b-9bc8-206263984d6c.png)
1. Active Directory Right Management Service (AD RMS)
![](/home/cipher/.local/share/AppFlowy/data/images/f2c5e8e6-2a9d-4e5d-adf4-2da4482f2dc0.png)
1. Active Directory Federation Service (AD FS): _ you can access into your service from outside via only port https(443) or http(80) and no need to open more ports to reduce security._
![](/home/cipher/.local/share/AppFlowy/data/images/141748c0-9ed9-40e8-b0fb-5c2e34f5efab.png)


### Domain
**Domain -> **limited teritury of our Domain controller.
**Forest -> **collection of one or more Domains with the same root. -> _the first Domain is the Forest Root._
**Child -> **subdomain of its parent. i.e shop.blacktie.local ⇒ shop < blacktie.local
**Tree -> ** another Domain in the same level of our root domain, can also have independent domain name. ** But all of them are inside a forest. **we call every of the a tree. but all of the tree have the same root `(first Tree Root)`. 

### How to list all expendable clients 
```powershell
$IncativeDays = 30
$InactiveDate = (GET-Date).AddDays(-$InactiveDays)
$ADAccounts = Get-ADComputer -Filter {LastLogonTimeStamp -lt $InactiveDate} -Properties LastLogonTimeStamp

foreach ($Account in $ADAccounts) {
    Write-Output "Computer Name: $($Account.Name)"
    Write-Output "Last Logon Time: $([DateTime]::FromFileTime($Account.LastLogonTimeStamp))"
}
```

### **How to Delete expendable computers**
for one user:
```powershell
Remove-ADComputer -Identity CL-2 -Confirm:$false 
```

for list of users:
1. first make a list name of the computers
```powershell
$computers  Get-Content -Path "C:\computers.txt"
foreach ($computer in $computers) {
    Remove-ADComputer -identity $computer -Confirm:false
}
```
** Notice that first you should use it on administrator and don't forget to check the user names on .txt list.**
