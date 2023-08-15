<p align="center">
<img src=https://i.imgur.com/KoHYr6Z.png/>
</p>

<h1>Digging Deeper into Networking and Active Directory</h1>

[Network Security Groups (NSGs) and Inspecting Network Protocols](https://github.com/ethanblomquist/azure-network-protocols) is a prerequisite to this tutorial. We will continue to use the Domain Controller and Client to experiment with networking concepts.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)
- Active Directory

<h2>Operating Systems Used </h2>

- Windows 10 (22H2)
- Windows Server 2022
- Ubuntu Server 20.04

<h2>Wireshark</h2>

On Client-1 [download](https://www.wireshark.org/download.html) and install Wireshark. This is used to analyze our network activity. You will have to enter the credentials of our previously created admin in order to install the program. Default installation settings are sufficient. Open Wireshark.
<p>
<img src=https://i.imgur.com/W8Bwyzy.png/>
</p>

<h2>ICMP</h2>
<h4>Internet Control Message Protocol (ICMP): A network layer protocol used by network devices to diagnose network communication issues.</h4>
<p>
Select the blue shark fin icon to start analyzing the network traffic. You will see a constant stream of packets being sent and received. Let's filter by the icmp protocol. Type "icmp" in the filter bar at the top of the window. Using the command prompt, ping DC-1's private IP. You should be able to see the traffic between the two VMs. 
</p>
<p>
<img src=https://i.imgur.com/71RcKMQ.png/>
</p>
<p>
Next, open the Azure portal and use the search bar to find the Network Security Groups page -> Select DC-1 -> Inbound Security Rules -> Add -> Protocol: ICMP -> Action: Deny -> Set a priority higher then other rules -> Name: DENY_ICMP_PING_FROM_ANYWHERE -> you should be able to see the traffic stop on Wireshark and the pings start to time out. -> Return to the newly created rule and Deny ICMP traffic -> The pings should start to go through again.
</p>
<p>
<img src=https://i.imgur.com/XRhqXTn.png/>
</p>

<h2>SSH</h2>
<h4>SSH (Secure Shell): A network protocol that allows users to securely access a computer over a network. SSH is primarily used for remote access to servers and devices.</h4>
<p>
For this experiment we will mix it up and create an Ubuntu Linux VM. Go to Virtual Machines in Azure -> Create -> Resource Group: Active Directory -> Name: LinuxServer -> Authentication type: Password -> Username: jane_admin -> All other settings can be left to default -> On Client-1, enter ssh (Because SSH uses TCP port 22, we can also enter tcp.port == 22) into the Wirseshark filter -> To access LinuxServer, enter the following command: ssh + username@VMs private IP -> My command: ssh jane_admin@10.0.0.6 -> yes -> Enter password (must be manually typed) -> We now have command line access to the Linux VM. We can enter in some commands to see the traffic in Wireshark.
</p>
<p>
<img src=https://i.imgur.com/ZouHzOV.png/>
</p>
<p>
<img src=https://i.imgur.com/ZRR6qUh.png/>
</p>

<h2>DHCP</h2>
<h4>Dynamic Host Configuration Protocol (DHCP): A network protocol that automatically assigns IP addresses and other configuration information to devices connected to a network.</h4>
<p>
Next we can filter for DHCP traffic. -> Enter the command ipconfig/renew -> There should be traffic of Client-1 requesting a refresh of it's IP address
</p>
<p>
<img src=https://i.imgur.com/XYykpLI.png/>
</p>

<h2>DNS</h2>
<h4>Domain Name System (DNS): A naming database that translates domain names into Internet Protocol (IP) addresses.</h4>
<p>
Next we can filter for DNS traffic. -> Enter nslookup www.google.com -> This will check the IP Addresses Google uses 
</p>
<p>
<img src=https://i.imgur.com/Bhb5Wsq.png/>
</p>

<h2>RDP</h2>
<h4>Remote Desktop Protocol (RDP): A secure network communications protocol that allows users to access and control a computer remotely.</h4>
<p>
Next we can filter for RDP traffic. You should see ALOT of traffic since we are currently using RDP to access Client-1.
</p>
<p>
<img src=https://i.imgur.com/9j3pzl0.png/>
</p>

<h2>A-Records</h2>
<h4>A record: A type of DNS record that maps a domain name to the IP address of the computer hosting that domain. The "A" in "A record" stands for "address".</h4>
<p>
First we need to make both DC-1 and Client-1 are up and running on Azure, then we need to access both with RDP. Within the command prompt we can try to ping a random domain, i.e. "mainframe". It will tell us "Ping request could not find host mainframe. Please check the name and try again." The picture below shows what is going wrong. When Client-1 asks DC-1 for the IP Address for "mainframe", nothing happens because DC-1 has no records of "mainframe".
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
Return to DC-1 and edit the IP address of "mainframe" to 8.8.8.8 -> Return to Client-1 and try pinging "mainframe" again. -> You will observe that it is still pinging mainframe's previous IP (10.0.0.4). This is because Client-1 still has 1.0.0.4 cached as mainframe's IP. Let's enter the ipconfig /flushdns to remove all of the cached DNS information Client-1 has stored. Make sure to open the command prompt as an administrator. As you can see if we run ipconfig/displaydns, there is no cached DNS info.
</p>
<p>
<img src=https://i.imgur.com/iMEYUGY.png/>
</p>

<p>
Now if we ping mainframe, it will successfully connect. Entering ipconfig/displaydns now shows mainframe and the DNS server on DC-1.
</p>
<p>
<img src=https://i.imgur.com/ypivtND.png/>
</p>

<h2>CNAME</h2>
<h4>Canonical Name (CNAME): A type of Domain Name System (DNS) database record that indicates that a domain name is the nickname or alias for another domain name.</h4>
<p>
On the DNS Manager within Active Directory on DC-1, we can right click mydomain.com and create a New Alias (CNAME) -> Alias name: search -> FQDN: www.google.com -> ping "search" on Client-1. As you can see the "search" CNAME successfully resolved to www.google.com
</p>
<p>
<img src=https://i.imgur.com/Js0d7NX.png/>
</p>

<h2>Root Hints</h2>
<h4>Root hints are a list of DNS servers on the internet that a DNS server can use to resolve queries for names that it does not know.</h4>
<p>
Using Client-1 we can access any website we want using a web browser. For example, if we open onto www.youtube.com, we are able to access the home page and watch videos. Now if we were to run ipconfig/displaydns we would see many more records than before.  If our DNS server on DC-1 does not have a record of the YouTube's domain, how can it still access the site? Go to the DNS Manager on DC-1. Right click DC-1 -> Properties -> Root Hints. Here you can see all the root servers DC-1 is using to access sites not explicitly listed under mydomain.com.
</p>
<p>
<img src=https://i.imgur.com/hZmxZZc.png/>
</p>

<h2>Network File Shares and Permissions</h2>
<h4>You can share folders over the network and control which users within Active Directory may access those files and folders. This </h4>
<h3>Step 1: Log into Client-1</h3>
<p>
Open the Active Directory Users and Computers EMPLOYEES OU -> Find a random user and copy their display name, I chose pimom.qisix -> Sign in to Client-1 with this user.
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

<p align="center">
<img src=https://i.imgur.com/MiAXxi0.png/>
</p>

<h2>Cleaning up Azure Resources</h2>
<p>
To conclude this project, I would recommend deleting the Vitrual Machines and Resource Groups we've vreated in Azure. First navigate to Virtual machines -> Select All -> Delete -> Type "yes" for confirmation. -> Resource Groups -> Active_Directory -> Delete resource group -> Enter resource group name to confirm deletion
</p>

<h2>BONUS: Running a VPN within Azure</h2>

A VPN, or Virtual Private Network, is a technology that allows you to create a secure and encrypted connection over the internet. It acts as a secure tunnel between your device and a server located in a different location, encrypting your internet traffic and hiding your IP address. This helps protect your online privacy and security by shielding your data from potential hackers, ISPs (Internet Service Providers), and other third parties who may try to monitor or intercept your internet activities. A  VPN can be used to access company resources located on private networks without being onsite. Let's have aa little fun experimenting how a VPN interacts with IP.

<h3>Three Different IP Addresses on one Physical Machine</h3>
<p>
Let's start by checking our local PC's IP Address. We can do this by opening https://whatismyipaddress.com/ . We can see my local IP address(I left out the Host ID), located in Minneapolis, MN.
</p>
<img src=https://i.imgur.com/jQ8ru8g.png/>

<p>
Next we can create a new virtual machine within Azure. I used a Windows 10 VM with 2 vcpus and 16 GiB of RAM. We can choose to locate our VM in many different parts of the world. I chose the Norway East (Zone 1) location.
</p>
<img src=https://i.imgur.com/te6nsxe.png/>

<p>
Using Remote Desktop Connection on Windows we can use the VM's public IP to access the VM's desktop.
</p>
<img src=https://i.imgur.com/aFLiwKo.png/>

<p>
Using the same site as above, we can check the VM's IP Address. Interestingly, we can see the location matches what we set our VM server to. It is located in Oslo, Norway and the ISP is listed as Microsoft since we are using Azure. We can also confirm the IP of 51.120.241.218 matches what we entered into Remote Desktop Connection.
</p>
<img src=https://i.imgur.com/b7cg9zr.png/>

<p>
Now we can download and install Proton VPN on the virtualized desktop. Once installed, open the program and choose a server to connect to. I personally chose Japan. The remote connection to the VM may disconnect temporarily.
</p>
<img src=https://i.imgur.com/nlSFAI9.png/>

<p>
Once again we can check https://whatismyipaddress.com/ to confirm the VM's IP. Now with the VPN running we are connected to Tokyo, Japan.
</p>
<img src=https://i.imgur.com/PS8vjLT.png/>

<h3>Project Complete</h3>
I hope you learned something and enjoyed the process! As recommended above, delete the virtual machine and resource group.

<h2>Tutorial Complete</h2>
<p>
Hopefully after this guide you are more comfortable with both Active Directory and a variety of networking concepts! If you have any questions or suggestions on how I can improve this guide, please feel free to contact me at: efblomquist@gmail.com
</p>
