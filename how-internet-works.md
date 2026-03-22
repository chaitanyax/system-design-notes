# How the Internet Works

This chapter provides a foundational guide to understanding the mechanics of the internet, covering essential protocols and network internals critical for system design.

## Table of Contents

1. [URL Resolution Process](#url-resolution-process)
2. [TCP/IP Stack and OSI Model](#tcpip-stack-and-osi-model)
3. [Protocol Internals](#protocol-internals)
4. [Domain Name System (DNS)](#domain-name-system-dns)
5. [Additional Resources](#additional-resources)

## 1. URL Resolution Process

When a user enters a Uniform Resource Locator (URL) such as `https://example.com` into a web browser, a sequence of events occurs to establish a connection and retrieve the requested resources.

### DNS Resolution
The browser must first resolve the human-readable domain name to an IP address. The system checks several layers of cache before initiating a network query:
- Browser cache
- Operating System cache
- Recursive resolver (typically provided by the Internet Service Provider or public DNS)
- If the IP is not cached, the resolver traverses the DNS hierarchy: Root Name Servers -> Top-Level Domain (TLD) Servers -> Authoritative Name Servers.

*Protocol:* DNS relies primarily on UDP over port 53, with a fallback to TCP for large responses.

### TCP Connection Establishment
Once the IP address is known, the client establishes a transmission connection with the server using the Transmission Control Protocol (TCP). This involves a 3-way handshake:
1. The client sends a `SYN` (synchronize) packet.
2. The server responds with a `SYN-ACK` (synchronize-acknowledge) packet.
3. The client replies with an `ACK` (acknowledge) packet.

### TLS Handshake
For secure connections (HTTPS), a Transport Layer Security (TLS) handshake occurs over the established TCP connection. This handshake utilizes asymmetric cryptography (such as RSA or Diffie-Hellman) to securely negotiate a symmetric session key, ensuring subsequent communications are encrypted.

### HTTP Request and Response
With the secure connection established, the browser transmits an HTTP request to the server. The server processes the request and returns an HTTP response containing the requested assets—such as HTML documents, JavaScript files, and images.

## 2. TCP/IP Stack and OSI Model

Computer networking relies on layered communication models. The practical framework used by the modern internet is the TCP/IP stack.

| Layer | Protocols | Description |
|---|---|---|
| **Application** | HTTP, DNS, FTP, SMTP, TLS | Facilitates end-user communication and data presentation. |
| **Transport** | TCP, UDP | Manages reliable (TCP) or fast/unreliable (UDP) data delivery. |
| **Network** | IP (IPv4/IPv6), ICMP | Handles logical addressing and routing across subnets. |
| **Data Link** | Ethernet, ARP, Wi-Fi | Manages physical addressing (MAC) and local delivery. |
| **Physical** | Fiber optics, copper, radio waves | The physical medium for raw bit transmission. |

## 3. Protocol Internals

### Internet Protocol (IP)
The Internet Protocol operates at the Network layer. It is a stateless and unreliable protocol responsible for packet addressing and network routing. Key characteristics include tracking the Time to Live (TTL) and defining source and destination IP addresses.

### Transmission Control Protocol (TCP)
TCP is a connection-oriented transport protocol that guarantees reliable, ordered delivery of data packets. Essential features include:
- The 3-Way Handshake for connection setup.
- Sequence numbers to order packets correctly.
- Acknowledgments to confirm packet receipt.
- Automatic retransmission of lost packets.
- Congestion control algorithms.
- Flow control mechanisms utilizing sliding windows.

### User Datagram Protocol (UDP)
UDP is a connectionless transport protocol that prioritizes speed over reliability. It does not provide delivery guarantees, making it suitable for applications that can tolerate packet loss, such as DNS lookups, Voice over IP (VoIP), and live video streaming.

### Transport Layer Security (TLS)
TLS operates to secure communication over potentially insecure networks. By using asymmetric encryption (for establishing connections securely) and exchanging X.509 certificates, TLS allows the client and server to negotiate symmetric session keys. It is the core technology behind HTTPS.

### HTTP Versions
The Hypertext Transfer Protocol (HTTP) has evolved significantly over time to address performance bottlenecks:

| Version | Key Features |
|---|---|
| **HTTP/1.1** | Text-based protocol. Supports connection keep-alive for reusing a single connection per request. |
| **HTTP/2** | Binary protocol. Introduces multiplexed streams, resolving head-of-line blocking at the application layer. |
| **HTTP/3** | Operates over QUIC (a transport protocol built on UDP). Provides faster connection establishment and built-in security. |

## 4. Domain Name System (DNS)

The Domain Name System functions as the contact directory of the internet, mapping human-readable domain names to numerical IP addresses. 
The system relies heavily on both recursive resolvers (which find the answer for the client) and authoritative name servers (which hold the actual DNS records). The hierarchical structure ensures distributed query handling across Root servers, TLD servers (e.g., .com, .net), and domain-specific authoritative servers.

## 5. Additional Resources

- *Computer Networking: A Top-Down Approach* by James F. Kurose and Keith W. Ross
