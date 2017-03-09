# Covert-TCP-IP-Protocol
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
