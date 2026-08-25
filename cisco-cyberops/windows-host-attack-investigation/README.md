# Windows Host Attack Investigation

> Cisco Networking Academy — CyberOps  
> Lab 27.2.16: Investigating an Attack on a Windows Host

In this lab, I investigated alerts from a Windows machine that had been infected with malware.

I followed the traffic from the first alerts to the download of executable files, Remcos RAT communication and later activity related to Dridex.

The idea was to understand what happened to the machine and separate normal Windows traffic from what was actually suspicious.

## Investigation Flow

`DNS Alert` → `test1.exe Download` → `Trojan Identified` → `Remcos RAT Check-in` → `C2 Communication` → `F4.exe Download` → `Dridex SSL Alerts`

## Tools Used

`Security Onion` `Sguil` `Wireshark` `Kibana` `Zeek` `Cisco Talos` `VirusTotal`

## Investigation

The questions below are a selection from the original lab, focusing on the parts that I found most relevant during the investigation.

### Initial Alert Analysis

> **Do you think this alert was the result of an IDS misconfiguration or a legitimate suspicious communication?**

It looked like a configuration problem. The DNS update was between IP addresses from the same internal network, but the IDS reported it as a DNS update coming from the external network. Because of this, I treated it as a false positive.

> **What is the hostname, domain name, and IP address of the source host in the DNS update?**

The hostname was Bobby-Tiger-PC, the domain was littletigers.info and the source IP was 10.0.90.215.

---

### Executable Download

> **What was the request for?**

It was an HTTP GET request for test1.exe. In this case, the IP 10.0.90.215 was requesting the executable from the server 209.141.34.8.

> **Did Talos recognize the file hash and identify it as malware?**

Yes. Talos identified the SHA256 hash as a Trojan.

---

### Remcos RAT Activity

> **What is the destination port of the communication? Is it a well-known port?**

The destination port was 2404. It is not a very common port, so it could indicate communication specific to the malware.

> **Is the communication readable or is it encrypted?**

The communication was encrypted, so the content could not be read clearly from the transcript.

> **What does Remcos stand for?**

Remcos stands for Remote Control and Surveillance Software. It is used for remote control and surveillance.

> **What type of communication do you think was being transmitted?**

It looked like communication between the infected machine and a command-and-control server. In other words, the machine could already be sending information outside or receiving commands.

> **What type of encryption and obfuscation was used to bypass detection?**

RC4 encryption, Base64 encoding and packers were used to try to hide the traffic and make detection more difficult.

---

### Second Executable and Dridex Activity

> **From which server IP address and port number was the file downloaded?**

The file was downloaded from 217.23.14.81 using port 80.

> **What is the name of the file that was downloaded?**

The file was F4.exe.

> **Is the executable known malware and what is the AMP detection name?**

Talos identified it as a Trojan and the AMP Detection Name was Win.Dropper.Cridex::1201.

> **How are the remaining three alerts related?**

The three alerts involved suspicious SSL traffic linked to certificates on a blacklist. Dridex also appeared in the alerts, so they seemed to be related to activity happening after the machine was already infected.

---

### Kibana and Zeek Analysis

> **What are the CSPCA.crl and ncsi.txt files related to?**

CSPCA.crl was related to Microsoft certificates and ncsi.txt is used by Windows to check if there is Internet connectivity. Because of this, they looked like normal Windows requests and not part of the malware activity.

> **Did any of the domains seem potentially unsafe?**

Yes. The domain toptoptop1.online looked suspicious. VirusTotal identified it as malicious, so it appeared to be related to the malware.

## Key Findings

| Finding | Result |
|---|---|
| Affected host | **Bobby-Tiger-PC** |
| Host IP | **10.0.90.215** |
| Domain | **littletigers.info** |
| First executable | **test1.exe** |
| First file classification | **Trojan** |
| Remote access malware | **Remcos RAT** |
| Communication port | **2404** |
| Second executable | **F4.exe** |
| AMP detection | **Win.Dropper.Cridex::1201** |
| Additional activity | **Dridex-related SSL alerts** |
| Suspicious domain | **toptoptop1.online** |

## What Happened

The investigation started with alerts from Bobby-Tiger-PC. The first DNS alert looked like a false positive, but after checking the next alerts I found that the machine had requested test1.exe from an external server.

The hash of the file was checked in Cisco Talos and it was identified as a Trojan. Later, Remcos RAT appeared in the alerts and the machine was communicating through port 2404 using encrypted traffic. This looked like communication between the infected machine and a command-and-control server.

Another executable, F4.exe, was also downloaded and Talos identified it as a Trojan with the AMP Detection Name Win.Dropper.Cridex::1201. The last alerts also showed suspicious SSL traffic related to Dridex.

In Kibana and Zeek I could also see that not everything in the traffic was malicious. Some requests were normal Windows activity, while others, such as the domain toptoptop1.online, were related to the malware.
