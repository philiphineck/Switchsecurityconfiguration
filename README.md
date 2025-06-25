# Switch Security Configuration

Introduction
This lab activity provides a comprehensive review of Layer 2 security features in a network environment. It guides users through configuring essential network devices, including a Cisco 4221 router (R1) and Cisco Catalyst 2960 switches (S1 and S2). The lab focuses on a variety of configurations, starting with basic switch settings and progressing to more advanced security implementations. Key objectives include configuring VLANs, specifically **VLAN 10 for management**, **VLAN 333 as the native VLAN**, and **VLAN 999 as a "Parking Lot" for unused ports**. A significant portion of the lab is dedicated to switch security, covering **802.1Q Trunking**, **securing unused switchports, implementing port security, DHCP snooping, PortFast, and BPDU guard**. The lab ensures a practical understanding of securing network infrastructure by verifying end-to-end connectivity and troubleshooting as needed. 
# A Step by Step Walkthrough
# Part 1: Configure the Network Devices
Part 1 of this lab focuses on configuring the fundamental network devices. This involves physically cabling the network as depicted in the topology diagram. Following the physical setup, the objective guides the user through configuring Router R1 with a pre-defined script, including IP addressing, DHCP settings, and interface descriptions. Finally, it outlines the steps for configuring and verifying basic settings on both switches, S1 and S2, such as hostnames, preventing unwanted DNS lookups, and setting interface descriptions for active ports.
# Step 1: Cable the Network as shown in Figure 1, using a packet tracer. Ensure to initialize all the devices.
 
[Figure 1: Network Topology](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/My%20Topology.png)
 
[Figure 2: Initializing Devices](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/Initializing%20Device.png)
# Step 2: Configure R1 based on details in the addressing table. 
As given in [Figure 3](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/Basic%20Router%20Configuration.png), the R1 is configured with #hostname R1 for easy identification, IP address and subnet mask, and #no ip domain lookup to prevent the router from attempting to resolve mistyped commands such as domain name that can cause delays and consume network resources.   
The router is further configured to exclude IP addresses from being assigned by the DHCP server. The addresses excluded include the addresses of the router interface and those to be statically assigned to devices within the network. 

#ip dhcp excluded-address 192.168.10.1 192.168.10.9  
#ip dhcp excluded-address 192.168.10.201 192.168.10.202  
Since router R1 acts as a DHCP server, it has to know the DHCP pool and other network parameters such as the default router (default gateway), network address, and domain name.  
#ip dhcp pool Students (Defines the name of the dhcp pool)  
#network 192.168.10.0 255.255.255.0 (the network address of the dhcp pool)  
#default-router 192.168.10.1 (default gateway/router interface)  
#domain-name secure.com 
# Loopback Interface 
Any router has to have a loopback interface. This is a logical, virtual interface that is always up-state and never goes down (stable) unless it is administratively shut down. A loopback interface primarily offers reliable management access, ensures routing protocol stability, provides a consistent source address for router-originated services, and simulates networks for testing purposes. To configure:  
#interface loopback 0   	
#ip address 10.0.1.1 255.255.255.0

Router interfaces have to be configured too. As you noted in Figure 1, the arrows between S1 and R1 were red, suggesting that the router was in its initial state. Once configured, the arrows turned green, indicating that transmission of traffic between R1 and S1 is enabled. To further manage the traffic transmission, it is prudent to configure the interface g0/0/1 linking S1 and S2.   
Enter into the interface #interface g0/0/1  
Configure the IP address and subnet mask # ip address 192.168.10.1 255.255.255.0   
Describe # description Link to S1   
Configure the interface as a trusted source for DHCP relay information # ip dhcp relay information trusted  
Administratively enable the interface to bring it to upstate #no shutdown.   

It is good practice to configure a secure access and control console port used for administrative interaction. To configure:  
Enter the line console port #line con 0  
Prevent the console message, including the Syslog and debug output, from interrupting the command input being typed #logging synchronous  
Disable the automatic logout feature for the console session # exec-timeout 0 0. 

At this stage, these configurations are in the RAM as 'running configurations,' and if the device reboots, they will be lost. Thus, it is good practice to save the configurations to non-volatile memory (NVRAM) 

#copy running-config startup-config.   
To verify the configurations, the #show interface brief is needed. 
 
[Figure 3: Basic Router Configurations](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/Basic%20Router%20Configuration.png)
# Step 3: Configure and verify basic switch settings according to information in the addressing table. 
S1
#enable  
#configure terminal  
#hostname S1  
#no ip domain-lookup  
#ip default-gateway 192.168.10.1.   
Enter the interface configuration mode and configure interfaces  
#interface fa0/1  
#description Link to S2   
*Repeat for interfaces fa0/5 and fa0/6 *  
#no shutdown  
#copy running-config startup-config 
#show ip interface brief | include up  
[Figure 4: S1 Configurations](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/S1%20Configurations.png)
 
# S2
Repeat the steps in the S1 configuration in S2 and provide the appropriate details in the addressing table. As shown in Figure 5, it is possible to explore other commands that can lead to specified output, like pipe used with exclude only to show interfaces that are preferred #show ip interface brief | exclude down. Pipe command is often used with filtering expressions, including include, exclude, begin etc. Section command can be handy, too. 
 
[Figure 5: S2 Configurations](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/S2%20Configurations.png)
# Part 2: Configure VLANs on Switches
# Part 2 of this lab focuses on configuring VLANs on the switches S1 and S2. 
The primary objectives include creating and naming VLAN 10 as "Management," which will serve as the management VLAN for both switches. Additionally, this section guides the configuration of the Switched Virtual Interface (SVI) for VLAN 10, assigning IP addresses according to the Addressing Table. Furthermore, VLAN 333, named "Native," and VLAN 999, called "Parking Lot," will be configured on both switches, laying the groundwork for secure network segmentation.
# Step 1: Configure VLAN 10
S1. Enter the global configuration mode #enable, then #configure terminal. Enter the interface #interface vlan 10, and name the VLAN # name Management. 
# Step 2: Configure SVI for VLAN 10
Switch Virtual Interface (SVI) is a logical interface on a switch that serves as the Layer 3 representation of a VLAN. SVI enables remote management and inter-VLAN routing on Layer 3 switches. It is useful because it has IP connectivity for switch management, acting as a default gateway for VLANs and facilitating network segmentation and security. To configure the SVI for VLAN 10, give the IP address and subnet mask to provide IP connectivity for switch management.   
#interface vlan 10
#ip address 192.168.10.201 255.255.255.0  
#description Management SVI  
#no shutdown  
#end  
#copy run startup 
# Step 3: Configure VLAN 333 called Native
Native VLAN is often used to transmit untagged traffic on an 802.1Q trunk link. It serves as a default for all the untagged layer 2 frames without a specified VLAN ID.  
#interface vlan 333  
#name Native  
#description Default for Untagged Traffic
#end  
#copy run startup 
# Step 4: Configure VLAN 999
#interface vlan 999  
#name ParkingLot
#description Unused Ports Stay Here!
#end  
#copy run startup 
**Verify configuration
#show vlan brief. 
 
[Figure 6: S1 VLAN Configurations](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/S1%20and%20VLAN%20Configurations.png)
# S2
Repeat steps 1, 2,3 & 4 using the correct information in the addressing table. See Figure 7. 
 
[Figure 7: S2 VLAN Configuration] (https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/S2%20Trunk%20and%20DTP%20Configurations.png)
# Part 3: Configure Switch Security
# Part 3 focuses on configuring essential switch security features to fortify the network. 
This section details the implementation of 802.1Q Trunking on the link between S1 and S2, including setting a dedicated native VLAN. It also covers securing access ports by associating them with specific VLANs and turning off unused ports to prevent unauthorized access. Crucially, port security features are documented and applied, alongside enabling DHCP snooping, PortFast, and BPDU guard to create a robust and secure switch environment. This section has seven steps, and there are several other steps to follow in each step. 
# Step 1: Implement 802.1Q Trunking
A trunk is a logical link that carries traffic from multiple VLANs. It is often implemented using the 802.1Q encapsulation protocol and allows multiple VLANs to traverse a single physical link between the switches or between the switch and a router. In this activity, VLAN 333 is explicitly configured as the native VLAN for the trunk, and Dynamic Trunking Protocol (DTP) negotiation is turned off to ensure static trunking operation. 
# S1
Configure trunking on fa0/1, Disable Dynamic Trunking Protocol (DTP) negotiation, and verify the configurations (verify that Trunking is configured and DTP is disabled).   
#interface fa0/1  
#switchport mode trunk  
#switchport trunk Native vlan 333
#switchport nonegotiate  
#show interface trunk. [See Figure 8](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/S1%20Trunk%20Configuration.png). 
 
[Figure 8: S1 Trunk Configuratio](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/S1%20Trunk%20Configuration.png)
# S2
Repeat the same in S2. See Figure 9. 
 
[Figure 9: S2 Trunk and DTP Configurations](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/S2%20Trunk%20and%20DTP%20Configurations.png)

# Step 2: Configure Access Ports 
Access ports are switchport that connect directly to end devices, such as computers, printers, or IP phones, and belong to a single VLAN. These ports transmit frames untagged, as the end device is typically unaware of VLAN tagging, and the switch implicitly assigns the traffic to the configured access VLAN. In the topology given for this activity, access ports are fa0/5, fa0/6 (S1), and fa0/18 (S2) as they connect to the router, PC-A, and PC-B. 
# S1
Enter the interface (use range to configure multiple interfaces)   
#interface range fa0/5-6  
#switchport mode access  
#switchport access vlan 10   
Save configurations to NVRAM. [See Figure 10](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/S1%20Access%20Ports%20Configurations.png).
 
[Figure 10: S1 Access Ports Configurations](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/S1%20Access%20Ports%20Configurations.png)
S2. Repeat the same for S2. [See Figure 11](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/S2%20Trunk%20and%20DTP%20Configurations.png).
 
[Figure 11: S2 Access Port Configurations](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/S2%20Trunk%20and%20DTP%20Configurations.png)
# Step 3: Secure and Disable Unused Switchports
VLAN 1 is not generally secure. All the ports left unused and, in their default configuration, are active in VLAN 1. Also, suppose an actor has physical access to the switch whose ports have not been reconfigured. In that case, they will automatically land in VLAN1, suggesting that it provides an easy entry point for an attacker planning to exploit such vulnerability. To minimize the attack surface, it is prudent to move all the unused ports to a dedicated VLAN like 999 instead of leaving them in default VLAN 1. This is a critical security measure for any network administration and security professional seeking to design and implement secure and resilient network infrastructure. This practice isolates inactive ports from legitimate network traffic and management VLANs, significantly reducing the attack surface and mitigating potential unauthorized access or VLAN-hopping vulnerabilities.
Move the unused ports from VLAN 1 to VLAN 999, disable them, and verify that unused ports have been moved to VLAN 999 and are disabled. See Figure 12. 
# S1
#interface range fa0/2-4, fa0/7-24, g0/0/1-2
#switchport mode access
#switchport access vlan 999
#shutdown
#show interface status
 
Figure 12: S1 Unused Access Ports on VLAN 999 Configuration
# S2. 
Repeat the same for S2. See Figure 13.
 
Figure 13: S2 Unused Access Ports on VLAN 999 Configurations
# Step 4: Document and implement port security features
In this step, the focus is on securing access ports and implementing port security. In step 3, all the unused switchports are moved to VLAN 999 (the "Parking Lot" VLAN) and administratively disabled to prevent unauthorized access (Figure 12 and Figure 13). However, there are active access ports, including ports connected to end devices like PC-A and PC-B. In this case, specific port security features are configured, such as defining the maximum number of MAC addresses allowed, setting the violation mode, enabling sticky MAC address learning, and configuring ageing time for learned MAC addresses. 
First, verify the default port security configurations 
# S1
#show port-security interface fa0/6 to display the default port security settings. 
 
[Figure 14: Fa0/6 Port Security Status](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/Fa0_6%20Port%20Security%20Status.png)

[Figure 15: Critical Features for Fa0/6 Security status](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/Critical%20Features%20for%20Fa0_6%20Security%20status.png)
# S2
Repeat the procedures for S2. See Figures 16 and 17.
 
[Figure 16: Fa0/18 Port Security Status](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/Fa0_18%20Port%20Security%20Status.png)

[Figure 16: Critical Features for Fa0/18 Security status](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/Critical%20Features%20for%20Fa0_18%20Security%20Status.png)
Enable and configure port security
# S1
On fa0/6, enable port security with the given settings. Enter the interface and configure Max no. of MAC Address (3), Violation type (restrict), Aging time (60), and ageing type (Inactivity). Verify port security using #show port-security interface fa0/6. See Figure 18.
#interface fa0/6
#Switchport port-security
#Switchport port-security maximum 3
#Switchport port-security violation restrict
#Switchport port-security aging time 60
#Switchport port-security aging type inactivity
End
#show port-security fa0/6
#show port-security address
 
[Figure 18: Fa0/6 Port Security Enabled](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/Fa0_6%20Port%20Security%20Enabled.png)
*#switchport port-security aging time command is not available for the version of IOS installed in S1. This is evident when the '?' command is issued to determine the options available to complete the activity. In return, the only available option is ageing time. This explains why, upon verifying the port security status, the ageing time remains absolute. Ideally, it should show as Inactivity.
# S2
Repeat the procedures in S1 using the correct information given in the addressing table. See Figure 19
 
[Figure 19: Enable port security on Fa0/18](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/Enable%20port%20security%20on%20Fa0_18.png)
# Step 5: Implement DHCP Snooping
Step 5 seeks to implement advanced Layer 2 security features. This includes globally enabling DHCP snooping and, for specific VLANs, configuring trusted interfaces for legitimate DHCP traffic. Additionally, DHCP snooping rate limits are set on access ports to prevent starvation attacks. Finally, PortFast and BPDU Guard are enabled on access ports. These measures collectively prevent rogue DHCP servers, mitigate MAC flooding, and protect the spanning-tree topology from unauthorized BPDUs. This is done in the following section.
# S2
Enable dhcp snooping and configure dhcp snooping on VLAN 10. Enter on S2:   	
#ip dhcp snooping     
#ip dhcp snooping vlan 10
Configure the trunk port on S2 as trusted. Enter  
#interface fa0/1   
#ip dhcp snooping trust
Limit the untrusted ports fa0/18 to five DHCP packets per second, and verify dhcp snooping in S2.
 
[Figure 19: IP DHCP Snooping Enabled on S2](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/IP%20DHCP%20Snooping%20Enabled%20on%20S2.png)  
Then release and renew the IP configurations in PC-B command prompt, and verify dhcp snooping binding on S2 again. 
#ip dhcp snooping limit rate 5  
#show ip dhcp snooping    
#show ip dhcp snooping binding
This activity presents interesting findings. There is a successful configuration of DHCP snooping as seen in Figure 19. Close look at the figure suggests that fa0/18 is not trusted unlike fa0/1. As it is, when **#ipconfig /release and #ipconfig /renew** are issued in PC-B, the results are as shown in Figure 20. When **#show ip dhcp snooping binding** is issued while the switch is in these same configurations, the output indicates that the number of bindings remains 0. See Figure 20.
 
[Figure 20: DHCP Request Failure and 0 Number of Bindings](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/DHCP%20Request%20Failure%20and%200%20Number%20of%20Bindings.png)
Solution to this issue is to make fa0/18 trust port. Issue **#interface fa0/18**, **#ip dhcp snooping trust**, safe the configurations and verify **#show ip dhcp snooping binding**. This will configure fa0/18 as trusted port, and will allow the switch to allow DHCPACK and DHCPOFFER messages from the router that acts as the DHCP server (It was configured earlier in this activity). The output is as shown in Figure 21. 
 
[Figure 21: DHCP Binding Successful](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/DHCP%20Binding%20Successful.png)
# Step 6: Implement PortFast and BPDU Guard
Step 6 seeks to implement and validate two crucial Spanning Tree Protocol (STP) enhancements for switch security - **PortFast and BPDU Guard**. On access ports connected to end devices, configure PortFast to enable them to transition to the forwarding state immediately without standard STP delays. Also, BPDU is enabled on these same ports to prevent unauthorized switches from **injecting Bridge Protocol Data Units (BPDUs)**, thus protecting the integrity of the spanning tree topology.
# S1
First, configure portfast on access ports (fa0/5-6).
#interface range fa0/5-6  
#spanning-tree portfast
Second, configure bpdu guard on VLAN 10 access ports (fa0/6)
#interface fa0/6	
#spanning-tree bpduguard enable
Third, verify that BPDU guard and PortFast are enabled. See Figure 22.
#show spanning-tree interface fa0/6 detail
 
[Figure 22: PortFast and BPDU Enabled in Fa0/6 and Fa0/](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/PortFast%20and%20BPDU%20Enabled%20in%20Fa0_6%20and%20Fa0_5.png)
# S2
Repeat the procedures used in configuring PortFast and BPDU guard for S1. See Figure 23. 
 
[Figure 23: PortFast and BPDU Enabled in Fa0/18](https://github.com/philiphineck/Switchsecurityconfiguration/blob/main/PortFast%20and%20BPDU%20Enabled%20in%20Fa0_18.png)
# Step 7: Verify End-to-End Connectivity
This section will verify the implemented network configurations and troubleshoot any issues. Here, it is prudent to confirm end-to-end connectivity between devices after all security features have been applied. Utilize ping commands to ensure the network functions as intended, providing secure and reliable communication and diagnosing and resolving any discrepancies encountered during the verification process. After pinging from and to both PCs, all pings were successful, suggesting a successful end-to-end connectivity. 
# Reflection Questions to Answer 
1. In reference to Port Security on S2, why is there no timer value for the remaining age in minutes when sticky learning was configured? 
**This switch does not support the port security aging of sticky secure addresses. For sticky learned MAC addresses in port security, the Lease(sec) often shows 0 because they are treated as persistent, secure static entries rather than dynamic leases with a countdown timer**
2. In reference to Port Security on S2, if you load the running-config script on S2, why will PC-B on port 18 never get an IP address via DHCP? 
**Port security is set for only two MAC addresses and port 18 has two "sticky" MAC addresses bound to the port. If PC-B's MAC address is different or exceeds the maximum 2 limit, a port security violation will occur. Due to the protect violation mode, frames from PC-B will be dropped, preventing it from obtaining an IP address via DHCP. The 'protect' violation will never send a console/syslog message or increment the violation counter.** 
3. In reference to Port Security, what is the difference between the absolute aging type and inactivity aging type? 
**On the one hand, Inactivity aging removes a secure MAC address from the table if no traffic is seen from it for the specified time. On the other hand, absolute ageing removes the address after the specified time ends, irrespective of whether there has been continuous activity from that MAC address.**
# Conclusion
This lab activity offers hands-on experience in configuring and securing a small network, moving from basic device setup to advanced Layer 2 security. VLANs were created to segment the network, enable efficient traffic management, and create isolated broadcast domains. The activity of configuring robust port security measures has been exciting. Unused ports have been moved to a "ParkingLot" VLAN 999, sticky MAC addresses configured, and violation modes set to control access. Furthermore, the lab explored DHCP snooping to prevent rogue DHCP servers and PortFast with BPDU Guard to enhance spanning tree stability and protect against unauthorized devices. These configurations collectively demonstrate how to build a foundational yet secure, wired network infrastructure, emphasizing the importance of defence-in-depth at the access layer. 
