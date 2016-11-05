---
title:      "Minotaur WriteUp"
date:       2016-11-01 19:39:00
tags: [pentesting,writeup,vulnhub,minotaur]
---
This week we will take up another [vulnhub](https://www.vulnhub.com) machine. Not very high skill levels will be required, but knowing the proper tools to use will be key for solving this challenge.

## Preparation
As with [the previous one](/2016/SickOS1.1-WriteUp/), we have to [download the virtual machine](https://www.vulnhub.com/entry/sectalks-bne0x00-minotaur,139/) and add the resource to our hypervisor. Unfortunately, __this time things may not work just out of the box__. As *Minotaur* description states, __the machine must be inside the 192.168.56.0/24 network__, so I'm afraid that VMWare Player won't be valid because some advanced settings must be changed.

On __VMWare Workstation__ (don't panic __VirtualBox users__, there is also a solution for you creating a new NAT network on *Preferences > Network > NAT Networks* and editing the __Network CIDR__), we go to *Edit > Virtual Network Editor* (sudo password may be asked) and choose **Add Network**. We select __NAT__ and press __Add__.

{% include image.html
            img="images/minotaur/minotaur1.png"
            title="Settings for the new network"
            caption="Settings for the new network" %}

The only remaining thing to do is changing the **subnet IP** and the **gateway IP**, this last one after clicking on **NAT Settings** (see below).


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
Via __*netdiscover*__ we find out the address of our Minotaur.

{% include image.html
            img="images/minotaur/minotaur4.png"
            title="netdiscover -r 192.168.56.0/24"
            caption="Discovering the victim inside the network" %}

## Scanning
We will continue with our old habits and perform a full __*nmap*__ scan against any TCP port on the machine.

{% include image.html
            img="images/minotaur/minotaur5.png"
            title="nmap -A -p- 192.168.56.223"
            caption="Preliminary scan with nmap" %}

The scan reveals the following:

* OS: Linux 3.2 - 4.4, probably __Ubuntu__
* An __SSH server__ is running (does not seem a version with disclosed vulnerabilities *yet*)
* __Apache 2.4.7__ running on __port 80__
* __vsFTPd 2.0.8 service__ with allowed anonymous access, also no quick vulnerabilities found

As a rule of thumb, an intruder will usually head towards the lower hanging fruits, so we will address to the web server first.

### Web server
We find ourselves with a __default Apache2 welcome page__, so there is a high possibility that some subdomain contains juicy things for us.

{% include image.html
            img="images/minotaur/minotaur6.png"
            title="Default Apache welcome page"
            caption="It works!" %}

In order to discover available subdirectories, we will use a tool called __*dirb*__. First attempt with the default dictionary offered no results, but the *big.txt* wordlist included in Kali got the job done.

{% include image.html
            img="images/minotaur/minotaur7.png"
            title="dirb http://192.168.56.223 /usr/share/wordlists/dirb/big.txt"
            caption="Using dirb to discover new paths inside the web application" %}

We find out a new web running under the */bull* domain, so let's check it out.

### WordPress
A blog about bulls! Pretty appropriate, regarding the name of the machine.

{% include image.html
            img="images/minotaur/minotaur8.png"
            title="Bull Blog"
            caption="Blog powered by WordPress" %}

The web hosted appears to be a __WordPress blog__, so __*wpscan*__ will make wonders out of this.

{% include image.html
            img="images/minotaur/minotaur9.png"
            title="wpscan http://192.168.56.223/bull"
            caption="Scanning the WordPress application" %}

Between all the vulnerabilities found, one of them seems to allow uploading files (see below), so maybe we could try some funky PHP script. The only problem with this vulnerability is that __we will require authorised access first__.

{% include image.html
            img="images/minotaur/minotaur10.png"
            title="Slideshow Gallery < 1.4.7 Arbitrary File Upload"
            caption="Vulnerability we will focus on" %}

### User and password obtention
Following with __*wpscan*__, now it is time to obtain users. The _enumeration_ option will return us an username called __bully__.

{% include image.html
            img="images/minotaur/minotaur11.png"
            title="wpscan --url http://192.168.56.223/bull --enumerate -u"
            caption="wpscan --url http://192.168.56.223/bull --enumerate -u" %}

I have to confess that for the password I had to peek in another walkthrough. Bruteforcing anything is always my last option for a challenge, as I think is more a matter of luck than ability. In spite of this, truth is that I did not know a thing about __wordlist generation__ or __password mutation__, so it was an interesting technique to acquire.

We will obtain the list of words that can be found on the web via __*cewl*__, and generate our wordlist of passwords with __*john*__. Finally, the bruteforcing option from __*wpscan*__ will give us the password.

{% include image.html
            img="images/minotaur/minotaur12.png"
            title="Password list generation and start of the bruteforcing attack"
            caption="Password list generation and start of the bruteforcing attack" %}

{% include image.html
            img="images/minotaur/minotaur13.png"
            title="Password acquisition"
            caption="Password acquisition: Bighornedbulls" %}

## Intrusion
Now that we have valid credentials for WordPress (they are not valid for the SSH or FTP servers, sorry!), we are going to exploit that plugin for uploading a web shell. It is as simple as login with our obtained credentials and, from the _Dashboard_, going to _Slideshow > Manage Slides > Add New_ and uploading our __PHP__ shell script. Surprisingly, not even the file extension was renamed (the __lack of file detection mechanisms__ goes without saying).

{% include image.html
            img="images/minotaur/minotaur14.png"
            title="Uploading the shell to the platform"
            caption="Uploading the shell to the platform" %}


Whatever shell you may upload, you will find it under the URL __*http://192.168.56.223/bull/wp-content/uploads/slideshow-gallery/*__, as it is an indexable directory.

{% include image.html
            img="images/minotaur/minotaur15.png"
            title="http://192.168.56.223/bull/wp-content/uploads/slideshow-gallery/php.php"
            caption="Web shell up and running on the server" %}

#### __Note:__
From here on, I used a __reverse TCP connection__ to the terminal, as the web shell was a bit uncomfortable. Moreover, __the web shell does not keep context between commands__, so changing directories and other actions were not possible.

### First flag
An easy flag found on the root of the web application. I happened to come across it just by jumping directories back and forth.

{% include image.html
            img="images/minotaur/minotaur16.png"
            title="cat /var/www/html/flag.txt"
            caption="First flag found" %}


### Second flag and shadow.bak file
Continuing with our inspection from this unprivileged user (*www-data*), we encounter interesting things on _/tmp_, like a __second flag__ or a __backup file of */etc/shadow*__

{% include image.html
            img="images/minotaur/minotaur17.png"
            title="cat /var/flag.txt"
            caption="Second flag found" %}

{% include image.html
            img="images/minotaur/minotaur18.png"
            title="cat /etc/shadow.bck"
            caption="Getting the shadow file" %}


### Cracking passwords with john
With this shadow file and the already accessible _/etc/passwd_ we will compose a unique file (__unshadow passwd shadow > crackme__) for our friend __*John the Ripper*__ and see if we can get some more credentials.

{% include image.html
            img="images/minotaur/minotaur19.png"
            title="john crackme"
            caption="More passwords acquired" %}

The previously crafted wordlist did not make the trick this time, but john's default mode got us __two more passwords__ (maybe it would have spat more afterwards, but actually I got the final flag before getting more results).

### SSH access and privilege escalation
Now we can access with both users and have some more fun. The user __heffer__ did not show anything special at first, but __minotaur__ surely did. Looking at the __sudo permissions__ this user has, we will discover that __we already have full access to root__.

{% include image.html
            img="images/minotaur/minotaur20.png"
            title="sudo su"
            caption="My name is minotaur, but my friends call me root" %}


## Final flag
The flag is now ours to take now.


{% include image.html
            img="images/minotaur/minotaur21.png"
            title="cat /root/flag.txt"
            caption="Final flag of the challenge" %}

## Conclusions
In my opinion this wasn't a very instructive machine. The whole challenge is basically about using the proper wordlist (with __*dirb*__, with __*wpscan*__, with __*john*__...), so my last feeling is that it was more a matter of being lucky than anything else. Of course that you need to know the tools in order to succeed, and of course that it is important to know when to use the right wordlist, but I sincerely don't have the impression of getting much of an outcome from this machine. 

Still, it is always good to keep on practising and face any variety of scenarios, so I will not consider it a waste of time either.


<iframe width="560" height="315" src="https://www.youtube.com/embed/KEfzmf8CiAs" frameborder="0" allowfullscreen="1"></iframe>
