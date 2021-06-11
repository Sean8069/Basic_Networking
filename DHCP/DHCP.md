# Dynamic Host Configuration Protocol
  > Estimated reading time for begineers (30 minutes)
## Table of Content
  1. [Why we need DHCP?](#intro)
  2. [D.O.R.A Concept](#dora)
  3. [Configuration](#config)
  4. [ip helper-address](#iphelp)
  5. [Conclusion](#conclusion)
  6. [Reference](#reference)
 
 ## Why we need DHCP? <a name="intro"></a>
 
 Dynamic Host Configuration Protocol aka DHCP is probably one of the most commonly used services in the TCP/IP network. At its core, DHCP is a protocol that assigned ip addresses automatically to the clients. Before diving any deeper into DHCP, I always like to ask myself why, why is DHCP needed in the first place, what would happen if we do not have DHCP. Well, let's say you are a network administrator at Company A, your boss Derek wanted you to assign ip addresses of 5 client computers for the new employees, so you did what he said and statically assigned the ip addresses and subnet masks of each computer respectively. Next year, Company A grows bigger and Derek has the capital to hire more employees, so he did. He hired 100 employees, and you as the network administrator is asked again to now configure the ip addresses of the 100 client computers. See the problem now? This is why DHCP protocol is needed, it helps to automate task so that we humans can focus on other important tasks at hand and let the protocol perform the repetitive tasks. The following are a few DHCP charateristics:
 - Auto configuration of host IP settings: Main function of DHCP, result in less human error.
 - Host IP configuration is controlled by IT staff
 - Assigns a temporary lease of ip addresses: The act of assigning ip addresses to host is called leasing. With DHCP leasing, the DHCP server can reclaim IP addresses when a device is removed from the network, making better use of the available addresses.
 - Enables mobility
 - Allow permenant assignment of ip addresses: This is usually performed on the routers, servers and other critical infrastructures.  


 ## D.O.R.A Concept <a name="dora"></a>
 
 We now understand that DHCP leases ip addresses automatically, but how? We first need to understand a key concept first. DHCP is a service in the TCP/IP network, a service is provided by the server, and a client uses the service provided by the server. So, we can conclude we need a server in order to lease ip addresses to clients. Actually router can lease ip addresses to the clients as well, which is why we can have internet connectively in our home router* without having to manually input the ip address. 
 
  > \* Technically a home router is called an Access Point (AP), it has router function built into it, but for now we will refer it as router to avoid confusion.

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
Upon receiving the OFFER message, PC1 will send a REQUEST message, requesting for that ip address that was offer by R1, which is 192.168.0.11. Note that the source ip address is still 0.0.0.0, because the R1 hasn’t acknowledge the requested ip address from PC1. The DHCP server address is now determined by the option (54) to prevent any other DHCP server in the network from acknowledging it.
<br />
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121629608-217ea400-caae-11eb-914a-08b1fa7e402b.png)
<br />
<br />
<br />
When R1 receives the REQUEST message, it will reply back to PC1 with an ACKNOWLEDGE message by giving PC1 the permission to use the requested ip address and other parameter. This concludes the whole DORA process.
<br />
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121630398-a61df200-caaf-11eb-8fdb-40c8e77925fa.png)
<br />
<br />
<br />
One last thing after the DORA process, notice in frame 23, there is a gratuitous ARP reply send, what it that? Gratuitous ARP in this instance is used to announce a node existence, this happens when a host newly joins a network. This is an attempt to pre-emptively populate ARP caches of neighbouring hosts without requiring them to initiate the Traditional ARP process. Note that the Opcode is 2, signify that it is a reply, despite the fact the there is no request. This can be interpreted as Gratuitous ARP is a broadcast reply that was not prompted by an ARP Request.
<br />
<br />
<br />
 
 
 ## Configuration <a name="config"></a>
 Now that we are done with the boring concept theory, let's get our hands dirty and start configuring!
<br />
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121643231-29e1d980-cac4-11eb-9936-b57f1984f49e.png)
<br />
<br />
<br />
This is a simple topology created for this purpose. R1 is a router that will act as the DHCP Server, S1 is a switch, PC1-4 indicate that is a user computer. First, we will start by configuring the router to be a DHCP server. Open up the CLI of the router, we start at the global config mode. We first need to configure the interface connected to the switch to allow DISCOVER messages to be recevied on that interface. In the global config mode, entered as such:
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121644895-2bac9c80-cac6-11eb-9a2c-f7a6e7735b85.png)
<br />
<br />
Now that we have configure the interface, we will configure the DHCP pool. By configuring the DHCP pool, we allow a set of ip addresses to be leased by the DHCP server. Back in the global config mode, do as such:
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121645182-7a5a3680-cac6-11eb-98c5-20405534437c.png)
<br />
<br />
Why we need exclude ip addresses first? This is becasue we want to make sure certain devices will have that ip addresses permenantly, this is usually assigned on DHCP server, routers and etc. We then create a DHCP pool named POOL_ONE. Upon creating a DHCP pool, we will enter (dhcp-config)#. In (dhcp-config)#, we need to set a list of ip address and subnet mask for the server to assign ip address, thus the network command follow by the network-id and subnet mask. After that, we need to provide a default router ip address and a dns-server address (optional). With this, we are done with the DHCP configuration. Now we just have to check that our running config is correct.
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121646632-1a648f80-cac8-11eb-9fc8-29d0e3406140.png)
<br />
<br />
Everything looks good, now we will move on with requesting ip address from the PCs.
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121647936-885d8680-cac9-11eb-83f3-ca0a580d6243.png)
<br />
<br />
Nice, PC1-4 successfully assigned a ip address by the DHCP server. Let's take a look at the show DHCP command in the router.
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121648517-2c473200-caca-11eb-8845-aabcdc7413fa.png)
<br />
<br />
The 'show ip dhcp binding' shows all the ip addresses that are leased by the DHCP server, it also contains the lease time expiration date indicating when will the ip addresses expire and renew again.
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121649519-3a498280-cacb-11eb-9ecc-0088502eb0af.png)
<br />
<br />
The 'show ip dhcp pool' command shows the the number of ip addresses that are leased by the DHCP server, also indicating which ip address is to be used upon the next request. Thus, we have successfully configure our DHCP server.
<br />
<br />
>Remeber to use the 'write' command or 'copy running-config startup-config' command to save the configuration.

>Do note that I'm using GNS3 to run these simulation, however, you can also achieved this by using Cisco Packet Tracer.


 ## ip helper-address <a name="iphelp"></a>
 We are done with DHCP right? Well, not quite. Let's take a look at another topology for now.
<br />
<br />
 ![image](https://user-images.githubusercontent.com/73285881/121661357-b39aa280-cad6-11eb-87ec-b5c3664c19f4.png)
<br />
<br />
What if our DHCP server is located at another subnet/network? You might be thinking it works just fine, the DISCOVER message will forward the message to the DHCP server at 172.16.0.0 subnet. No, remember the DORA process we discussed? DISCOVERY message set its destination ip address to 255.255.255.255, which is called a local broadcast address. DHCP server will then look at this address and send an OFFER message back. However, as mentioned previously, all routers by default drops the local broadcast address, it will not forward the packet to 172.16.0.0 network, where the DHCP server resides. So, we need another command to do so. This is where the ip helper-address comes into play, it tells the router to forward the local broadcast address to the DHCP server. We will enter this command at R1 192.168.0.0 facing interface, which is int g0/0.
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121662824-5bfd3680-cad8-11eb-90a4-f1e0dff9bbf8.png)
<br />
<br />
Once this command is entered, the local broadcast address will now be forwared to the DHCP server.
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121663231-c910cc00-cad8-11eb-947d-60848ddab66b.png)
<br />
<br />
As shown, PC1 has successfully obtained an ip address from the DHCP server at 172.16.0.0/24 network. Notice that the DHCP server is reside at a another network, different from the previous example where the DHCP server resides at the local subnet same as the PCs.


 ## Conclusion <a name="conclusion"></a>
This conludes the basic of DHCP. DHCP is one of the most basic yet important concept to understand, it really helps you to visualize and map a network topology better. Also, this is one of my favourite protocol to discuss. Thank you for reading till this far, hope to see you again next time.

## References <a name="reference"></a>
- <a href="https://www.amazon.com/CCNA-200-301-Official-Guide-Library/dp/1587147149?ref_=Oct_s9_apbd_obs_hd_bw_bPfhUB&pf_rd_r=HSB9PE8V06FJ1MV8QH97&pf_rd_p=d847c1a7-83f0-55db-8de8-f9a8e8fc20a1&pf_rd_s=merchandised-search-8&pf_rd_t=BROWSE&pf_rd_i=379347011">CCNA 200-301 Official Cert Guide Library 1st Edition</a>
- <a href="https://study-ccna.com/configure-cisco-router-as-dhcp-server/">Configuring DHCP</a>
- <a href="https://www.practicalnetworking.net/series/arp/gratuitous-arp/">Gratuitous ARP</a>

