---
title: Setting Up a SIEM (Pt. 3) and Examining a Vuln
date: 2025-11-12
author: Grant Oncay
layout: post
tags:
  - Virtual Machines
  - SIEM
  - Wazuh
category: Set-up
---
## Adding Another Agent
I picked up where I left off last by repeating the process to add an agent to the other Ubuntu server box running in the network. Just like the previous post, I followed the prompts in the Wazuh dashboard to generate the required code to run on the server itself. Again, I selected DEB amd64 for the system, kept the same IP for the server running Wazuh, and added a name for the agent (leaving it in the default group).

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.0-1_amd64.deb && sudo WAZUH_MANAGER='192.168.100.224' WAZUH_AGENT_NAME='ubuntu2' dpkg -i ./wazuh-agent_4.14.0-1_amd64.deb
```

![Wazuh Agent Setup](assets/img/agent_creation.png)

![Wazuh Agent Created](assets/img/agent_created.png)

After running the following, the agent appeared on my Wazuh dashboard without issue; the MANAGER_IP value failing to properly set must have been a typo with the code I ran last time.

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
sudo systemctl status wazuh-agent
```
![Two Running Agents](assets/img/2_agents.png)

## Examining the Agents
With these two agents running on my two Ubuntu VMs, I wanted to take a preliminary look at some of their vulnerabilities. 

Right off the bat, Wazuh detected numerous high severity issues. Of course, in a lab environment, this is a great chance to jump in and learn rather than freak out over potential threats to these machines.

I started with [CVE-2025-9841](https://nvd.nist.gov/vuln/detail/CVE-2025-8941). I think it will be helpful to read through as much CVE documentation as I can over the next year and a half-ish. Getting confident with the language will be beneficial at a minimum. More useful will be exposing myself to as many methods of attack and exploitation. These CVE descriptions always have a variety of jargon/technical concepts/etc. Even if I am aware of a term or concept at a high-level, it's helpful to dig into them and really try to work out what the impact is of this particular vulnerability, how to mitigate it, and if it truly is a threat to this environment specifically. After all, a severity of 'high' is just a label, and doesn't mean much without context.

## Examining CVE-2025-8941
Continuing on, I read up on CVE-2025-8941. Per NIST, this flaw involves the exploitation of symbolic links in the pam_namespace module, which can allow a local user privesc to root opportunities. This is a good chance to walk through the corresponding CVSS as well (it's been a minute since I took CySA+!)

At an overall score of 7.8, CVE-2025-9841 breaks out to AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H. In plain english, an attacker must have local access to the machine, no special conditions are required, the attacker only needs basic privileges, no user interaction is needed, exploitation of this particular vuln doesn't affect any other parts of the system, and all three components of the CIA triad are impacted at a high level.

Reading about the pam_namespace module was helpful. Per [man7.org](https://man7.org/linux/man-pages/man8/pam_namespace.8.html), the pam_namespace module "disassociates the session namespace from the parent namespace..." More simply, it sets up user-specific directories that are isolated to that user (such as /tmp when multiple users are on the system).

While proof of concept code exists, it did not look like there was any record of exploitation in the wild (at least that I could find).
Exploitation of this vuln would involve creating a race condition (TOCTOU) by creating a path that is checked by the pam module but then changed to a symbolic link before the path is actually acted upon, allowing privesc to root.

To ensure the system had an updated pam module, I ran:

```bash
apt list --upgradeable 2>/dev/null | grep -i pam
```

which indicated a non-current pam module, and upgraded with:
```bash
sudo apt full-upgrade -y
```

I was not really familiar with the purpose of `full-upgrade` as opposed to the standard `apt upgrade`, and found that it installs new dependencies and removes obsolete packages, unlike it's simpler brother.

With that, I'll end this post, and keep working through building familiarity with Wazuh in follow-on entries. There is a ton to learn from even a single CVE - looking at 2025-8941 supported the following:
- Review of CVSS meaning and 'so what'
- Understanding of an important Linux module (especially with security impact)
- a bit on linux commands and updating a system
- Review of a broader attack type (race conditions/TOCTOU)

Looking forward to continuing to work on this lab!
