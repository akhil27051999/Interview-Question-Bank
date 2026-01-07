# Networking Fundamentals — Quick Reference

A concise, interview-ready README covering core networking concepts: IPv4, network types, LANs, VPNs, topologies, protocols, OSI/TCP‑IP models, and common troubleshooting/terms.

---

## Table of Contents

1. [What is an IPv4 address?](#what-is-an-ipv4-address)  
2. [IPv4 Classes](#ipv4-classes)  
3. [Private & Special IP Addresses](#private--special-ip-addresses)  
4. [Types of Networks](#types-of-networks)  
5. [Local Area Network (LAN)](#local-area-network-lan)  
6. [Virtual Private Network (VPN)](#virtual-private-network-vpn)  
   - [Advantages of VPN](#advantages-of-vpn)  
   - [Types of VPN](#types-of-vpn)  
7. [Nodes and Links](#nodes-and-links)  
8. [Network Topology](#network-topology)  
   - [Common Topologies](#common-topologies)  
9. [Network Classification by Area](#network-classification-by-area)  
10. [DNS](#dns)  
11. [Router vs Gateway](#router-vs-gateway)  
12. [Common Protocols (SMTP, FTP, TCP, UDP, ICMP, DHCP, ARP)](#common-protocols)  
13. [MAC Address vs IP Address](#mac-address-vs-ip-address)  
14. [Subnetting (What is a subnet?)](#subnetting-what-is-a-subnet)  
15. [Hub vs Switch](#hub-vs-switch)  
16. [ipconfig vs ifconfig](#ipconfig-vs-ifconfig)  
17. [Firewall](#firewall)  
18. [Unicast / Anycast / Multicast / Broadcast](#unicast--anycast--multicast--broadcast)  
19. [What happens when you enter `google.com` in a browser?](#what-happens-when-you-enter-googlecom-in-a-browser)  
20. [Conclusion](#conclusion)

---

# Networking Interview Questions & Answers

This README contains the provided networking questions and their answers, formatted for clarity and readability. Each question is shown as a heading; the answer side-heading is highlighted as "### Answer" and the original answer text has been preserved (reformatted for presentation only). Each question is separated from the next with a horizontal rule.

---

## 1. What is an IPv4 address? What are the different classes of IPv4?

### Answer
An IP address is a 32-bit dynamic address of a node in the network. An IPv4 address has 4 octets of 8-bit each with each number with a value up to 255.  
IPv4 classes are differentiated based on the number of hosts it supports on the network. There are five types of IPv4 classes and are based on the first octet of IP addresses which are classified as Class A, B, C, D, or E.

IPv4 classes (based on first octet):

| IPv4 Class | IPv4 Start Address | IPv4 End Address     | Usage                     |
|------------|---------------------|----------------------|---------------------------|
| A          | 0.0.0.0             | 127.255.255.255     | Used for Large Network    |
| B          | 128.0.0.0           | 191.255.255.255     | Used for Medium Size Network |
| C          | 192.0.0.0           | 223.255.255.255     | Used for Local Area Network |
| D          | 224.0.0.0           | 239.255.255.255     | Reserved for Multicasting |
| E          | 240.0.0.0           | 255.255.255.254     | Study and R&D             |

---

## 2. Explain different types of networks.

### Answer
Below are few types of networks:

| Type | Description |
|------|-------------|
| PAN (Personal Area Network) | Let devices connect and communicate over the range of a person. E.g. connecting Bluetooth devices. |
| LAN (Local Area Network) | It is a privately owned network that operates within and nearby a single building like a home, office, or factory |
| MAN (Metropolitan Area Network) | It connects and covers the whole city. E.g. TV Cable connection over the city |
| WAN (Wide Area Network) | It spans a large geographical area, often a country or continent. The Internet is the largest WAN |
| GAN (Global Area Network) | It is also known as the Internet which connects the globe using satellites. The Internet is also called the Network of WANs. |

---

## 3. Explain LAN (Local Area Network)

### Answer
LANs are widely used to connect computers/laptops and consumer electronics which enables them to share resources (e.g., printers, fax machines) and exchange information. When LANs are used by companies or organizations, they are called enterprise networks. There are two different types of LAN networks i.e. wireless LAN (no wires involved achieved using Wi-Fi) and wired LAN (achieved using LAN cable). Wireless LANs are very popular these days for places where installing wire is difficult. The below diagrams explain both wireless and wired LAN.

LAN (Local Area Network)

---

## 4. Tell me something about VPN (Virtual Private Network)

### Answer
VPN or the Virtual Private Network is a private WAN (Wide Area Network) built on the internet. It allows the creation of a secured tunnel (protected network) between different networks using the internet (public network). By using the VPN, a client can connect to the organization’s network remotely. The below diagram shows an organizational WAN network over Australia created using VPN:

VPN (Virtual Private Network)

---

## 5. What are the advantages of using a VPN?

### Answer
Below are few advantages of using VPN:
- VPN is used to connect offices in different geographical locations remotely and is cheaper when compared to WAN connections.
- VPN is used for secure transactions and confidential data transfer between multiple offices located in different geographical locations.
- VPN keeps an organization’s information secured against any potential threats or intrusions by using virtualization.
- VPN encrypts the internet traffic and disguises the online identity.

---

## 6. What are the different types of VPN?

### Answer
Few types of VPN are:
- Access VPN: Access VPN is used to provide connectivity to remote mobile users and telecommuters. It serves as an alternative to dial-up connections or ISDN (Integrated Services Digital Network) connections. It is a low-cost solution and provides a wide range of connectivity.
- Site-to-Site VPN: A Site-to-Site or Router-to-Router VPN is commonly used in large companies having branches in different locations to connect the network of one office to another in different locations. There are 2 sub-categories as mentioned below:
  - Intranet VPN: Intranet VPN is useful for connecting remote offices in different geographical locations using shared infrastructure (internet connectivity and servers) with the same accessibility policies as a private WAN (wide area network).
  - Extranet VPN: Extranet VPN uses shared infrastructure over an intranet, suppliers, customers, partners, and other entities and connects them using dedicated connections.

---

## 7. What are nodes and links?

### Answer
Node: Any communicating device in a network is called a Node. Node is the point of intersection in a network. It can send/receive data and information within a network. Examples of the node can be computers, laptops, printers, servers, modems, etc.  
Link: A link or edge refers to the connectivity between two nodes in the network. It includes the type of connectivity (wired or wireless) between the nodes and protocols used for one node to be able to communicate with the other.

Nodes and Links

Advance your career with  Mock Assessments  
Real-world coding challenges for top company interviews  
Real-Life Problems  
Detailed reports  
Attempt Now

---

## 8. What is the network topology?

### Answer
Network topology is a physical layout of the network, connecting the different nodes using the links. It depicts the connectivity between the computers, devices, cables, etc.

---

## 9. Define different types of network topology

### Answer
The different types of network topology are given below:

Bus Topology:
- All the nodes are connected using the central link known as the bus.
- It is useful to connect a smaller number of devices.
- If the main cable gets damaged, it will damage the whole network.

Star Topology:
- All the nodes are connected to one single node known as the central node.
- It is more robust.
- If the central node fails the complete network is damaged.
- Easy to troubleshoot.
- Mainly used in home and office networks.

Ring Topology:
- Each node is connected to exactly two nodes forming a ring structure
- If one of the nodes are damaged, it will damage the whole network
- It is used very rarely as it is expensive and hard to install and manage

Mesh Topology:
- Each node is connected to one or many nodes.
- It is robust as failure in one link only disconnects that node.
- It is rarely used and installation and management are difficult.

Tree Topology:
- A combination of star and bus topology also know as an extended bus topology.
- All the smaller star networks are connected to a single bus.
- If the main bus fails, the whole network is damaged.

Hybrid:
- It is a combination of different topologies to form a new topology.
- It helps to ignore the drawback of a particular topology and helps to pick the strengths from other.

---

## 10. How are Network types classified?

### Answer
Network types can be classified and divided based on the area of distribution of the network. The below diagram would help to understand the same:

Network Types

---

## 11. What are Private and Special IP addresses?

### Answer
Private Address: For each class, there are specific IPs that are reserved specifically for private use only. This IP address cannot be used for devices on the Internet as they are non-routable.

Private IPv4 ranges:

| IPv4 Class | Private IPv4 Start Address | Private IPv4 End Address |
|------------|-----------------------------|--------------------------|
| A          | 10.0.0.0                    | 10.255.255.255           |
| B          | 172.16.0.0                  | 172.31.255.255           |
| C          | 192.168.0.0                 | 192.168.255.255          |

Special Address: IP Range from 127.0.0.1 to 127.255.255.255 are network testing addresses also known as loopback addresses are the special IP address.

---

# Intermediate Interview Questions

(Each question below is shown with its highlighted answer.)

---

### 1. What is the DNS?

### Answer
DNS is the Domain Name System. It is considered as the devices/services directory of the Internet. It is a decentralized and hierarchical naming system for devices/services connected to the Internet. It translates the domain names to their corresponding IPs. For e.g. interviewbit.com to 172.217.166.36. It uses port 53 by default.  

Get Access to 250+ Guides with Scaler Mobile App!  
Experience free learning content on the Scaler Mobile App  
4.5  
100K+  
Play Store

---

### 2. What is the use of a router and how is it different from a gateway?

### Answer
The router is a networking device used for connecting two or more network segments. It directs the traffic in the network. It transfers information and data like web pages, emails, images, videos, etc. from source to destination in the form of packets. It operates at the network layer. The gateways are also used to route and regulate the network traffic but, they can also send data between two dissimilar networks while a router can only send data to similar networks.

---

### 3. What is the SMTP protocol?

### Answer
SMTP is the Simple Mail Transfer Protocol. SMTP sets the rule for communication between servers. This set of rules helps the software to transmit emails over the internet. It supports both End-to-End and Store-and-Forward methods. It is in always-listening mode on port 25.

SMTP Protocol

---

### 4. Describe the OSI Reference Model

### Answer
Open System Interconnections (OSI) is a network architecture model based on the ISO standards. It is called the OSI model as it deals with connecting the systems that are open for communication with other systems.  
The OSI model has seven layers. The principles used to arrive at the seven layers can be summarized  briefly as below:
- Create a new layer if a different abstraction is needed.
- Each layer should have a well-defined function.
- The function of each layer is chosen based on internationally standardized protocols.

---

### 5. Define the 7 different layers of the OSI Reference Model

### Answer
Here the 7 layers of the OSI reference model:

| Layer       | Unit Exchanged | Description |
|-------------|----------------|-------------|
| Physical    | Bit            | • It is concerned with transmitting raw bits over a communication channel.<br>• Chooses which type of transmission mode is to be selected for the transmission. The available transmission modes are Simplex, Half Duplex and Full Duplex. |
| Data Link   | Frame          | • The main task of this layer is to transform a raw transmission facility into a line that appears free of undetected transmission errors.<br>• It also allows detecting damaged packets using the CRC (Cyclic Redundancy Check) error-detecting, code.<br>• When more than one node is connected to a shared link, Data Link Layer protocols are required to determine which device has control over the link at a given time.<br>• It is implemented by protocols like CSMA/CD, CSMA/CA, ALOHA, and Token Passing. |
| Network     | Packet         | • It controls the operation of the subnet.<br>• The network layer takes care of feedback messaging through ICMP messages. |
| Transport   | TPDU - Transaction Protocol Data Unit | • The basic functionality of this layer is to accept data from the above layers, split it up into smaller units if needed, pass these to the network layer, and ensure that all the pieces arrive correctly at the other end.<br>• The Transport Layer takes care of Segmentation and Reassembly. |
| Session     | SPDU - Session Protocol Data Unit | • The session layer allows users on different machines to establish sessions between them.<br>• Dialogue control is using the full-duplex link as half-duplex. It sends out dummy packets from the client to the server when the client is ideal. |
| Presentation| PPDU - Presentation Protocol Data Unit | • The presentation layer is concerned with the syntax and semantics of the information transmitted.<br>• It translates a message from a common form to the encoded format which will be understood by the receiver. |
| Application | APDU - Application Protocol Data Unit | • It contains a variety of protocols that are commonly needed by users.<br>• The application layer sends data of any size to the transport layer. |

---

### 6. Describe the TCP/IP Reference Model

### Answer
It is a compressed version of the OSI model with only 4 layers. It was developed by the US Department of Defence (DoD) in the 1980s. The name of this model is based on 2 standard protocols used i.e. TCP (Transmission Control Protocol) and IP (Internet Protocol).

---

### 7. Define the 4 different layers of the TCP/IP Reference Model

### Answer
Layers of TCP/IP:

| Layer     | Description |
|-----------|-------------|
| Link      | Decides which links such as serial lines or classic Ethernet must be used to meet the needs of the connectionless internet layer. |
| Internet  | • The internet layer is the most important layer which holds the whole architecture together.<br>• It delivers the IP packets where they are supposed to be delivered. |
| Transport | Its functionality is almost the same as the OSI transport layer. It enables peer entities on the network to carry on a conversation. |
| Application | It contains all the higher-level protocols. |

---

### 8. Differentiate OSI Reference Model with TCP/IP Reference Model

### Answer

| OSI Reference Model | TCP/IP Reference Model |
|---------------------|------------------------|
| 7 layered architecture | 4 layered architecture |
| Fixed boundaries and functionality for each layer | Flexible architecture with no strict boundaries between layers |
| Low Reliability | High Reliability |
| Vertical Layer Approach | Horizontal Layer Approach |

---

### 9. What are the HTTP and the HTTPS protocol?

### Answer
HTTP is the HyperText Transfer Protocol which defines the set of rules and standards on how the information can be transmitted on the World Wide Web (WWW). It helps the web browsers and web servers for communication. It is a ‘stateless protocol’ where each command is independent with respect to the previous command. HTTP is an application layer protocol built upon the TCP. It uses port 80 by default.  
HTTPS is the HyperText Transfer Protocol Secure or Secure HTTP. It is an advanced and secured version of HTTP. On top of HTTP, SSL/TLS protocol is used to provide security. It enables secure transactions by encrypting the communication and also helps identify network servers securely. It uses port 443 by default.

---

# Advanced Interview Questions

---

### 1. What is the FTP protocol?

### Answer
FTP is a File Transfer Protocol. It is an application layer protocol used to transfer files and data reliably and efficiently between hosts. It can also be used to download files from remote servers to your computer. It uses port 27 by default.

---

### 2. What is the TCP protocol?

### Answer
TCP or TCP/IP is the Transmission Control Protocol/Internet Protocol. It is a set of rules that decides how a computer connects to the Internet and how to transmit the data over the network. It creates a virtual network when more than one computer is connected to the network and uses the three ways handshake model to establish the connection which makes it more reliable.

---

### 3. What is the UDP protocol?

### Answer
UDP is the User Datagram Protocol and is based on Datagrams. Mainly, it is used for multicasting and broadcasting. Its functionality is almost the same as TCP/IP Protocol except for the three ways of handshaking and error checking. It uses a simple transmission without any hand-shaking which makes it less reliable.

---

### 4. Compare between TCP and UDP

### Answer

| TCP/IP | UDP |
|--------|-----|
| Connection-Oriented Protocol | Connectionless Protocol |
| More Reliable | Less Reliable |
| Slower Transmission | Faster Transmission |
| Packets order can be preserved or can be rearranged | Packets order is not fixed and packets are independent of each other |
| Uses three ways handshake model for connection | No handshake for establishing the connection |
| TCP packets are heavy-weight | UDP packets are light-weight |
| Offers error checking mechanism | No error checking mechanism |
| Protocols like HTTP, FTP, Telnet, SMTP, HTTPS, etc use TCP at the transport layer | Protocols like DNS, RIP, SNMP, RTP, BOOTP, TFTP, NIP, etc use UDP at the transport layer |

TCP VS UDP

---

### 5. What is the ICMP protocol?

### Answer
ICMP is the Internet Control Message Protocol. It is a network layer protocol used for error handling. It is mainly used by network devices like routers for diagnosing the network connection issues and crucial for error reporting and testing if the data is reaching the preferred destination in time. It uses port 7 by default.

---

### 6. What do you mean by the DHCP Protocol?

### Answer
DHCP is the Dynamic Host Configuration Protocol.  
It is an application layer protocol used to auto-configure devices on IP networks enabling them to use the TCP and UDP-based protocols. The DHCP servers auto-assign the IPs and other network configurations to the devices individually which enables them to communicate over the IP network. It helps to get the subnet mask, IP address and helps to resolve the DNS. It uses port 67 by default.

---

### 7. What is the ARP protocol?

### Answer
ARP is Address Resolution Protocol. It is a network-level protocol used to convert the logical address i.e. IP address to the device's physical address i.e. MAC address. It can also be used to get the MAC address of devices when they are trying to communicate over the local network.

ARP Protocol

---

### 8. What is the MAC address and how is it related to NIC?

### Answer
MAC address is the Media Access Control address. It is a 48-bit or 64-bit unique identifier of devices in the network. It is also called the physical address embedded with Network Interface Card (NIC) used at the Data Link Layer. NIC is a hardware component in the networking device using which a device can connect to the network.

---

### 9. Differentiate the MAC address with the IP address

### Answer

| MAC Address | IP Address |
|-------------|------------|
| Media Access Control Address | Internet Protocol Address |
| 6 or 8-byte hexadecimal number | 4 (IPv4) or 16 (IPv6) Byte address |
| It is embedded with NIC | It is obtained from the network |
| Physical Address | Logical Address |
| Operates at Data Link Layer | Operates at Network Layer. |
| Helps to identify the device | Helps to identify the device connectivity on the network. |

---

### 10. What is a subnet?

### Answer
A subnet is a network inside a network achieved by the process called subnetting which helps divide a network into subnets. It is used for getting a higher routing efficiency and enhances the security of the network. It reduces the time to extract the host address from the routing table.

Subnet

---

### 11. Compare the hub vs switch

### Answer

| Hub | Switch |
|-----|--------|
| Operates at Physical Layer | Operates at Data Link Layer |
| Half-Duplex transmission mode | Full-Duplex transmission mode |
| Ethernet devices can be connectedsend | LAN devices can be connected |
| Less complex, less intelligent, and cheaper | Intelligent and effective |
| No software support for the administration | Administration software support is present |
| Less speed up to 100 MBPS | Supports high speed in GBPS |
| Less efficient as there is no way to avoid collisions when more than one nodes sends the packets at the same time | More efficient as the collisions can be avoided or reduced as compared to Hub |

---

### 12. What is the difference between the ipconfig and the ifconfig?

### Answer

| ipconfig | ifconfig |
|----------|----------|
| Internet Protocol Configuration | Interface Configuration |
| Command used in Microsoft operating systems to view and configure network interfaces | Command used in MAC, Linux, UNIX operating systems to view and configure network interfaces |
| Used to get the TCP/IP summary and allows to changes the DHCP and DNS settings |  |

---

### 13. What is the firewall?

### Answer
The firewall is a network security system that is used to monitor the incoming and outgoing traffic and blocks the same based on the firewall security policies. It acts as a wall between the internet (public network) and the networking devices (a private network). It is either a hardware device, software program, or a combination of both. It adds a layer of security to the network.

Firewall

---

### 14. What are Unicasting, Anycasting, Multicasting and Broadcasting?

### Answer
- Unicasting: If the message is sent to a single node from the source then it is known as unicasting. This is commonly used in networks to establish a new connection.
- Anycasting: If the message is sent to any of the nodes from the source then it is known as anycasting. It is mainly used to get the content from any of the servers in the Content Delivery System.
- Multicasting: If the message is sent to a subset of nodes from the source then it is known as multicasting. Used to send the same data to multiple receivers. 
- Broadcasting: If the message is sent to all the nodes in a network from a source then it is known as broadcasting. DHCP and ARP in the local network use broadcasting.

---

### 15. What happens when you enter google.com in the web browser?

### Answer
Below are the steps that are being followed:
- Check the browser cache first if the content is fresh and present in cache display the same.
- If not, the browser checks if the IP of the URL is present in the cache (browser and OS) if not then request the OS to do a DNS lookup using UDP to get the corresponding IP address of the URL from the DNS server to establish a new TCP connection.
- A new TCP connection is set between the browser and the server using three-way handshaking.
- An HTTP request is sent to the server using the TCP connection.
- The web servers running on the Servers handle the incoming HTTP request and send the HTTP response.
- The browser process the HTTP response sent by the server and may close the TCP connection or reuse the same for future requests.
- If the response data is cacheable then browsers cache the same.
- Browser decodes the response and renders the content.

---

## Conclusion

### Answer
In today’s world, it is very hard to stay away from the Internet and that is what makes networking one of the most important interview topics. As of 2021 if we check the facts, there is a total of 1.3 million kilometers of submarine optical fiber cables set globally to connect the world to the Internet. These cables are more than enough to revolve around the earth more than 100 times.
## Top 100 Networking Interview Questions & Answers 
#### source: http://career.guru99.com/
### 1) What is a Link?

A link refers to the connectivity between two devices. It includes the type of cables and protocols used in order for 
one device to be able to communicate with the other.

### 2) What are the layers of the OSI reference model?

There are 7 OSI layers: Physical Layer, Data Link Layer, Network Layer, Transport Layer, Session Layer, Presentation Layer and Application Layer.

### 3) What is backbone network?

A backbone network is a centralized infrastructure that is designed to distribute different routes and data to various networks.
It also handles management of bandwidth and various channels.

### 4) What is a LAN?

LAN is short for Local Area Network. It refers to the connection between computers and other network devices that are located within a small physical location.

### 5) What is a node?

A node refers to a point or joint where a connection takes place. It can be computer or device that is part of a network. Two or more nodes are needed in order to form a network connection.

### 6) What are routers?

Routers can connect two or more network segments. These are intelligent network devices that store information in its routing table
such as paths, hops and bottlenecks. With this info, they are able to determine the best path for data transfer. Routers operate 
at the OSI Network Layer.

### 7) What is point to point link?

It refers to a direct connection between two computers on a network. A point to point connection does not need any other
network devices other than connecting a cable to the NIC cards of both computers.

### 8) What is anonymous FTP?

Anonymous FTP is a way of granting user access to files in public servers. Users that are allowed access to data 
in these servers do not need to identify themselves, but instead log in as an anonymous guest.

### 9) What is subnet mask?

A subnet mask is a combined with an IP address in order to identify two parts: the extended network address and the host address. 
Like an IP address, a subnet mask is made up of 32 bits.

Subnet mask is a mask used to determine what subnet an IP address belongs to.

### 10) What is the maximum length allowed for a UTP cable?

A single segment of UTP cable has an allowable length of 90 to 100 meters. This limitation can be overcome by using repeaters and switches.

### 11) What is data encapsulation?

Data encapsulation is the process of breaking down information into smaller manageable chunks before it is transmitted across the network. It is also in this process that the source and destination addresses are attached into the headers, along with parity checks.

### 12) Describe Network Topology

Network Topology refers to the layout of a computer network. It shows how devices and cables are physically laid out, as well as how they connect to one another.

### 13) What is VPN?

VPN means Virtual Private Network, a technology that allows a secure tunnel to be created across a network such as the Internet. For example, VPNs allow you to establish a secure dial-up connection to a remote server.

### 14) Briefly describe NAT.

NAT is Network Address Translation. This is a protocol that provides a way for multiple computers on a common network to share single connection to the Internet.

### 15) What is the job of the Network Layer under the OSI reference model?

The Network layer is responsible for data routing, packet switching and control of network congestion. Routers operate under this layer.

### 16) How does a network topology affect your decision in setting up a network?

Network topology dictates what media you must use to interconnect devices. It also serves as basis on what materials, connector and terminations that is applicable for the setup.

### 17) What is RIP?

RIP, short for Routing Information Protocol is used by routers to send data from one network to another.
It efficiently manages routing data by broadcasting its routing table to all other routers within the network.
It determines the network distance in units of hops.

### 18) What are different ways of securing a computer network?

There are several ways to do this. Install reliable and updated anti-virus program on all computers. 
Make sure firewalls are setup and configured properly. User authentication will also help a lot. 
All of these combined would make a highly secured network.

### 19) What is NIC?

NIC is short for Network Interface Card. This is a peripheral card that is attached to a PC in order to connect to a network. 
Every NIC has its own MAC address that identifies the PC on the network.

### 20) What is WAN?

WAN stands for Wide Area Network. It is an interconnection of computers and devices that are geographically dispersed. It connects networks that are located in different regions and countries.

### 21) What is the importance of the OSI Physical Layer?

The physical layer does the conversion from data bits to electrical signal, and vice versa. This is where network devices and cable types are considered and setup.

### 22) How many layers are there under TCP/IP?

There are four layers: the Network Layer, Internet Layer, Transport Layer and Application Layer.

### 23) What are proxy servers and how do they protect computer networks?

Proxy servers primarily prevent external users who identifying the IP addresses of an internal network. 
Without knowledge of the correct IP address, even the physical location of the network cannot be identified. 
Proxy servers can make a network virtually invisible to external users.

### 24) What is the function of the OSI Session Layer?

This layer provides the protocols and means for two devices on the network to communicate with each other by holding a session.  This includes setting up the session, managing information exchange during the session, and tear-down process upon termination of the session.

### 25) What is the importance of implementing a Fault Tolerance System? Are there limitations?

A fault tolerance system ensures continuous data availability. This is done by eliminating a single point of failure. 
However, this type of system would not be able to protect data in some cases, such as in accidental deletions.

### 26) What does 10Base-T mean?

The 10 refers to the data transfer rate, in this case is 10Mbps. The word Base refers to base band, as oppose to broad band. T means twisted pair, which is the cable used for that network.

### 27) What is a private IP address?

Private IP addresses are assigned for use on intranets.
These addresses are used for internal networks and are not routable on external public networks.
These ensures that no conflicts are present among internal networks while at the same time the same range of private IP addresses are reusable for multiple intranets since they do not “see” each other.

### 28) What is NOS?

NOS, or Network Operating System, is specialized software whose main task is to provide network connectivity to a
computer in order for it to be able to communicate with other computers and connected devices.

### 29) What is DoS?

DoS, or Denial-of-Service attack, is an attempt to prevent users from being able to access the internet or any 
other network services. Such attacks may come in different forms and are done by a group of perpetuators.
One common method of doing this is to overload the system server so it cannot anymore process legitimate traffic and will be forced to reset.

### 30) What is OSI and what role does it play in computer networks?

OSI (Open Systems Interconnect) serves as a reference model for data communication. 
It is made up of 7 layers, with each layer defining a particular aspect on how network devices connect and communicate with one another. One layer may deal with the physical media used, while another layer dictates how data is actually transmitted across the network.

### 31) What is the purpose of cables being shielded and having twisted pairs?

The main purpose of this is to prevent crosstalk. Crosstalks are electromagnetic interferences or noise that can affect data being transmitted across cables.

### 32) What is the advantage of address sharing?

By using address translation instead of routing, address sharing provides an inherent security benefit. That’s because host PCs on the Internet can only see the public IP address of the external interface on the computer that provides address translation and not the private IP addresses on the internal network.

### 33) What are MAC addresses?

MAC, or Media Access Control, uniquely identifies a device on the network. It is also known as physical address or Ethernet address. A MAC address is made up of 6-byte parts.

### 34) What is the equivalent layer or layers of the TCP/IP Application layer in terms of OSI reference model?

The TCP/IP Application layer actually has three counterparts on the OSI model: the Session layer, Presentation Layer and Application Layer.

### 35) How can you identify the IP class of a given IP address?

By looking at the first octet of any given IP address, you can identify whether it’s Class A, B or C. 
If the first octet begins with a 0 bit, that address is Class A. If it begins with bits 10 then that address is a Class B address.
If it begins with 110, then it’s a Class C network.

### 36) What is the main purpose of OSPF?

OSPF, or Open Shortest Path First, is a link-state routing protocol that uses routing tables to determine the best possible path for data exchange.

### 37) What are firewalls?

Firewalls serve to protect an internal network from external attacks. These external threats can be hackers who want to steal data or computer viruses that can wipe out data in an instant. It also prevents other users from external networks from gaining access to the private network.

### 38) Describe star topology

Star topology consists of a central hub that connects to nodes. This is one of the easiest to setup and maintain.

### 39) What are gateways?

Gateways provide connectivity between two or more network segments. It is usually a computer that runs the gateway
software and provides translation services. This translation is a key in allowing different systems to communicate on the network.

### 40) What is the disadvantage of a star topology?

One major disadvantage of star topology is that once the central hub or switch get damaged, the entire network becomes unusable.

### 41) What is SLIP?

SLIP, or Serial Line Interface Protocol, is actually an old protocol developed during the early UNIX days. This is one of the protocols that are used for remote access.

### 42) Give some examples of private network addresses.

10.0.0.0 with a subnet mask of 255.0.0.0
172.16.0.0 with subnet mask of 255.240.0.0
192.168.0.0 with subnet mask of 255.255.0.0

### 43) What is tracert?

Tracert is a Windows utility program that can used to trace the route taken by data from the router to the destination network. It also shows the number of hops taken during the entire transmission route.

### 44) What are the functions of a network administrator?

A network administrator has many responsibilities that can be summarize into 3 key functions: installation of a network, configuration of network settings, and maintenance/troubleshooting of networks.

### 45) Describe at one disadvantage of a peer to peer network.

When you are accessing the resources that are shared by one of the workstations on the network, that workstation takes a performance hit.

### 46) What is Hybrid Network?

A hybrid network is a network setup that makes use of both client-server and peer-to-peer architecture.

### 47) What is DHCP?

DHCP is short for Dynamic Host Configuration Protocol. Its main task is to automatically assign an IP address to devices across the network. It first checks for the next available address not yet taken by any device, then assigns this to a network device.

### 48) What is the main job of the ARP?

The main task of ARP or Address Resolution Protocol is to map a known IP address to a MAC layer address.

### 49) What is TCP/IP?

TCP/IP is short for Transmission Control Protocol / Internet Protocol. This is a set of protocol layers that is designed to make data exchange possible on different types of computer networks, also known as heterogeneous network.

### 50) How can you manage a network using a router?

Routers have built in console that lets you configure different settings, like security and data logging. You can assign restrictions
to computers, such as what resources it is allowed access, or what particular time of the day they can browse the internet. You can even put restrictions on what websites are not viewable across the entire network.

### 51) What protocol can be applied when you want to transfer files between different platforms, such between UNIX systems and Windows servers?

Use FTP (File Transfer Protocol) for file transfers between such different servers. This is possible because FTP is platform independent.

### 52) What is the use of a default gateway?

Default gateways provide means for the local networks to connect to the external network. The default gateway for connecting to the external network is usually the address of the external router port.

### 53) One way of securing a network is through the use of passwords. What can be considered as good passwords?

Good passwords are made up of not just letters, but by combining letters and numbers. A password that combines uppercase and lowercase letters is favorable than one that uses all upper case or all lower case letters. Passwords must be not words that can easily be guessed by hackers, such as dates, names, favorites, etc. Longer passwords are also better than short ones.

### 54) What is the proper termination rate for UTP cables?

The proper termination for unshielded twisted pair network cable is 100 ohms.

### 55) What is netstat?

Netstat is a command line utility program. It provides useful information about the current TCP/IP settings of a connection.

### 56) What is the number of network IDs in a Class C network?

For a Class C network, the number of usable Network ID bits is 21. The number of possible network IDs is 2 raised to 21 or 2,097,152. The number of host IDs per network ID is 2 raised to 8 minus 2, or 254.

### 57) What happens when you use cables longer than the prescribed length?

Cables that are too long would result in signal loss. This means that data transmission and reception would be affected, because the signal degrades over length.

### 58) What common software problems can lead to network defects?

Software related problems can be any or a combination of the following:
– client server problems
– application conflicts
– error in configuration
– protocol mismatch
– security issues
– user policy and rights issues

### 59) What is ICMP?

ICMP is Internet Control Message Protocol. It provides messaging and communication for protocols within the TCP/IP stack. This is also the protocol that manages error messages that are used by network tools such as PING.

### 60) What is Ping?

Ping is a utility program that allows you to check connectivity between network devices on the network. You can ping a device by using its IP address or device name, such as a computer name.


### 61) What is peer to peer?

Peer to peer are networks that does not reply on a server. All PCs on this network act as individual workstations.

### 62) What is DNS?

DNS is Domain Name System. The main function of this network service is to provide host names to TCP/IP address resolution.
It is responsible for translating domain names into IPs address or vice versa.

Generally, the computer looks for the IP address in the local cache of the operating system, if it finds, it retrieves the url of page. 
However if not, it initiates  a DNS request to look for the domain name of the requested IP. The request is made to an external DNS server.

### 63) What advantages does fiber optics have over other media?

One major advantage of fiber optics is that is it less susceptible to electrical interference. It also supports higher bandwidth, meaning more data can be transmitted and received. Signal degrading is also very minimal over long distances.

### 64) What is the difference between a hub and a switch?

A hub acts as a multiport repeater. However, as more and more devices connect to it, it would not be able to efficiently manage the volume of traffic that passes through it. A switch provides a better alternative that can improve the performance especially when high traffic volume is expected across all ports.

### 65) What are the different network protocols that are supported by Windows RRAS services?

There are three main network protocols supported: NetBEUI, TCP/IP, and IPX.

### 66) What are the maximum networks and hosts in a class A, B and C network?

For Class A, there are 126 possible networks and 16,777,214 hosts
For Class B, there are 16,384 possible networks and 65,534 hosts
For Class C, there are 2,097,152 possible networks and 254 hosts

### 67) What is the standard color sequence of a straight-through cable?

orange/white, orange, green/white, blue, blue/white, green, brown/white, brown.

### 68) What protocols fall under the Application layer of the TCP/IP stack?

The following are the protocols under TCP/IP Application layer: FTP, TFTP, Telnet and SMTP.

### 69) You need to connect two computers for file sharing. Is it possible to do this without using a hub or router?

Yes, you can connect two computers together using only one cable. A crossover type cable can be use in this scenario.
In this setup, the data transmit pin of one cable is connected to the data receive pin of the other cable, and vice versa.

### 70) What is ipconfig?

Ipconfig is a utility program that is commonly used to identify the addresses information of a computer on a network. It can show the physical address as well as the IP address.

### 71) What is the difference between a straight-through and crossover cable?

A straight-through cable is used to connect computers to a switch, hub or router. A crossover cable is used to connect two similar devices together, such as a PC to PC or Hub to hub.

### 72) What is client/server?

Client/server is a type of network wherein one or more computers act as servers. Servers provide a centralized repository of resources such as printers and files. Clients refers to workstation that access the server.

### 73) Describe networking.

Networking refers to the inter connection between computers and peripherals for data communication. Networking can be done using wired cabling or through wireless link.

### 74) When you move the NIC cards from one PC to another PC, does the MAC address gets transferred as well?

Yes, that’s because MAC addresses are hard-wired into the NIC circuitry, not the PC. This also means that a PC can have a different MAC address when the NIC card was replace by another one.

### 75) Explain clustering support

Clustering support refers to the ability of a network operating system to connect multiple servers in a fault-tolerant group. The main purpose of this is the in the event that one server fails, all processing will continue on with the next server in the cluster.

### 76) In a network that contains two servers and twenty workstations, where is the best place to install an Anti-virus program?

An anti-virus program must be installed on all servers and workstations to ensure protection. That’s because individual users can access any workstation and introduce a computer virus when plugging in their removable hard drives or flash drives.

### 77) Describe Ethernet.

Ethernet is one of the popular networking technologies used these days. It was developed during the early 1970s and is based on specifications as stated in the IEEE. Ethernet is used in local area networks.

### 78) What are some drawbacks of implementing a ring topology?

In case one workstation on the network suffers a malfunction, it can bring down the entire network. Another drawback is that when there are adjustments and reconfigurations needed to be performed on a particular part of the network, the entire network has to be temporarily brought down as well.

### 79) What is the difference between CSMA/CD and CSMA/CA?

CSMA/CD, or Collision Detect, retransmits data frames whenever a collision occurred. CSMA/CA, or Collision Avoidance, will first broadcast intent to send prior to data transmission.

### 80) What is SMTP?

SMTP is short for Simple Mail Transfer Protocol. This protocol deals with all Internal mail, and provides the necessary mail delivery services on the TCP/IP protocol stack.

### 81) What is multicast routing?

Multicast routing is a targeted form of broadcasting that sends message to a selected group of user, instead of sending it to all users on a subnet.


### 82) What is the importance of Encryption on a network?

Encryption is the process of translating information into a code that is unreadable by the user. It is then translated back or decrypted back to its normal readable format using a secret key or password. Encryption help ensure that information that is intercepted halfway would remain unreadable because the user has to have the correct password or key for it.

### 83) How are IP addresses arranged and displayed?

IP addresses are displayed as a series of four decimal numbers that are separated by period or dots. Another term for this arrangement is the dotted decimal format. An example is 192.168.101.2

### 84) Explain the importance of authentication.

Authentication is the process of verifying a user’s credentials before he can log into the network. It is normally performed using a username and password. This provides a secure means of limiting the access from unwanted intruders on the network.

### 85) What do mean by tunnel mode?

This is a mode of data exchange wherein two communicating computers do not use IPSec themselves. Instead, the gateway that is connecting their LANs to the transit network creates a virtual tunnel that uses the IPSec protocol to secure all communication that passes through it.

### 86) What are the different technologies involved in establishing WAN links?

Analog connections – using conventional telephone lines; Digital connections – using digital-grade telephone lines; switched connections – using multiple sets of links between sender and receiver to move data.

### 87) What is one advantage of mesh topology?

In the event that one link fails, there will always be another available. Mesh topology is actually one of the most fault-tolerant network topology.

### 88) When troubleshooting computer network problems, what common hardware-related problems can occur?

A large percentage of a network is made up of hardware. Problems in these areas can range from malfunctioning hard drives, broken NICs and even hardware startups. Incorrectly hardware configuration is also one of those culprits to look into.

### 89) What can be done to fix signal attenuation problems?

A common way of dealing with such a problem is to use repeaters and hub, because it will help regenerate the signal and therefore prevent signal loss. Checking if cables are properly terminated is also a must.

### 90) How does dynamic host configuration protocol aid in network administration?

Instead of having to visit each client computer to configure a static IP address, the network administrator can apply dynamic host configuration protocol to create a pool of IP addresses known as scopes that can be dynamically assigned to clients.

### 91) Explain profile in terms of networking concept?

Profiles are the configuration settings made for each user. A profile may be created that puts a user in a group, for example.

### 92) What is sneakernet?

Sneakernet is believed to be the earliest form of networking wherein data is physically transported using removable media, such as disk, tapes.

### 93) What is the role of IEEE in computer networking?

IEEE, or the Institute of Electrical and Electronics Engineers, is an organization composed of engineers that issues and manages standards for electrical and electronic devices. This includes networking devices, network interfaces, cablings and connectors.

### 94) What protocols fall under the TCP/IP Internet Layer?

There are 4 protocols that are being managed by this layer. These are ICMP, IGMP, IP and ARP.

### 95) When it comes to networking, what are rights?

Rights refer to the authorized permission to perform specific actions on the network. Each user on the network can be assigned individual rights, depending on what must be allowed for that user.

### 96) What is one basic requirement for establishing VLANs?

A VLAN is required because at switch level there is only one broadcast domain, it means whenever new user is connected to switch this information is spread throughout the network. VLAN on switch helps to create separate broadcast domain at  switch level. It is used for security purpose.

### 97) What is IPv6?

IPv6 , or Internet Protocol version 6, was developed to replace IPv4. At present, IPv4 is being used to control internet traffic, butis expected to get saturated in the near future. IPv6 was designed to overcome this limitation.

### 98) What is RSA algorithm?

RSA is short for Rivest-Shamir-Adleman algorithm. It is the most commonly used public key encryption algorithm in use today.

### 99) What is mesh topology?

Mesh topology is a setup wherein each device is connected directly to every other device on the network. Consequently, it requires that each device have at least two network connections.

### 100) what is the maximum segment length of a 100Base-FX network?

The maximum allowable length for a network segment using 100Base-FX is 412 meters. The maximum length for the entire network is 5 kilometers.


### 101) What is Socket?

A socket is one endpoint of a two-way communication link between two programs running on the network. A socket is bound to a port number so that the TCP layer can identify the application that data is destined to be sent to. 


### 102) What is a Socks Proxy?

 Socks proxy is a general puporse proxy server that establish TCP connection to another 
	server on the behalf of a client, then routes all the traffic back and forth between
	client and the server

### 103) What is the difference between 2.4 GHz and 5 GHz?

The primary differences between the 2.4 GHz and 5GHz wireless frequencies are range and bandwidth. 5GHz provides faster data rates at a shorter distance, whereas 2.4GHz offers coverage for farther distances, but may perform at slower speeds
