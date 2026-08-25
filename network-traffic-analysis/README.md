# Network Traffic Analysis with Wireshark

> IEFP — Técnico/a Especialista em Cibersegurança  
> UC 1490: Network Traffic Analysis with Wireshark

In this project, I created two Linux virtual machines, one as a client and the other as a server, and used Wireshark to analyze the traffic between them.

I tested ICMP, HTTP, FTP and SSH. With FTP I was able to see the login information and commands in the captured traffic. When I repeated a similar test with SSH, the traffic was encrypted and the contents could not be read.

## Analysis Flow

`Client / Server Setup` → `ICMP` → `HTTP` → `FTP` → `SSH`

## Tools Used

`Wireshark` `Linux` `VirtualBox` `Apache` `vsftpd` `OpenSSH`

## Environment Setup

I created two Linux virtual machines, one for the client and another for the server. Both were on the same network and had different MAC addresses.

When I checked the IP configuration, the server was using 10.0.2.3. On the client, 10.0.2.3 was also assigned, together with 10.0.2.4.

Because 10.0.2.3 was present on both machines, I removed the duplicate address from the client:

    sudo ip addr del 10.0.2.3/24 dev enp0s3

After running ip a again, the server remained on 10.0.2.3 and the client was using 10.0.2.4.

![IP conflict identified and corrected](images/ip-conflict-resolution.png)

On the server I installed the services needed for the tests:

- Apache for HTTP
- vsftpd for FTP
- OpenSSH Server for SSH

---

## ICMP Traffic

I opened Wireshark on the server and filtered only ICMP traffic.

Then, from the client, I pinged the server.

In Wireshark I could see the Echo Requests going from the client to the server and the Echo Replies coming back.

![ICMP traffic captured in Wireshark](images/icmp-traffic.png)

This also confirmed that the two machines could communicate.

---

## HTTP Traffic

For the HTTP test, I accessed the Apache server from the client using:

    http://10.0.2.3/

At the same time, Wireshark was capturing HTTP traffic on the server.

In the capture I could see the communication between both IP addresses and the TCP handshake used to establish the connection.

![HTTP traffic and Apache server](images/http-traffic.png)

---

## FTP Traffic

For the FTP test, I connected from the client to the FTP server while Wireshark was capturing the traffic.

After logging in, I went back to Wireshark and used Follow FTP Stream.

![FTP credentials and commands visible in Wireshark](images/ftp-plaintext.png)

In the stream I could see the username and password entered from the client, as well as the FTP commands used during the session.

This showed that the FTP login information and commands were being sent in readable text.

---

## SSH Traffic

I then did a similar test using SSH.

From the client, I connected to the server, logged in, ran ls -l and then exited the session.

After that, I went back to Wireshark and followed the stream.

![Encrypted SSH traffic in Wireshark](images/ssh-encrypted.png)

This time the contents were encrypted and unreadable. Unlike FTP, I could not see the password or the commands from the SSH session.

## FTP vs SSH

| Protocol | What I could see in Wireshark |
|---|---|
| **FTP** | Username, password and FTP commands were readable |
| **SSH** | Session contents were encrypted and unreadable |

The difference was easy to see in Wireshark. FTP showed the login information and commands in readable text, while SSH protected the contents of the session.

## Key Findings

| Test | Result |
|---|---|
| Environment | **Two Linux VMs, client and server** |
| Duplicate IP | **10.0.2.3 was assigned to both machines** |
| After correction | **Server 10.0.2.3 / Client 10.0.2.4** |
| ICMP | **Echo Requests and Replies visible** |
| HTTP | **TCP handshake visible** |
| FTP | **Credentials and commands readable** |
| SSH | **Session contents encrypted** |
| Web server | **Apache** |
| FTP server | **vsftpd** |
| SSH server | **OpenSSH** |
| Main tool | **Wireshark** |

## What I Learned

This project gave me a practical view of what different protocols look like in Wireshark.

Before starting the traffic analysis, I had to correct the IP configuration because 10.0.2.3 was assigned to both machines. After removing the duplicate address from the client, the server remained on 10.0.2.3 and the client used 10.0.2.4.

With ICMP I could see the requests and replies between the two machines. With HTTP I could follow the connection between the client and the Apache server.

The FTP and SSH tests were the most useful comparison. In FTP I could read the login information and commands directly from the stream. With SSH, I could still see that traffic was being exchanged, but the contents were encrypted.

This made the difference between plaintext and encrypted communication much easier to understand.
