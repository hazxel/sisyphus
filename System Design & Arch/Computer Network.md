# Computer Network

### OSI model (5 layers)

1. Physical layer
2. Data link layer: Mac header
3. Network layer (IP layer): IP/ICMP(test connectivity)/ARP(IP->MAC)
4. Transport layer: TCP/UDP header
5. Application layer: HTTP/FTP/SSH/SMTP/DNS

> MAC: 物理网卡的唯一标识，像人的身份证号，IP:网络中的逻辑地址，支持路由等功能，类似人的物理位置

### TCP (Transmission Control Protocal)

##### Connection Establishment: Three Way Handshake

1. SYN: In the first step, the client wants to establish a connection with a server, so it sends a segment with SYN (Synchronize Sequence Number) which informs the server that the client is likely to start communication and with what sequence number it starts segments with.
2. SYN + ACK: Server responds to the client request with SYN-ACK signal bits set. Acknowledgement(ACK) signifies the response of the segment it received and SYN signifies with what sequence number it is likely to start the segments with
3. ACK: In the final part client acknowledges the response of the server and they both establish a reliable connection with which they will start the actual data transfer

> Why we need the third handshake? Because the First Package might be delayed, in which case the client may send one or more such packages, and with the third handshake, the client can make sure that only one connection is established. (otherwise the server will be confused)

##### Connection Termination: Four Way Handshake

1. FIN: Firstly, from one side of the connection, either from the client or the server the FIN flag will be sent as the request for the termination of the connection.
2. ACK: In the second step, whoever receives the FIN flag will then be sending an ACK flag as the acknowledgment for the closing request to the other side.
3. FIN: And, at the Later step, the server will also send a FIN flag as the closing signal to the other side.
4. ACK: In the final step, the TCP, who received the final FIN flag, will be sending an ACK flag as the final Acknowledgement for the suggested connection closing.

> Why we are having 4 handshakes, not 3? Why not send 2 and 3 in the same packet? 
>
> The four-way handshake works as a pair of two-way Handshakes, each pair of FIN-ACK meaning one side not sending message any more (but still receive). That is to say, the termination process is divided into 2 stages, in each stage one side trying to **notify the other that itself is not sending messages**, and we can move to the next stage only when the previous stage is done. The 2 and 3 belongs to different stages, so they cannot be put into the same package.
>
> In other words, when establishing the connection, there are no data transmitting on the network, so server ACK and server SYN can be combined. However, when closing the connection, there are data transmitting. That is to say, when server sending the ACK, it might still have data to transmit, so it can't send FIN at that time.

##### Flow Control

The flow control mechanism tells the sender the maximum speed at which the data can be sent to the receiver device. The sender adjusts the speed as per the receiver’s capacity to avoid sending packets when the receive buffer is full. 流量控制是避免「发送方」的数据填满「接收方」的缓存。

##### Congestion Control

TCP uses congestion control algorithm that includes various aspects of an additive increase/multiplicative decrease (AIMD) scheme, along with other schemes including slow start and congestion window (CWND), to achieve congestion avoidance. TCP 被设计成一个无私的协议，当网络发送拥塞时，TCP 会降低发送的数据量，避免数据填满整个网络。

##### Sliding Window Protocal

发送窗口的值是swnd = min(cwnd, rwnd)，也就是拥塞窗口和接收窗口中的最小值



### TCP vs UDP

|                             TCP                              |                        UDP                        |
| :----------------------------------------------------------: | :-----------------------------------------------: |
|                     Connection-oriented                      |                  Connectionless                   |
|                Able to send data in sequence                 |                  No fixed order                   |
|                            Slower                            |                      Faster                       |
| Reliable (Error checking, guaranteed delivery, retransmission) |               only basic checksums                |
|             Flow control,  Congestion avoidance              |                         -                         |
|                header doesn't contain length                 |        header contains length (redundent)         |
|                              -                               |               Multicast & Broadcast               |
|           Used by HTTPS, HTTP, SMTP, POP, FTP, etc           | Video conferencing, streaming, **DNS**, VoIP, etc |

### Header

##### IP header: 20-60 bytes

##### TCP header: 20-60 bytes

##### UDP header: 8 bytes

- source port (2 bytes): sender's IP address

- destination port (2 bytes): receiver's IP address

- length (2 bytes): theoretical limit of 65535 bytes of data, the length information is actually redundent

  >  but actually 65507 bytes for IPv4, 8 for UPD header, 20 for IP header

- checksum (2 bytes)

### Socket

##### read/write vs recv/send

recv/send are interfaces designed fir socket; read/write is os call provided by OS for file read/write.

### Distributed Denial of Service (DDos) Attack

Denial of service is typically accomplished by flooding the targeted machine or resource with superfluous requests in an attempt to overload systems and prevent some or all legitimate requests from being fulfilled.

> SYN Flood Attack is a typical kind of DDoS attack: an attacker rapidly initiates a connection to a server without finalizing the connection (sending SYN). The server has to spend resources waiting for half-opened connections, which can consume enough resources to make the system unresponsive to legitimate traffic.

> ICMP Flood Attack: attacker rapidly sends ICMP package to the server, and the server must process and reply every package, and will eventually run out of its resources.

### HTTP

Http is based on TCP/IP.

##### HTTP vs HTTPS

Hypertext Transfer Protocol Secure (HTTPS) is still based on HTTP protocal, but using SSL/TLS to encrypt the package.

##### Status Code

- *1xx*：message, request received (maybe processing)
- *2xx*：success, request accepted and processed
  - *200*：success (of get/post)
- *3xx*：redirect, further operation need to be done
- *4xx*：client error
  - *401*：unauthorized, requres user authentication 
  - *403*：forbidden, server refused to execute
  - *404*：resource not found
  - *405*：method not allowed
- *5xx*：server error
  - *500*：server internal error
  - *501*：not implemented

### Endianness

不管是大端还是小端，**数据一定是从内存的低地址依次向高地址读取和写入**。

- Big Endian:  stores the most significant **byte** of a **word** at the smallest memory address. (TCP)
- Little Endian:  stores the most significant **byte** of a **word** at the smallest memory address.

> Big-endianness is the dominant ordering in networking protocols. (transmitting the most significant byte first.)

> a word is the natural unit of data used by a particular processor design. A word is a fixed-sized datum handled as a unit by the instruction set or the hardware of the processor.

