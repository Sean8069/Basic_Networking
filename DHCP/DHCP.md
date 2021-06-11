# Dynamic Host Configuration Protocol
  > Estimated reading time for begineers (30 minutes)
## Table of Content
  1. [Why we need DHCP?](#intro)
  2. [D.O.R.A Concept](#dora)
  3. [Configuration](#config)
  4. [ip helper-address](#iphelp)
  5. [Conclusion](#conclusion)
 
 ## Why we need DHCP? <a name="intro"></a>
 
 Dynamic Host Configuration Protocol aka DHCP is probably one of the most commonly used services in the TCP/IP network. At its core, DHCP is a protocol that assigned ip addresses automatically to the clients. Before diving any deeper into DHCP, I always like to ask myself why, why is DHCP needed in the first place, what would happen if we do not have DHCP. Well, let's say you are a network administrator at Company A, your boss Derek wanted you to assign ip addresses of 5 client computers for the new employees, so you did what he said and statically assigned the ip addresses and subnet masks of each computer respectively. Next year, Company A grows bigger and Derek has the capital to hire more employees, so he did. He hired 100 employees, and you as the network administrator is asked again to now configure the ip addresses of the 100 client computers. See the problem now? This is why DHCP protocol is needed, it helps to automate task so that we humans can focus on other important tasks at hand and let the protocol perform the repetitive tasks. The following are a few DHCP charateristics:
 - Auto configuration of host IP settings: Main function of DHCP, result in less human error.
 - Host IP configuration is controlled by IT staff
 - Assigns a temporary lease of ip addresses: The act of assigning ip addresses to host is called leasing. With DHCP leasing, the DHCP server can reclaim IP addresses when a device is removed from the network, making better use of the available addresses.
 - Enables mobility
 - Allow permenant assignment of ip addresses: This is usually performed on the routers, servers and other critical infrastructures.  


 ## D.O.R.A Concept <a name="dora"></a>
 
 We now understand that DHCP leases ip addresses automatically, but how? We first need to understand a key concept first. DHCP is a service in the TCP/IP network, a service is provided by the server, and a client uses the service provided by the server. So, we can conclude we need a server in order to lease ip addresses to clients. Actually router can lease ip addresses to the clients as well, which is why we can have internet connectively in our home router* without having to manually input the ip address. 
 
  > \* Technically it's called an Access Point (AP), it has router function built into it, but for now we will refer it as router to avoid confusion.

For clients to obtain ip address from a DHCP server or router, client and the server will exchange DHCP messages on UDP client port 68 and UDP server port 67. They will exchange 4 messages during this process, namely DISCOVER, OFFER, REQUEST and ACKNOWLEDGE. DISCOVER and REQUEST are sent by the client, OFFER and ACKNOWLEDGE are sent by the server.
<br />
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121533644-0e7abe00-ca33-11eb-83d4-dcac81302a0f.png)
<br />
<br />
The above image shows the DHCP DORA process, as shown it has DISCOVER, OFFER, REQUEST and ACKNOWLEDGE. We will discuss all the messages to understand DHCP DORA process better. (Click this [link](./DHCP_Messages.pcapng) to download the .pcapng file)
<br />
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121621755-60592d80-ca9f-11eb-8604-13ffdd00abc8.png)
<br />
<br />
<br />
First, let's take a look at the DISCOVER message which is the first step in the DORA process. To request for an ip address on its interface, PC1 sends a broadcast DISCOVER message into the network. Well, why send a broadcast message? This is because we do not know where the DHCP server is located, so the only way to find out where the DHCP server is, is by sending DISCOVER messages to every single device on that network. Let's take a closer look at wireshark capture of the DISCOVER message. In the ethernet header, the destination MAC is set to ffff.ffff.ffff (all f's is L2 broadcast) and the source MAC is the PC1. In the network layer, we have the destination ip set to 255.255.255.255 which is known as a local broadcast address, meaning that it only forwards the packet to every devices that reside in that subnet, router will drop this packet by default. The source ip address is then set to 0.0.0.0 since we do not have an ip address. In the transport layer, UDP is used with client port being 68 and the server port being 67 as mentioned previously. The packet is then encapsulated with DHCP DISCOVER, with almost all the information set to 0.0.0.0 since we do not have these information yet. This packet is then send to all the devices in the network. In this case, it only sends to R1.
<br />
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121624860-e9269800-caa4-11eb-941f-0e8f8b255b67.png)
<br />
<br />
<br />
Wait, before the server reply with a DHCP OFFER, what is frame 19 in the wireshark packet capture? Before replying an OFFER message to the PC1, R1 needs to make sure that there is no duplicate ip address that it about to assign. Therefore, R1 sends an ARP probe request, which is a broadcast message into the network, if it does not receive a reply, meaning that there is no duplicate ip address and it will proceed with the OFFER message. In this case, the sender ip address is R1, is asking if anyone has the ip address of 192.168.0.11, no one reply so the R1 proceed to send the OFFER message.
<br />
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121626597-4ff98080-caa8-11eb-9814-463053206d6a.png)
<br />
<br />
<br />
R1 sends the OFFER message with broadcast message on L2 and L3. Well, you might be wondering, how does PC1 knows that the message is for itself and not other machine? This is a great question, look at option (61) Client identifier in the DISCOVER message, it contains the MAC address of the PC1, and look at the ‘Client MAC address’ in the OFFER message, it is the MAC address of PC1! The DISCOVER message sends an option (61) that contains PC1 MAC address R1 and R1 look at option 61 and understand that it needs to send this OFFER packet this is particular MAC address which is how PC1 knows that the OFFER message is for itself. Now, look closer in the options that the OFFER message has. We can see that, it contains DHCP server identifier, IP Address Lease Time, Subnet Mask, Router and Domain Name Server. These options are sent by R1 because PC1 requests for these parameters in the option (55) Parameter Request List. Note that some of these options are optional.
<br />
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121628458-d8c5eb80-caab-11eb-90d3-146377adadea.png)
<br />
<br />
<br />

<br />
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121490823-f8a3d380-ca07-11eb-8336-2cd77232be15.png)




Clicking [this link](./DHCP_Messages.pcapng)



 
 
 
 ## Configuration <a name="config"></a>
 ## ip helper-address <a name="iphelp"></a>
 ## Conclusion <a name="conclusion"></a>
