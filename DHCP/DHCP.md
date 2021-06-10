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

For clients to obtain ip address from a DHCP server or router, client and the server will exchange DHCP messages on client port 68 and server port 67. They will exchange 4 messages during this process, namely DISCOVER, OFFER, REQUEST, ACKNOWLEDGE. DISCOVER and REQUEST are sent by the client, OFFER and ACKNOWLEDGE are sent by the server.
<br />
<br />
<br />
![image](https://user-images.githubusercontent.com/73285881/121489723-f0976400-ca06-11eb-8355-d35e2f1d6fd7.png)
![image](https://user-images.githubusercontent.com/73285881/121490720-df028c00-ca07-11eb-992c-cd27966a93a7.png)
![image](https://user-images.githubusercontent.com/73285881/121490354-80d5a900-ca07-11eb-882a-5fd0740c6cf0.png)
![image](https://user-images.githubusercontent.com/73285881/121490823-f8a3d380-ca07-11eb-8336-2cd77232be15.png)



 
 
 
 ## Configuration <a name="config"></a>
 ## ip helper-address <a name="iphelp"></a>
 ## Conclusion <a name="conclusion"></a>
