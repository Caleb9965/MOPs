# Basic Setup
## Backup Configuration (Device-Setup-Operations)
Navigate to Device->Setup->Operations
1. Save named configuration snapshot
	1. candidate config
2. Export Named Configuration
	2. Candidate Config
3. Save this somewhere you can find it. you can import this if a rollback is needed.
Alternatively, you can just verify current configuration version
		1. Load Configuration version
		2. Verify Highest number and most recent date.
		3. Roll back to this number after any changes completed.
## Zones (Network-Zones)
For the most control, we need to configure a zone for our GlobalProtect. This is *optional* however **HIGHLY** recommended.
![[Pasted image 20250414093701.png]]
## Certificates (Device-Certificate Management-Certificates)
Okay we need to generate two certificates. One is our root cert, and one will be our GlobalProtect Specific.
### Root Certification Generation
![[Pasted image 20250414094339.png]]
### GP (GlobalProtect) Certification Generation
You can have multiple IPs in the "SAN" field 
![[Pasted image 20250414094511.png]]
### Certificate Profile (Device-Certificate Management-Certificate Profile)
Next up we will create a certificate profile, as shown below. We're referencing our Root Cert, and leaving everything else blank
![[Pasted image 20250414095111.png]]
### SSL/TLS Service Profile (Device-Certificate Management-SSL/TLS Service Profile)
Configure this section as shown in the picture, reference the GPCert. and set the min/max versions.
![[Pasted image 20250414095542.png]]
## Portal Configuration
Configure as shown in the pictures below.
### General
![[Pasted image 20250414100259.png]]
### Authentication
Make sure we choose the Service and Certificate profiles then add a client authentication
![[Pasted image 20250414101454.png]]
For this we will reference our authentication profile tied to active directory that was previously made
![[Pasted image 20250414101557.png]]
Please note: The bottom box has to be set to YES in order to skip the future "device configuration" section. This will, however, make our network less secure.
### Agent
You can Skip the Portal Data Collection tab for now. Here in Agent, we need to do two things:
1. Configure an Agent
2. Add the root and have it install in local root store
	As you can see, you just add the trusted root ca, select RootCert and then check the box
![[Pasted image 20250414102017.png]]
Now for the agent, go ahead and add
- Name: Arbitrary, I used Client-Config-1
- Client certificate - local - pick your GPCert
- Everything else can be left alone on this tab
![[Pasted image 20250414102958.png]]
Jump to the "External" tabs and add
![[Pasted image 20250414103046.png]]
Here you would put your FQDN or IP, however you want the portal reachable. I used my outside Interfaces IP of 10.2.191.10 with a source region of any.
![[Pasted image 20250414103139.png]]
At this point you can go ahead and click okay until you reach the main page and commit. you are done configuring the portal.
## Before we get started with the gateway we need to configure a few things.
### IPSec Crypto Policy (network-network profiles-GlobalProtect IPSec Crypto)
 - Add a new profile
- Select aes-256-gcm as this is the best option
- Name the Profile (I named it GP)
![[Pasted image 20250415103541.png]]
### Tunnel interface (Network-Interfaces-Tunnel)
We need to create two tunnel interfaces, one for the GlobalProtect base and one for the satellite. Create and name accordingly
- Assign to VR-1 virtual router
- Place both into the GP security zone
- Tunnel 2 only
	- IPv4 address of 192.168.1.244
![[Pasted image 20250415103821.png]]
## Gateway
### General
We're going to pick our interface, select ipv4 only and then use the IP on this interface, keep everything else the same.
![[Pasted image 20250415102959.png]]
### Authentication
We're going to once again select our SSL/TLS Service profile, our standard Certificate profile, and and setup our authentication as was done in the previous step.
![[Pasted image 20250415103207.png]]
### Agent
We're going to go ahead and check "tunnel mode" and choose our tunnel interface 1 (Or whichever one you created for the GlobalProtect base). Make sure we select the IPSec profile we created and enable IPSec is checked. Then move on to "Client Settings"
![[Pasted image 20250415103931.png]]
For the Client Settings, add an any-any profile. All you have to do is name it.
![[Pasted image 20250424164826.png]]
For Client IP Pools, we need to create a pool of IPs for the devices, these will be added to the firewalls routing tables but remember: ***if you place them on their own network any other routers will need to add a static route or learn through dynamic routing!***
- For this example, I used some available IPs in my internal network
![[Pasted image 20250415104136.png]]
Finally, in the network services tab, add your primary and secondary DNS so your clients can resolve!
![[Pasted image 20250415105357.png]]
### Satellite
Although we are not using the satellite functionality at this time, it still is required to setup here. Pick our interface we setup for the satellite (tunnel.2), configure a range of IPs to use, and keep everything else the same for now.
![[Pasted image 20250415144631.png]]
![[Pasted image 20250415144642.png]]
## Click OK and commit all changes. You are now ready for the PC side configuration
# PC configuration
**This section isn't needed if deploying GlobalProtect through Group policy**
Due to the way we set up this deployment, upon joining the portal your PC will download the certificates once authenticated by user. 
1. Navigate to the IP you set for your portal (10.2.191.10)
2. You should see the below screen
![[Pasted image 20250417101225.png]]
3. Login using your Active Directory Credentials, which will bring you to the below screen
![[Pasted image 20250417101425.png]]
4. Download the corresponding agent to your OS
	1. You do not have to change anything so just next through the installer and let it install.
5. Click get started
![[Pasted image 20250417101752.png]]
6. Enter Portal address (10.2.191.10) and connect
![[Pasted image 20250417101836.png]]
7. It will ask for your credentials once again, enter them and hit connect.
![[Pasted image 20250417101918.png]]
### It will now show you connected to the portal.
![[Pasted image 20250417101956.png]]
# Verifications
1. For our first verification we can refer to the above picture. If it shows connected then we are working.
2. We can also look in the logs located at *Monitor-Logs-GlobalProtect*
![[Pasted image 20250417102155.png]]
We're looking specifically for the log showing "calehamm" (or your user) and "gateway auth" showing successful
3. Finally we can navigate to *ACC-Network Activity*
	1. This will show us the traffic being sent over our user.