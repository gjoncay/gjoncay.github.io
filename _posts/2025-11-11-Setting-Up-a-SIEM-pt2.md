---
title: Setting Up a SIEM (Pt. 2)
date: 2025-11-11
author: Grant Oncay
layout: post
tags:
  - Virtual Machines
  - SIEM
  - Wazuh
category: Set-up
---
## Installing Agents
To install an agent, I followed the quick wizard found within the web dashboard. After selecting the package to download, setting the server address (the address of the server running Wazuh), and setting some optional fields such as the agent name and group, I ran the command that the Wazuh agent tool produces. 

![Wazuh Agent Setup](assets/img/agent-1.png)

Though everything seemed to work initally, I did get an error upon attempting to start the agent. The first two `systemctl` commands ran without issue, but I was notified that I did not have an address set for 'IP_MANAGER' on the third:

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

I had to dig around a little bit, but eventually found that, despite running the command provided by the agent installation tool, one of the agent's .conf files did not properly set the value for 'IP_MANAGER' (leaving it as IP_MANAGER rather than an actual address). I simply edited the value with nano to the IP of the SIEM server, and ran:

```bash 
sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent
```

Everything looked good on the machine installing the agent at that point.

![Wazuh Agent Setup](assets/img/agent-2.png)


## Verifying on the Dashboard
I then moved over the the desktop environment that will be used to monitor the network, and confirmed that the agent installed on 'ubuntu1' was correctly registered and sending telemetry to the SIEM. I think my next steps will be to install an additional agent on the other ubuntu machine as well as segment the network so that it better reflects a live environment.

![Wazuh Dashboard](assets/img/agent-3.png)
