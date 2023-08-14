<p align="center">
<img src=https://i.imgur.com/KoHYr6Z.png/>
</p>

<h1>Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines</h1>

[Network Security Groups (NSGs) and Inspecting Network Protocols](https://github.com/ethanblomquist/azure-network-protocols) is a prerequisite to this tutorial. We will continue to use the Domain Controller and Client to experiment with DNS <br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- DNS
- Active Directory

<h2>Operating Systems Used </h2>

- Windows 10 (22H2)
- Windows Server 2022

<h2>A-Record Exercise</h2>
<h4>A record: A type of DNS record that maps a domain name to the IP address of the computer hosting that domain. The "A" in "A record" stands for "address".</h4>
<p>
First we need to make both DC-1 and Client-1 are up and running on Azure, then we need to access both with RDP. Within the command prompt we can try to ping a random domain, i.e. "mainframe". It will tell us "Ping request could not find host mainframe. Please check the name and try again." The picture below shows what is going wrong. When Client-1 asks DC-1 for the IP Address for "mainframe", nothing happens bcause DC-1 has no records of "mainframe".
</p>
<p>
<img src=https://i.imgur.com/kJIjZAl.png/>
</p>

<p>
On DC-1, navigate to Server Manager -> Tools -> DNS -> Forward Lookup Zones -> Right click mydomain.com -> New Host -> Name: mainframe -> IP address: Same as DC-1 -> Add Host -> Return to Client-1 -> Open Command Prompt -> Ping "mainframe" -> nslookup "mainframe" -> As you can see, we've created a new record tied to 10.0.0.4.
</p>
<p>
<img src=https://i.imgur.com/lGPlyWv.png/>
</p>

<p>
Next we can enter ipconfig /displaydns to see which records are now located on the local cache of Client-1. Client-1 no longer needs to use DC-1 as a DNS server to access 'mainframe.
</p>
<p>
<img src=https://i.imgur.com/ifZXdwb.png/>
</p>

<p>
Return to DC-1 and edit the IP address of "mainframe" to 8.8.8.8 -> Return to Client-1 and try pinging "mainframe again. -> You will observe that it is still pinging mainframe's previous IP (10.0.0.4). This is because Client-1 still has 1.0.0.4 cached as mainframe's IP. Let's enter the ipconfig /flushdns to remove all of the cached DNS information Client-1 has stored. Make sure to open the command prompt as an administrator. As you can see if we run ipconfig/displaydns, there is no cached DNS info.
</p>
<p>
<img src=https://i.imgur.com/iMEYUGY.png/>
</p>

<p>
Now if we ping mainframe, it will successfully connect. Entering the ipconfig/disply DNS now shows mainframe and the DNS server on DC-1.
</p>
<p>
<img src=https://i.imgur.com/ypivtND.png/>
</p>

<h2>CNAME Exercise</h2>
<h4>Canonical Name (CNAME): A type of Domain Name System (DNS) database record that indicates that a domain name is the nickname or alias for another domain name.</h4>
<p>
On the DNS Manager withinn Active Directory on DC-1, we can right click mydomain.com and create a New Alias (CNAME) -> Alias name: search -> FQDN: www.google.com -> ping "search" on Client-1. As you can see the "search" CNAME successfully resolved to www.google.com
</p>
<p>
<img src=https://i.imgur.com/Js0d7NX.png/>
</p>

<h2>Root Hints Exercise</h2>
<h4>Root hints are a list of DNS servers on the internet that a DNS server can use to resolve queries for names that it does not know.</h4>
<p>
Using Client-1 we can access any website we want using a web browser. For example, if we open onto www.youtube.com, we are able to access the home page and watch videos. Now if we were to run ipconfig/display dns we would see many more ecords then before/  If our DNS server on DC-1 does not have a record of the YouTube's domain, how can it still access the site? Go to the DNS Manager on DC-1. Right click DC-1 -> Properties -> Root Hints. Here you can see all the root servers DC-1 is using to access sites not explicitly listed under mydomain.com.
</p>
<p>
<img src=https://i.imgur.com/hZmxZZc.png/>
</p>

<h2>Network File Shares and Permissions</h2>
<h4>You can share folders over the netowrok and contol which users within Active Directory may access those files and folders. This </h4>
<h3>Step 1: Log into Client-1</h3>
<p>
Open the Active Directory Users and Computers EMPLOYEES OU -> Find a random user and copy their display name, I chose pimom.qisix -> Sign in to Client-1 with as this user.
</p>
<p>
<img src=https://i.imgur.com/H8lmpcD.png/>
</p>


<h3>Step 2: Create Folders on C:\ Drive</h3>
<p>
On Dc-1, create 4 folders on C:\ “read-access”, “write-access”, “no-access” and “accounting”. To edit the permissions of these folders, right click -> Properties -> Sharing -> Enter desired group to share with -> Select Permission Level -> Share. Set the Permissions as follows:</p>

  - “read-access”, Group: “Domain Users”, Permission: “Read”
  - “write-access”,  Group: “Domain Users”, Permissions: “Read/Write”
  - “no-access”, Group: “Domain Admins”, “Permissions: “Read/Write”

<p>
<img src=https://i.imgur.com/FPl0Wk0.png/>
</p>

<h3>Step 3: Check Permissions</h3>
<p>
On Client-1, enter "\\dc-1" into the navigation bar of Windows Explorer. You will see the 3 folders we shared to the network. Check the permissions are set correctly. As a standard user, you should be able to open and create files in the write-access folder, only be able to open the read-access folder and not even be able to open the no-access folder at all. Create a text file on DC-1 in read-access. If you open on Client-1 that file you can read it, but if you try to overwrite it, Windows will not allow you to do that.
</p>
<p>
<img src=https://i.imgur.com/CTHHuFk.png/>
</p>

<h3>Step 4: Create a Security Group</h3>
<p>
On the DC-1 Active Directory Users and Computers window, create a new OU called SECURITY_GROUPS -> Right Click -> New -> Group -> ACCOUNTANTS
</p>
<p>
<img src=https://i.imgur.com/hMQ5c7N.png/>
</p>
<p>
Navigate back to the "accounting" folder -> Properties -> Sharing -> Share... -> ACCOUNTANTS -> Add -> Read/Write -> Return to Client-1 -> Check access to accounting folder -> Access should be denied -> Return to DC-1 -> Active Directory Users and Computers -> mydomain.com -> ACCOUNTANTS -> Right Click -> Properties -> Members -> Add -> Enter user's name (pimom.qisix) -> Check Names -> OK -> Apply -> Log off of Client-1 and re-login with the same user -> Enter \\dc-1 to Explorer -> Open accounting folder. If you successfully accessed the folder the permissions have been configured correctly.
</p>
<p>
<img src=https://i.imgur.com/GCAFlgF.png/>
</p>

<h2>Tutuorial Complete</h2>
<p>
Hopefully after this guide you are more comfortable with both Active Directory and DNS. Both concepts are vital to the smooth functioning of an organizations netowrks.
</p>
