# Snort and Firewall Response

> Cisco Networking Academy — CyberOps  
> Lab 26.1.7: Snort and Firewall Rules

In this lab, I used Snort to detect traffic going to a simulated malicious server and then used iptables to block that communication.

The test started with H5 downloading W32.Nimda.Amm.exe from a server running on port 6666. While the download was happening, I monitored the Snort alerts on R1 and captured the traffic with tcpdump.

After identifying the server and port involved, I added a firewall rule on R1 and tried the download again to see if the connection was blocked.

## Detection and Response Flow

`File Download` → `Snort Alert` → `Traffic Capture` → `Server Identified` → `iptables Rule` → `Connection Blocked`

## Tools Used

`Mininet` `Snort` `tcpdump` `iptables` `wget` `netstat`

## Investigation

Below are some of the main questions from the lab and what I found during the exercise.

### Snort Detection

> **What port was used when communicating with the malware web server?**

The port was 6666. I could see this in the wget command because the address appeared as 209.165.202.133:6666. The same server and port also appeared in the Snort alert.

> **Was the file completely downloaded?**

Yes. On H5 the download reached 100% and W32.Nimda.Amm.exe was saved.

> **Did the IDS generate an alert related to the file download?**

Yes. Snort generated a "Malicious Server Hit!" alert. This happened because Snort was analyzing the traffic passing through R1 and detected a signature associated with the malicious traffic.

> **What source and destination IP addresses appeared in the alert?**

The source IP was 209.165.200.235 and the destination IP was 209.165.202.133.

> **What message was recorded by the IDS signature?**

The message was "Malicious Server Hit!".

---

### Traffic Capture

> **How can the PCAP file be useful to a security analyst?**

The PCAP saved the traffic captured while the malicious file was being downloaded. With this, I could check the IP addresses, ports, protocols used and the sequence of the communication. The capture could also be opened later in Wireshark for a more detailed investigation.

The capture was saved as:

nimda.download.pcap

---

### Firewall Response

> **What chains were being used by iptables on R1?**

The chains were INPUT, FORWARD and OUTPUT.

In this case, the traffic was coming from another machine and passing through R1, so the blocking rule had to be added to the FORWARD chain.

The rule used was:

    iptables -I FORWARD -p tcp -d 209.165.202.133 --dport 6666 -j DROP

This blocks TCP traffic going to 209.165.202.133 on port 6666.

> **Was the download successful after adding the firewall rule?**

No. wget tried to connect to 209.165.202.133 on port 6666 again, but this time the connection timed out.

This happened because the iptables rule was dropping TCP traffic going to that server and port, so the file could no longer be downloaded.

> **What would be a more aggressive way of blocking the server?**

A more aggressive option would be to block all traffic going to 209.165.202.133 instead of only blocking traffic to port 6666.

## Key Findings

| Finding | Result |
|---|---|
| Malicious server | **209.165.202.133** |
| Server port | **6666/TCP** |
| Downloaded file | **W32.Nimda.Amm.exe** |
| IDS | **Snort** |
| Snort alert | **Malicious Server Hit!** |
| Traffic capture | **nimda.download.pcap** |
| Firewall | **iptables** |
| Firewall chain | **FORWARD** |
| Firewall action | **DROP** |
| Final result | **Connection blocked** |

## What Happened

H5 downloaded W32.Nimda.Amm.exe from the simulated malicious server at 209.165.202.133 on port 6666.

While the traffic was passing through R1, Snort generated a "Malicious Server Hit!" alert. I also captured the download traffic with tcpdump and saved it to a PCAP file.

After identifying the server, I added an iptables rule to the FORWARD chain to drop TCP traffic going to 209.165.202.133 on port 6666.

I tried the download again after adding the rule. This time wget could not connect to the server and the connection timed out, showing that the firewall rule was working.
