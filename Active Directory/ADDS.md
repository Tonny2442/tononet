# Active Directory Setup

I have an AD setup with OU's sorting all my Group Policies, Computers and Users. I usually start it off like this:
```
Tononet.local -> Managed -> Computers
                         -> Users -> Admin  
                                  -> Standard
                                  -> Service
```
This allows me to sort them based on their purpose. It also makes it easy to keep group policies organized. I have my Computer Policies put under Computers, with User policies put under the target user group to apply to. I put all my computer policies under a main Startup Policy under the Computers OU.

When connecting with a Deployment Server, you would want to make it so your computers join under the Computers OU to make it so your policies sync with the newly imaged system. This keeps you from having user interaction in an automated deployment, making it more effiecent and most importantly, you don't have to pay someone to do it for you. 

ADDS Supports fallback domain controllers which sync the SYSVOL and NETLOGON folders of each domain controller. In the case of the primary DC going offline, the fallback can take over, handling authentication requests and changes to group policies. This is not a permenant solution, however, if you don't transfer the FSMO roles in the case of a permenant shutdown of the primary, it could lead into a hassle. The best case there is to attempt to backup your policies and remake the domain. You would never want to do this in production, meaning you need to rejoin thousands of computers. In that case, you would transfer the FSMO roles first before doing a reimage of the DC. 

To minimize the attack surface of the domain, it is highly recommended to make a special user for handling joining of domain that you can use in automated deployments. This user will only have the permissions to join and create a computer on the domain, nothing else. Not even logging in.

Active Directories let you sync settings and even files across computers. You can specify the user folder to be stored on a seperate drive than %SYSTEMROOT% such as an external HDD you carry around, or even a network-mapped drive. This also lets you login with only one user and password, rather than two. Microsoft accounts though have mainly solved this issue too.

# How would I do this?

Firstly, I would setup a virtual machine. This lets you restore snapshots in the case of a failure or mistake, and they do happen. You can use bare metal hardware, but this is usually for more demanding domains that get logged into/out of hundreds of times a second. Usually for large enterprises. I setup the VM with 2 cores, 2GB memory, 32GB HDD space and a single gigabit link. Graphics cards are not required, you most of the time will be remoting into the server or simply using the Windows Remote Server Administration Tools, especially if you choose the GUI-less option.

I would next install the virtual machine using Desktop Experience. This makes it easier to configure a new domain controller. If you are using the eval, you have 180 days until the server will restart every hour. You can rearm the activation up to 6 times. Non-evals will act as unactivated Windows versions and do not restart every hour.

The first thing to do is to get the normal drivers and updates installed. Getting the system up to date driver and cumulative update wise is good to get out of the way before deploying the system to be in use. After that, you want to open Server Manager and configure the local system, such as changing network settings, disabling IE Advanced Security and setting a name. You also want to set a static IP on the system itself or your DHCP server to make sure the clients can always contact the DC.

After that, you want to install ADDS and the DNS server. This is required to have links (tononet.local) point to the IP (````10.0.0.10````). After the roles are installed you will see a message in the flag at the top right asking you if you want to promote it to a domain controller. Click that, make a new forest and specify the forest FQDN (````name.ext, or example.com````) Enter in a recovery password (in the case the system fails to boot) and then create the forest. The system will restart afterwards and you will want to login under ````DOMAIN\Administrator```` and your previous set password. After that, you are in the domain. 

You then want to make sure your DNS is set correctly. Right click the Network icon, Open Internet and Network settings, Change Adapter Options, right click your adapter and hit properties. Click TCP/IPv4, and hit properties. Use the following DNS server addresses, and have the first be the IP of the DC, and the second being your fallback DC or your fallback DNS in the case the DNS server stops working, such as ````8.8.8.8```` or ````1.1.1.1````. 

# Setting up the tree

The next part we fall into is the actual configuration of the tree. You have two MMC entries you want to open, ````Active Directory Users and Computers```` & ````Group Policy Management````. 

In ADUaC, open your domain tree, right click your domain, new, Organizational Unit. Name this one Managed and hit OK. Under this, make two new organizational units. ````Computers & Users````. Under Users, make three folders. ````Admin , Standard , Service````. Under Domain\Users, right click and copy Administrator. Give it your preferred name and logon name. Then hit next, set a password, and hit finish. Do CTRL+X, then open your Managed\Users\Admin OU and hit CTRL+V. That is how you make Administrator users. Then we are done with this MMC snap-in. 

Next open the GPM snap-in. Open the tree you just created and you will see all the OUs you just created. Create your first group policy by right clicking the parent OU and click ````Create a GPO in this Domain, and Link it here````. You then will name your GPU, usually in the Computers OU: ````Computers - Startup Policy````. Hit OK. Then you see the child object. Right click it and hit edit. 

The GPMEditor will open. This is how you specify what settings this specific GPO will apply. We will get into this part of it another time. You can now close out all the MMC Windows you have opened. We are done with the server side config.

# Joining a domain

Joining a domain is easy. Using the same way we set the DNS on the DC, set the client's DNS to your server IP. Second DNS can be the fallback server or a custom DNS.

### Pre-20H2

Go to run, type Control System and hit enter. Click Change Settings, Change, and then click the radio button named Domain, enter in the FQDN (````example.com````) and then hit OK. Specify your user (````DOMAIN\User````) and the password. You should see "Welcome to the (domain) Domain.". If you see that, you are now joined to the domain. After a restart, sign in with your user. 

### Post-20H2

MSFT changed where you get to a dialog. Open Settings, System, About. Then click Rename this PC (advanced) and then you will know where you are and follow from that point to the end of the Pre-20H2 guide.

# Domain join errors (common)

Sometimes things just don't work. If you get an error mentioning DNS, try checking if your DNS is set to the DC. If it is, restart the client and try again.

# No internet on the domain!

This is common. Go onto the DC Server, and open the DNS MMC. Click the computer name and open Forwarders. Click Edit and make sure you have your wanted DNS servers in there. Once the DNS queries get past ADDS, it goes through to those specified DNS servers. In most cases, it will be your gateway, but you can set it to another domain controller (on fallback DNSes) or one of your choice. 
