# Securing Network Traffic: Linux Gateway Setup and Firewall Configuration
## Content Page
- [Introduction](#introduction)
- [Objective](#objective)
- [Keywords](#keywords)
- [Content Page](#content-page)
- [VM Setup](#vm-setup)
- [Network Configuration](#network-configuration)
- [DNS Configuration](#dns-configuration)
- [Routing Setup](#routing-setup)
- [Testing](#testing)
## Introduction
This project aims to configure a Linux virtual machine (VM) to act as a gateway for another VM and enable traffic between these machines. Through the project, security controls will be implemented for traffic to and from the designated Windows VM. Important networking concepts such as routing, DNS, and firewalls will be covered during the practice.
<p align="center">
  <img src="https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/841f753c-b631-44ac-80e5-1bcc9576ddab" width="566">
  <br>
  Image 1: Simple illustration of network architecture
</p>

## Objective
1. **Establish a secure gateway**:  
   Set up a Linux VM as a secure gateway to route traffic from a Windows VM through the Linux system.
2. **Implement network controls**:  
   Configure IP forwarding, NAT, and firewall rules to regulate traffic flow between internal and external networks.
3. **Enhance DNS functionality**:  
   Install and configure BIND9 as a caching DNS server to optimize DNS resolution and caching.
## Keywords
  gateway, firewall, DNS, routing, iptables
## VM Setup
Two VM will be configured in this project: Linux VM as gateway and Windows VM as client or endpoint device. This setup enables the Linux VM to implement security controls just like a gateway for traffic to and from the Windows VM.
1. Linux VM - gateway:  
Configure this VM with two network interfaces in VirtualBox. Adapter 1 will be NAT and Adapter 2 will be an Internal Network. You can name Internet Network to whatever you want but make sure you use the same name in the WIndows VM.
<p align="center">
  <img width="486" alt="image" src="https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/d8c52998-8a5f-416b-b50a-47263985c223">

</p>

2. Windows VM - client:    
  Set up the network adapter and attach it to the Internal Network as the Linux VM.
  <p align="center">
    <img width="623" alt="image" src="https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/b6a25a9b-5455-4be1-be21-5f0a4b405059">

  </p>
  
## Network Configuration
### Configure Internal Network Interface on Linux VM
   1. Edit the Netplan Configuration file for the identified interface. We can first create the file by executing `$ sudo nano /etc/netplan/01-network-manager.yaml`
   2. Then, configure the internal network interface with a static IP address in the 10.0.100.0/24 CIDR range.  
  ```
network:
  version: 2
  ethernets:
    ensXX:    # Replace ensXX with your interface name
      addresses:
        - 10.0.100.5/24    # Choose an available IP address in the range
      routes:
        - to: 0.0.0.0/0
          via: 10.1.100.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]  # DNS servers
```
   3. Apply the netplan after configuration by `$ sudo netplan apply`
   4. Check if the internal network interface has a static IP address by `$ ifconfig`.  
      You should have something like this:
  <p align="center">
    <img src="https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/28cd8bf0-5d6c-4161-8268-82aa14e1661a" width="500">
  
  </p>


  5. Enable IP forwarding on Linux VM  
    `$ sudo sysctl -w net.ipv4.ip_forward=1`

### Configure Windows VM
- Set up the Ipv4 properties like the following:
  <p>
    <img width="366" alt="image" src="https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/6bc190e1-3d3b-4076-a6d8-7c60979f6bdc">
  </p>
- You need to choose a different IP address than the Linux VM but this static IP address needs to be within the same subnet as the Linux VM.  
- The default gateway should be the Linux VM's IP address.

### Verification
- Reboot both Linux VM and Windows VM to ensure network settings apply.
- For Linux VM：
  - use `curl` to check for network connection.
    ![image](https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/d78da880-4f48-4e2b-837c-7d72ab10e3c6)

  - try ssh into any available server from the VM
    ![image](https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/757a0be6-8570-47b6-8ef7-b9599c1e2a63)
  - Execute `$ifconfig` to see if the static IP address persists after reboot.
## DNS Configuration
1. Install BIND9 by `$ sudo apt-get install bind9`
2. Configure BING9 by `$sudo nano /etc/bind/named.conf.options` to specify Google DNS(8.8.8.8) as the forwarder.
  ![image](https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/86f0b563-a952-43d4-9ffd-6231e368a058)

3. Restart the BING9 service to apply the changes.
  `$sudo systemctl restart binf9`
4. Testing time !  
   - `dig` to test DNS resolution.  
   - Repeat the same command with same domain several times. You should observe caching behavior which the query time for latter query should be shorter.  
   - You should have something like this:
   ![image](https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/c00d079d-3df2-4e7d-8f64-687f96bf3dfa)

## Routing and firewall Setup
Traffic flow between these the gateway(Linux VM) and client(Windows VM) will follow these rules:
![image](https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/e638e57d-b743-4563-af64-44ea3bd84e7a)
1. Route traffic from the internal network to NAT ONLY when the destination port is SSH (10020/tcp), DNS (53/udp),  HTTPS (443/tcp). Drop all the other packets.
2. Route only RELATED and ESTABLISHED connections from NAT to the internal network.
3. Perform IP masquerading for all traffic leaving the NAT network (Hint: POSTROUTING chain in NAT)

We will implement NAT using `iptables`:
```
sudo apt update
sudo apt install iptables
sudo systemctl start iptables
```
Configure our `rules.v4` files:
```
sudo nano /etc/iptables.rules.v4
```
Your rules.v4 should look like this:  
![image](https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/a50d1e1a-f2eb-40ac-a0d5-598c8626d6ed)  

Apply these rules to iptables by:
```
sudo iptables-restore < /etc/iptables/rules.v4
```
**You need to import the rule file every time after rebooting the system.
Now, you can check what rules are applied by `$ sudp iptables -L`:
![image](https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/7a8814c4-ed98-4ab4-b1dd-547f769ff0aa)

## Testing
To check if rules are set up successfully, we can allow traffic flow and capture packets using `tcpdump `:
```
sudo tcpdump -i <interface> -U -w - | tee <yourname>.pcap | tcpdump -r -
```
** Don't forget that you should be capturing traffic of internal interface instead of the NAT interface.

Let's go back to our Windows VM and do the testing!
1. Browser any HTTPS website. You should be able to visit the website under https. Relevant handshaking packets should be captured in your Linux VM. You can use `Wireshark` to check for packets. Pay attention to keywords like TCP and ACk.
2. Now try to browser any website under http. HTTP traffic should be blocked. Connections fail.
   ![image](https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/5be37644-cb34-4489-91e9-6f461480418f)

3. Try `ping` any webpage. Your pings should time out. Since ICMP packets are blocked. You should have outputs like this:
  ![image](https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/24817705-8a20-4cb7-8d04-a3a62f845d1a)

4. Try SSH any available server and it should connect from the client(Windows VM). You should have something like this:
   ![image](https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/a66462ff-a513-4d1b-8f32-1dc1bcae4945)
    What appears in terminal of your Linux VM will be similar to this with the hand-shaking process:
   ![image](https://github.com/jenniferwingna/Network-Security-Gateway-Setup/assets/116328799/9251d9fe-fc33-4154-98d0-47fd2c70a4d9)

   
