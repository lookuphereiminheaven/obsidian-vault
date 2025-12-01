**The Open System Interconnection - OSI**
![[photo_5935954814267673230_w(1).jpg]]
- Layer 1 (Physical)
  1. Analog modem
  2. Hub
    - Similar to repeater, only receives traffic from a port, and sends to all ports without looking at data traffic
- Layer 2 (Data link)
  1. Switch
    - Looks at the MAC address of each packet of data forwarding to unique nodes
  2. Wireless Access Point - WAP
    - Connects wireless devices to the network
    - Connects wireless network to wired network
    - SSID - Service Set Identifier (32-bit Alphanumeric string)
- Layer 3 (Network)
  1. Multilayer switch
     Uses ASIC chip to pass data to non-local network devices
  2. Router
    - Can be a single dedicated device, incorporated in a multifunction device or as a software on a device or node that has two NICs
    - Can only be used with routable protocols
    - Can be a hardware or software
- Security devices
  1. Firewall
    - Can be placed on routers or hosts or can be its own device
    -  Functions at layers 2,3,4,7
    - Blocks packets entering or leaving network via
      1. Stateless inspection
         Examines every packet
      2. Stateful inspection
         Only examines state of connection between two networks
  2. Intrusion Detection System - IDS
     Passive system designed to inform of breaches
     - Signature based
     - Anomaly based
     - Policy based
     May be deployed at host level
  3. Intrusion Prevention System - IPS
     Active system designed to stop a breach
     - Best placed between router and destination network segment
     - Block attack IP
     - Close vulnerable interface
     - Terminate network session
     - Redirect attack
  4. VPN
     - Functions at multiple layers 2,3,7
     - Outside of internet most VPNs function at layer 3 network providing IPsec encryption through a secure tunnel
- Optimization devices
  1. Load balancer
    - Content switch or content filter
    - Used to load balance between multiple hosts with the same data spreading the workload for efficiency
    - Used to distribute requests to server farm helping to ensure no server gets overloaded
  2. Proxy server
     - Requests resources on behalf of the client machine
     - Often used to retrieve resource from outside untrusted networks
     - Hides and protects requesting client
     - Can increase network performance by caching commonly requested Web pages