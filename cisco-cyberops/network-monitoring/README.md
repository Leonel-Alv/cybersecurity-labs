# Network Monitoring with Syslog, AAA and NetFlow

> Cisco Networking Academy — CyberOps  
> Lab: Packet Tracer - Logging from Multiple Sources  
> Additional Lab: Packet Tracer - Explore a NetFlow Implementation

In these labs I used Cisco Packet Tracer to see how Syslog, AAA and NetFlow can be used to monitor network activity.

I generated different types of traffic and checked the information that was recorded, including network device logs, user login activity and NetFlow records.

## Project Flow

`Network Activity` → `Syslog` → `AAA Accounting` → `NetFlow` → `Traffic Analysis`

## Tools Used

`Cisco Packet Tracer` `Syslog` `AAA` `TACACS+` `NetFlow` `EIGRP`

## Syslog Monitoring

I started with Syslog.

The network devices were already configured to send their logs to the Syslog server. This included R1, R2, the Core Switch and the Firewall.

On R1, I used:

    enable
    debug eigrp packets

This generated EIGRP debug messages that I could then see on the Syslog server.

In the logs I could see information such as the interface, IP, flags and sequence. I could also see the date and time of the event and the IP of the device that sent the message.

This was useful because I could check messages from different network devices in the same place.

---

## AAA and TACACS+ Accounting

After that, I tested user access logging with AAA and TACACS+.

I logged in to R2 using the credentials provided in the lab and then checked the AAA Accounting records.

The log showed:

- Date and time
- Username
- Event type
- Device IP
- Access port

When I logged in, the event appeared as:

    Start

I then used:

    logout

After logging out, another record appeared with:

    Stop

So in this case I could clearly see when the user started and ended the session.

---

## NetFlow Monitoring

In the second lab I worked with the NetFlow Collector.

I started by pinging the default gateway from PC-1:

    ping 10.0.0.1

The flow record showed:

| Field | Value |
|---|---|
| Source IP | **10.0.0.10** |
| Destination IP | **10.0.0.1** |
| IP Protocol | **1 — ICMP** |
| Packets | **4** |
| Bytes | **512** |
| Input Interface | **G0/0** |

With this record I could see the source and destination IPs, the protocol being used, the interface and also the number of packets and bytes.

---

## Comparing Flows

I then generated more traffic using the other PCs.

Since the previous ping came from PC-1, I expected a ping from PC-2 to create another flow because it had a different source IP.

When I repeated the same ping from PC-1 to the same gateway, I expected the existing flow record to be updated instead of creating another one for the exact same communication.

I also generated pings from PC-3 and PC-4 to compare the different traffic in the collector.

---

## HTTP Flow Analysis

Next, I tested HTTP traffic.

PC-1 was using:

    10.0.0.10

The Web Server was using:

    192.0.2.100

The destination port for HTTP was:

    80

After opening the website, the NetFlow Collector showed separate flows for the request and the response.

I then opened the Copyrights page on the same Web Server.

This created two new flows. The first connection had used source port `1025`, while the new connection used `1026`.

The PC and Web Server were still the same, but because a different source port was used, it was a different connection.

---

## TCP Flags

I also compared the TCP flags in the HTTP flows.

Some of the values I found were:

    0x02 → SYN
    0x12 → SYN + ACK
    0x10 → ACK

The number of packets and bytes was also different between the request and response flows.

This makes sense because the request and response are part of the same communication, but they do not contain the same amount of data.

---

## DNS and HTTP Traffic

For the last test, I accessed the Web Server using:

    www.example.com

instead of using its IP address directly.

I expected to see the HTTP traffic again, but this time also with DNS traffic because the name first had to be resolved to an IP address.

In the NetFlow records I found:

| IP Protocol | Protocol |
|---|---|
| **6** | **TCP** |
| **17** | **UDP** |

Protocol `6` was TCP, which was being used for the HTTP communication.

Protocol `17` was UDP, which was being used for DNS.

So when I accessed the website using its name, I could see both the DNS and HTTP traffic.

## Key Findings

| Test | Result |
|---|---|
| Syslog messages | **Received from multiple network devices** |
| EIGRP debugging | **Generated Syslog messages** |
| AAA login | **Recorded as Start** |
| AAA logout | **Recorded as Stop** |
| ICMP | **Protocol 1** |
| TCP | **Protocol 6** |
| UDP | **Protocol 17** |
| HTTP traffic | **Request and response appeared as separate flows** |
| Different source ports | **Created different connections** |
| TCP flags | **SYN, SYN + ACK and ACK identified** |

## What I Learned

With these labs I learned how Syslog, AAA and NetFlow can give different information about what is happening on a network.

With Syslog I could see messages coming from different network devices. With AAA accounting I could see when a user logged in and logged out.

With NetFlow I could see information about the traffic such as source and destination IPs, ports, protocols, interfaces, packets and bytes.

I also understood better why two connections between the same devices can still appear as different flows when the ports are different.

Finally, when I accessed the website using its name instead of the IP address, I could also see the DNS traffic used before the HTTP communication.
