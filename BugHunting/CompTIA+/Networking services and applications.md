Virtual Private Network
- Used by remote hosts to access a private network through an encrypted tunnel through a public network
- Can provide cost reduction
1. Site to site VPN
   - Allows a remote site's network to connect to the main site's and be seen as a local segment
2. Remote access VPN (host to site)
   - Uses VPN client software to connect
   - Manages traffic from remote users
3. Host to host VPN (SSL)
   - Allows secure connection between two systems without VPN client software
   - Host seeking to connect uses SSL or TLS to connect
- Protocols used
  1. Internet Protocol Security (IPsec)
     - Works at layer 3 and above
     - Most common
     - Can be used with Authentication Header - AH protocol (only auth no encryption)
     - Can be used with Encapsulating Security Payload - ESP (both auth and encryption)
    - Both AH & ESP work in two modes
       1. Transport mode (between 2 devices)
       2. Tunnel mode (between 2 endpoints)
    - IPsec uses Internet Security Association and Key Management - ISAKMP
      - Provides a method for transferring security key and auth data between systems outside of security key generating process
  2. Generic Routing Encapsulation - GRE
     - Tunneling method capable of encapsulating a wide variety of network layer protocols
     - Often used to create a sub-tunnel within IPsec connection
     - Unlike IPsec which only transmits unicast packets, can do multicast and broadcast
  3. Point to point tunneling protocol - PPTP
     - Older VPN for dial-up
  4. Transport Layer Security - TLS
     - Cryptographic protocol used for encrypted connection between 2 end devices or applications
     - Uses asymmetrical cryptography to authenticate end points and then negotiates a symmetrical security key which is used to encrypt session
     - Has largely replaced SSL
     - Works on layer 5 and above
     - Most commonly used to create a SSL VPN
     - Supported by all modern browsers
  5. Secure Socket Layer - SSL
     - Like TLS but outdated
- Network access services
  1. Network Interface Controller - NIC
     - NIC is how a device connects to a network
     - At layer 2 determines which network protocol will be used (Ethernet or P2P)
     - At layer 1 determines how traffic will be converted a bit at a time into an electrical signal traversing the network media used
  2. Remote Authentication Dial In User Service - RADIUS
     - A remote access service used to auth remote users and grant access to authorized network resources
     - A popular AAA (Authentication, Authorization, Accounting)
     - Only the requester's pass is encrypted
  3. Terminal Access Controller Access-Control System Plus - TACACS+
     - Just like RADIUS but with all transmissions between devices encrypted
  4. Remote Access Service - RAS
     - A roadmap not a protocol
     - Software and hardware required for a remote access connection
  5. Web services
     - Provides means for comm between software packages or disparate platforms. 
     - Usually done by translating communication into an XML (Extensive Markup Language) format
  6. Unified voice service
     - Combination of software and hardware required to integrate voice comm channels into a network