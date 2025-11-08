###### 1. Stealth 
- sudo nmap -sS 
- sudo nmap -sS -f 
- sudo nmap -sS --mtu 8
###### 2. Decoy
- sudo nmap -sS -D <decoy_ip_1>,<decoy_ip_2>,ME,<decoy_ip_3> <target_ip>
###### 3. Slow
- sudo nmap -sS --scan-delay 100ms <target_ip>
- very slow: sudo nmap -sS -T0 <target_ip>
###### 4. Alternate Flags
- Send a packet with no flag: sudo nmap -sN <target_ip>
- sudo nmap -sF <target_ip>
- sudo nmap -sX <target_ip>