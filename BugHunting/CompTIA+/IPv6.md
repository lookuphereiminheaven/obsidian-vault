The answer to IPv4 running out 
- Works at layer 3 and its major focus is logical network and host addressing
- It's a 128-bit binary addressing scheme consisting of sets separated by a colon where each set is 2 bytes long
- For human readability it's converted to hex
- There are 2 to the power of 128 addresses available
- IPv6 local address structure
  - First 64 bits represent the local network and the last 64 bits the host
  - Local address is called ***link local address*** and always begins with **fe80**
- IPv6 global address structure
  - Host address is always the last 64 bits
  - The network portion is composed of routing prefix and subnet
  - They always begin in the range of 2000 to 3999
- IPv6 notation
  - Leading 0s in a set can be dropped
  - Any single set of consecutive 0s may be replaced by a double colon
- IPv6 network transmissions
  - Unicast
    - One to one comm
    - On the local network fe80 or on the global network 2000 to 3999
  - Multicast
    One to few comm
  - Always begins with ff
  - Anycast
    - One to the closest neighbor comm
    - Involves implementing DHCPv6
![[Pasted image 20251019203539.png]]
