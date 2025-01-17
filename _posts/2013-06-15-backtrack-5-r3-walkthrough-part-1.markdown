---
layout: post
title: "Backtrack 5 R3 Walkthrough part 1"
date: 2013-06-15 03:00
comments: true
tags: [security, backtrack]
---

Backtrack is one of the most popular Linux distributions used for Penetration testing and Security Auditing. The Backtrack development team is sponsored by Offensive Security. On 13th August 2012, Backtrack 5 R3 was released. This included the addition of about 60 new tools, most of which were released during the Defcon and Blackhat conference held in Las Vegas in July 2012\. In this series of articles, we will look at most of the new tools that were introduced with Backtrack 5 R3 and look at their usage. Some of the notable changes included tools for mobile penetration testing, GUI tools for Wi-fi cracking and a whole new category of tools called Physical Exploitation.

<!--more-->

## Getting Backtrack 5 R3

There are two ways to get up and running quickly with Backtrack 5 R3\. If you are already running Backtrack 5 R2, you can upgrade to Backtrack 5 R3 by following the steps described on this [page](http://www.backtrack-linux.org/backtrack/upgrade-from-backtrack-5-r2-to-backtrack-5-r3/). Or you can do a fresh install of Backtrack 5 R3 from the [downloads](http://www.backtrack-linux.org/downloads/) section on Backtrack's official website.

A list of the new tools released with Backtrack 5 R3 according to Backtrack's official website are libcrafter, blueranger, dbd, inundator, intersect, mercury, cutycapt, trixd00r, artemisa, rifiuti2, netgear-telnetenable, jboss-autopwn, deblaze, sakis3g, voiphoney, apache-users, phrasendrescher, kautilya, manglefizz, rainbowcrack, rainbowcrack-mt, lynis-audit, spooftooph, wifihoney, twofi, truecrack, uberharvest, acccheck, statsprocessor, iphoneanalyzer, jad, javasnoop, mitmproxy, ewizard, multimac, netsniff-ng, smbexec, websploit, dnmap, johnny, unix-privesc-check, sslcaudit, dhcpig, intercepter-ng, u3-pwn, binwalk, laudanum, wifite, tnscmd10g bluepot, dotdotpwn, subterfuge, jigsaw, urlcrazy, creddump, android-sdk, apktool, ded, dex2jar, droidbox, smali, termineter, bbqsql, htexploit, smartphone-pentest-framework, fern-wifi-cracker, powersploit, and webhandler. We will be discussing most of these tools in this series.

## Fern-Wifi-Cracker

Fern Wi-fi cracker is a program written in python that provides a GUI for cracking wireless networks. Normally, you need to run aireplay-ng, airodump-ng and aircrack-ng separately in order to crack wireless networks, but Fern-Wifi-cracker makes this job very simple for us by acting as a facade over these tools and hiding all the intricate details from us. It also comes with a bunch of tools that helps you perform attacks like Session Hijacking, locate a particular system's geolocation based on its Mac address etc.

Fern Wi-fi cracker can be found under the category Wireless Exploitation tools as shown in the figure below.

![1]({{site.baseurl}}/images/posts/bt5r1/1.png)

Before starting with Fern Wi-fi cracker, it is important to note that you have a Wi-fi card that supports packet injection. In my case, i am running Backtrack 5 R3 as a VM and i have connected an external Alfa Wi-fi card to it. You can verify if your card can be put into monitor mode by just typing airmon-ng and it will show you the list of interfaces that can be put in monitor mode. Once this is done, open up Fern Wi-fi cracker.

![2]({{site.baseurl}}/images/posts/bt5r1/2.png)

Select the appropriate interface on which you want to sniff on.

![3]({{site.baseurl}}/images/posts/bt5r1/3.png)

Once you have selected it, it will automatically create a virtual interface (mon0) on top of the selected interface (wlan0) as is clear from the image below.

![4]({{site.baseurl}}/images/posts/bt5r1/4.png)

Now, click on "Scan for access points". As you can see from the results, it found 4 networks with WEP and 1 network with WPA.

![5]({{site.baseurl}}/images/posts/bt5r1/5.png)

In this case, we will be cracking a WEP network named "Infosec test" which i set up for testing purposes. Click on the network "Infosec test" and it will show you its specific information like the BSSID of the access point, the channel on which the Access point is transmitting on etc. On the bottom right, you can select from a variety of attacks like the Arp request replay attack, caffe latte attack etc. In my case, i will be going for an Arp request replay attack. Once this is done, click on "Wi-fi attack" and this will start the whole process of cracking WEP.

You will now see that some IV's are being captured as shown in the image below. The tool will also tell you if your card is injecting arp packets properly or not as shown in the bottom right section of the image below.

![6]({{site.baseurl}}/images/posts/bt5r1/6.png)

Once enough IV's have been collected, it will start cracking the WEP key automatically.

![Wep]({{site.baseurl}}/images/posts/bt5r1/wep.png)

Similarly, Fern Wi-fi cracker can be used to crack WPA. It just makes the whole process so simple for us. It also provides some extra functionality for hijacking sessions and locating a computer's geolocation via its Mac address. I recommend you check it out.

## Dnmap

Imagine you have to scan a huge network containing thousands of computers. Scanning via nmap from a single computer will take quite a long time. In order to solve this problem, Dnmap was created. Dnmap is a framework which follows a client/server architecture. The server issues nmap commands to the clients and the clients execute it. In this way, the load of performing such a large scan is distributed among the clients. The commands that the server gives to its clients are put in a command file. The results are stored in a log file which are saved on both the server and the client. The whole process of running Dnmap follows these steps.

1.  Create a list of commands that you want to run and store it in a file, say commands.txt. Note the IP address of the server.
2.  Run the dnmap server and give the commands file as an argument.
3.  Connect the clients to the server. Note that the server should be reachable from the client.

Let's do the demo now. I have 2 virtual machines both running Backtrack 5 R3\. I am going to run the Dnmap server on one of the virtual machines and a client on the second one.

Open dnmap under the category Information Gathering --> Network Analysis --> Identify Live hosts. The next step is to create a commands.txt file. As you can see from the image below, i have 3 commands in the commands.txt file.

![7]({{site.baseurl}}/images/posts/bt5r1/7.png)

Now type the command as shown in the image below to start the dnmap server. I have started the dnmap server to listen on port 800\. As you can see, it currently detects no clients. Hence the next step is to get some clients to connect to this dnmap server. Also, it is better to specify the location of the log file that will be holding all the results.

![8]({{site.baseurl}}/images/posts/bt5r1/8.png)

On my other BT machine, i run the following command to connect the client to the server. Note that the internal IP address of my dnmap server is 10.0.2.15 and since my other virtual machine is also in the same internal network, it is able to reach to the server. You also need to specify the port to which you are connecting to on the server. Also, it is optional to specify an alias for the client.

![9]({{site.baseurl}}/images/posts/bt5r1/9.png)

Once the client establishes connection with the server, you will see that the client starts executing the commands that it is getting from the server.

![10]({{site.baseurl}}/images/posts/bt5r1/10.png)

On the server side, you will notice that it recognizes this client and shows it in the output. It also keeps giving you regular information like the number of commands executed, uptime, online status etc.

![11]({{site.baseurl}}/images/posts/bt5r1/11.png)

Once the scans are completed, dnmap stores the results in a directory named nmap_output. The results are saved in .nmap, .gnmap and xml formats. There are separate output files for each command. It is advisable to clear all the previous files in the nmap_output directory or save them somewhere else before starting a new scan. Here is what a sample response file looks like.

![12]({{site.baseurl}}/images/posts/bt5r1/12.png)

In this article, we looked at a couple of the most popular tools that were introduced with Backtrack 5 R3\. In further articles in this series, we will be discussing about many other new tools that were shipped with Backtrack 5 R3\. If there is a particular tool that you want me to write about or if you have any questions, comments, suggestions regarding this series, please write them down in the comments below.

## References:

*   Upgrade from Backtrack 5 R2 to R3  
    [http://www.backtrack-linux.org/backtrack/upgrade-from-backtrack-5-r2-to-backtrack-5-r3/](http://www.backtrack-linux.org/backtrack/upgrade-from-backtrack-5-r2-to-backtrack-5-r3/)

*   Fern-Wifi-Cracker  
    [http://code.google.com/p/fern-wifi-cracker/](http://code.google.com/p/fern-wifi-cracker/)

*   Dnmap framework official page  
    [http://sourceforge.net/projects/dnmap/](http://sourceforge.net/projects/dnmap/)

This article was originally published on the [resources](http://resources.infosecinstitute.com/) page at [Infosec Institute](http://infosecinstitute.com/). For more information, please visit my author [page](http://resources.infosecinstitute.com/author/prateek/).