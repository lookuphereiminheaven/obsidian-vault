Dynamic Host Configuration Protocol
- Computers get its IP config from a DHCP server
- It tells the PC where the default gateway was and how to find a DNS server
- IP is received in 2 ways
  1. Static
     - Admin assigns IP and subnet mask to each host in the network
     - Admin assigns a default gateway location and DNS server location to each host
     - Each time a change is made IP must be updated 
  2. Dynamic
     - Admin configures a DHCP server to handle the assigning process automatically
     - DHCP server listens on specific port for IP info request and then responds
- DHCP process
  - Upon boot up the PC requiring IP sends a DHCP ***discovery packet**
  - Discovery packet is sent to the broadcast address: 255.255.255.255:67 (UDP port 67)
  - DHCP server receives discovery packet and responds with an ***offer packet**
  - Offer packet is sent to the MAC of the PC using port 68
  - PC receives the offer packet and returns a ***request packet*** (requesting proper IP) to the DHCP server
  - Once the DHCP server receives the request packet it sends back an ***acknowledgement packet*** which contains the IP config
  - Upon receipt of the acknowledgment packet the PC changes its IP config to reflect the info received
- Components of DHCP
  - Leases
    - Configs are only good for a specified amount of time
    - Configured by admins
  - Options
    - Default gateway location
    - DNS
    - Time server addresses
- DHCP isnt required to reside on local network segment
Domain Name Service -DNS
- A hierarchical very ordered structure
- Requires FQDN (Fully Qualified Domain Name ) to function	![[Pasted image 20251019181648.png]]
- Different levels of DNS servers
  - Local
    - The server on the local net that has HOSTS file that maps the FQDN to IP addresses in the local subdomain
  - Top Level Domain - TLD
    - Contains records of top level domain
    - .COM .ORG .EDU .GOV .MIL .INT
    - Do delegate down to second level servers to ease the load
  - Root server
    Server containing records of TLD servers
  - Authoritative
    - Responds to a request that has been specifically configured to contain info
    - Comes from a DNS server that holds the original record
  - Non-authoritative
    - Responds to requests with DNS info that it received from another DNS server
    - Not a response from the official name server for the domain. Instead it's a second or third-hand response
- DNS records
  - A
    Maps hostnames to their IPv4 
  - AAAA
    Maps hostnames to their IPv6
  - CNAME
    Maps canonical (alias )names to hostnames
  - PTR
    Pointer record that points to a canonical name
  - MX
    Maps to the specific domain email server determining how emails travel from sender to recipient
- Dynamic DNS - DDNS
  - Allows lightweight and immediate updates to a local DNS database. Useful when FQDN remains the same but IP changes
  - An additional service to DNS
  - DDNS updating
    - Method of updating without an admin
    - Software that will monitor IP and once changed sends an update to the proper DNS server
    - Useful when accessing a domain whose IP is dynamically supplied by ISP