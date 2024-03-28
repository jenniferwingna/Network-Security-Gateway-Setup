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
- [Firewall Setup](#firewall-setup)
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
## DNS Configuration
## Routing Setup
## Firewall Setup
## Testing
