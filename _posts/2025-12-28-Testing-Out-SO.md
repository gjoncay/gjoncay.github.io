---
title: Confirming Security Onion Functions
date: 2025-12-28
author: Grant Oncay
layout: post
tags:
  - Virtual Machines
  - Traffic Mirroring
  - NSM
  - Security Onion
category: Set-up
---
## Testing Out Security Onion

Now that the Security Onion standalone machine seems to be properly detecting network traffic in the LAN, I wanted to ensure some of the NSM components were functioning properly as well. After generating some basic web traffic on the Ubuntu desktop in the LAN and seeing event generation in Elastic, I moved on to ensuring the server was generating PCAPs and that Zeek and Suricata were correctly processing traffic. 

On the Security Onion standalone server, I navigated to /nsm/suripcap/ and saw PCAPs generating. I also made sure to verify on the SO GUI that the sensor was set to capture all PCAPs, rather than PCAPs based on event triggers. With the SO standalone appearing to correctly capture all data, I ran:
```
curl -sSL https://raw.githubusercontent.com/0xtf/testmynids.org/master/tmNIDS -o /tmp/tmNIDS && chmod +x /tmp/tmNIDS && /tmp/tmNIDS
```
![TtmNIDS](assets/img/tmNIDS.png)

I ran several of the tests, and navigated back to the alerts tab on SO. Everything seemed to be functioning properly, as I had triggered corresponding events from those tests run on the Ubuntu desktop in the LAN.

![SO Alerts](assets/img/SO_alerts.png)

Finally, though alerts are triggering (and they have PCAP files from the standalone in their metadata), the SO GUI is still showing "incomplete" on the PCAP dash, preventing me from analyzing full packet data. Additionally, under "Grid", the standalone sensor is tagged as pending.

More troubleshooting to follow!
