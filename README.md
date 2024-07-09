## CybStSvpn
site-to-stie vpn with DMZ. Please refer the `topology image` for the architecture used.

### Before jumping to site-to-site vpn:
1. **Install GNS3**: Ensure GNS3 is installed and running on the specified server and listening on 0.0.0.0 not on a localhost.
2. **GNS3 Templates**: Ensure that the specified templates are available in GNS3. i.e pfSense 2.6.0, Ubuntu Host Machines etc.

### Create a new project or import the existing project. 
- Go to file --> create new project. 
- OR import the existing gns3 project. 

### Configuration steps for Site-To-Site VPN with DMZ.
**Prerequisites**:
- Two pfSense firewalls (one at each site)
- Assign IP address (WAN, LAN and DMZ interfaces).

** Network Details**:  
- Site A (Server Network): 172.16.31.0/24 (DMZ interface).  
- Site B (CSIRT Network): 192.168.1.0/24  
- Allowed Services: HTTPS(TCP 443), FTP (TCP 21).  

**Steps**:  
1. Configure the DMZ Interface:  
- Login to pfSense Web Interface at Site A:  
  . Navigate to Interface > Assignments  
  . Add a new interface and assign it to the DMZ network.  
  . Go to Interfaces > DMZ Interface, enable it and set the IP address to `172.16.31.1/24`  
- Login to pfSense Web Interface at site B:  
  . Repeat the above steps for the CSIRT network with the LAN IP address `192.168.1.1/24`  

2. Set Up the site-to-site VPN:  
- VPN configuration on site A:  
  . Navigate to `VPN > IPSec`.  
  . Add a new Phase 1 entry:  
  	. `Key Exchange Version`: IKEv2  
   	. `Interface`:WAN  
   	. `Remote Gateway`:Public IP or hostname of Site B  
  	. `Authentication Method`: Mutual PSK  
  	. `Pre-Shared Key`: (Enter a strong pre-shared key)  
  	. `Encryption Algorithm`: AES (256 bit)  
  	. `Hash Algorithm`: SHA256  
  	. `DH Group`: 14  
  	. Save and apply the changes.  
   . Add a new Phase 2 entry:  
  	. `Mode`: Tunnel IPv4  
  	. `Local Network`: 172.16.31.0/24  
  	. `Remote Network`: 192.168.1.0/24  
  	. `Protocol`: ESP  
  	. `Encryption Algorithm`: AES (256 bit)  
  	. `Hash Algorithm`: SHA256  
  	. Save and apply the changes  

- VPN configuration on Site B:  
  . Navigate to `VPN > IPsec`.  
  . Add a new Phase 1 entry with the same settings as Site A but with the local and remote gateway reversed.  
  . Add a new Phase 2 entry with the same settings as Site A but with the local and remote networks reversed.  
  
3. Configure firewall Rules:  
- On Site A (DMZ interface):  
  . navigate to `Firewall > Rules > DMZ`  
  . Add a rule to allow HTTPS traffic.  
  	. `Action`: Pass  
  	. `Interface`: DMZ  
  	. `Protocol`: TCP  
  	. `Source`: DMZ Subnet  
  	. `Destination`: CSIRT Subnet  
  	. `Destination Port`: 443  
  	. Save and apply the rule.  
  . Add a rule to allow FTP traffic:  
  	. `Action`: Pass  
  	. `Interface`: DMZ  
  	. `Protocol`: TCP  
  	. `Source`: DMZ Subnet  
  	. `Destination`: CSIRT Subnet  
  	. `Destination Port`: 21  
  	. Save and apply the rule.  

- On Site B(CSIRT Interface):  
  . navigate to `Firewall > Rules > CSIRT`.  
  . Add similar rules as Site A but with the source and destination reversed.  

4. Configure NAT (if needed).  
NAT is not considered in this case, so NAT process is not included here.  

5. Test the VPN Connection.  
- From a machine on Site B, try to access HTTPS or FTP service on a machine in Site B.   

**Note**: Make sure to adjust any specific firewall rules as per your security policies and requirements.

