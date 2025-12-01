Types of Network Topology
   - Broadcast: All recipients
   - Unicast: One Recipient
   - Multicast: 2 or more recipients
- **Topology**
  - Describes the layout of a network
  - Lines out the path data communication takes
  - Allows admins to see how devices are connected on the network
  1. Physical (Blueprint)
       - Shows how devices are connected
       - Describes the shape of the network tasks
       - Allows admins to see how the physical media or cable connects the devices together
    1. Bus: 
       - Easy and cheap to implement
       - Hard to troubleshoot and small fault tolerance
     ![[Pasted image 20251018190028.png]]
    1. Star 
     - Easy to troubleshoot and scale
     - Higher cost and a single point of failure (the hub)
     ![[Pasted image 20251018190504.png]]
    3. Ring
     - Stronger transmition
     - Longer transmition time
     - Can be uni or bidirectional
     ![[Pasted image 20251018191207.png]]
    4. Mesh
     - Most physical connections per device {n(n–1)/2}
     - High fault tolerance
     - Complex to manage
    ![[Pasted image 20251018191714.png]]
    5. Tree
     ![[how-tree-topology-works-d6f5ca7d199c5cfe29dd4d35ed669871.png]]
    6. Hybrid
     ![[Pasted image 20251018192506.png]]
  2. Logical
       - Describes pathway data will take regardless of the physical connection
       - Allows admins to troubleshoot problems with comms in transit by seeing the path data takes
       - Possibly very different from physical topology