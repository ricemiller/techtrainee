---
title:      "Minotaur WriteUp"
date:       2016-11-01 19:39:00
tags: [pentesting,writeup,vulnhub,minotaur]
---
This week we will take up another [vulnhub](https://www.vulnhub.com) machine. <!-- Set preview -->


## Preparation
As with [the previous one](/2016/SickOS1.1-WriteUp/), we just have to [download the virtual machine](https://www.vulnhub.com/entry/sectalks-bne0x00-minotaur,139/) and add the resource to our hypervisor. Unfortunately, __this time things may not work just out of the box__. As *Minotaur* description states, __the machine must be inside the 192.168.56.0/24 network__, so I'm afraid that VMWare Player won't be valid this time as some advanced settings must be changed.

On __VMWare Workstation__ (don't panic __VirtualBox users__, there is also a solution for you creating a new NAT network on *Preferences > Network > NAT Networks* and editing the __Network CIDR__), we go to *Edit > Virtual Network Editor* (sudo password may be asked) and select *Add Network*. We select __NAT Network__ and *Add*.

{% include image.html
            img="images/minotaur/minotaur1.png"
            title="Settings for the new network"
            caption="Settings for the new network" %}

The only remaining thing to do is __changing the *subnet IP* and the *gateway IP*__, this last one after clicking on *NAT Settings* (see below).


{% include image.html
            img="images/minotaur/minotaur2.png"
            title="Changing the subnet IP"
            caption="Changing the subnet IP" %}

{% include image.html
            img="images/minotaur/minotaur3.png"
            title="Changing the gateway IP"
            caption="Changing the gateway IP" %}

After setting the network interfaces of both victim and attacker for using this new network, we are ready for the action!

## Reconnaissance
They say that you cannot teach an old dog new tricks, but maybe the dog just prefers to sticks to the old but reliable tricks under its sleeve (or paw?). Via __*netdiscover*__ we find out the address of our Minotaur.

{% include image.html
            img="images/minotaur/minotaur4.png"
            title="netdiscover -r 192.168.56.0/24"
            caption="Discovering our victim inside the network" %}

## Scanning
We continue with our old habits and perform a full __*nmap*__ scan against any TCP port on the machine.

{% include image.html
            img="images/minotaur/minotaur5.png"
            title="nmap -A -p- 192.168.56.223"
            caption="Preliminary scan with nmap" %}

The scan reveals the following:
* OS: Linux 3.2 - 4.4, probably __Ubuntu__
* An __SSH server__ is running (does not seem a version with disclosed vulnerabilities *yet*)
* __Apache 2.4.7 running on port 80__
* __vsFTPd 2.0.8 service__ with allowed anonymous access, also no quick vulnerabilities found

As a rule of thumb, an intruder will usually head to the lower hanging fruit, so we will address the web server first.

### Web Server
We find ourselves with a __default Apache2 welcome page__, so there is a high possibility that some subdomain contains something juicy for us.

{% include image.html
            img="images/minotaur/minotaur6.png"
            title="Default Apache welcome page"
            caption="It works!" %}

In order to discover available subdirectories, we will use a tool called __*dirb*__.

{% include image.html
            img="images/minotaur/minotaur7.png"
            title="dirb http://192.168.56.223 /usr/share/wordlists/dirb/big.txt"
            caption="Using dirb to discover new paths inside the web application" %}

We find out a new web running under the */bull* path, so let's check it out.

### WordPress
We found a blog about bulls! Pretty appropriate, regarding the name of the machine.

{% include image.html
            img="images/minotaur/minotaur8.png"
            title="Bull Blog"
            caption="Blog powered by WordPress" %}

The web hosted appears to be a __WordPress blog__, so __*wpscan*__ will make wonders out of this.

{% include image.html
            img="images/minotaur/minotaur9.png"
            title="wpscan http://192.168.56.223/bull"
            caption="Scanning the WordPress application" %}

Between all the vulnerabilities found, one of them seems to allow uploading files (see below), so maybe we could try a reverse TCP connection.

{% include image.html
            img="images/minotaur/minotaur10.png"
            title="Slideshow Gallery < 1.4.7 Arbitrary File Upload"
            caption="Vulnerability we will focus on" %}



## Intrusion


## Flag


## Conclusions


<iframe width="560" height="315" src="https://www.youtube.com/embed/4DHHv5OuEzY" frameborder="0" allowfullscreen="1"></iframe>
