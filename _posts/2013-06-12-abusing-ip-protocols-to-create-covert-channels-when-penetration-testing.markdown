---
layout: post
title: "Abusing IP Protocols to Create Covert Channels when Penetration Testing"
date: 2013-06-12 04:18
comments: true
tags: [security, maintaining-access]
---

This article will talk about the maintaining access step in a penetration test. After an attacker has broken into the system and got access, escalated privileges etc, it is important for him to maintain his authority on the system so that he can access it at a later time. The exploited system could be a web server (directly accessible from the internet), or a system running inside a network with NAT, hence not directly accessible from the internet. The system could also be running in a network with a firewall that monitors incoming and outgoing packets, having filters set for different types of packets, protocols etc. There could be a number of different scenarios, and it is important from the attacker's perspective to maintain his access on the compromised host. In this article we will discuss all these cases, take up different real world scenarios and see all the different methods of bypassing those restrictions. So let's begin!

<!--more-->

As discussed previously the compromised system could be a web server directly accessible from the internet. In this scenario the attacker could install a web backdoor or a webshell on the system. The backdoor could be any type of file (php,jsp etc) which could be accessed from the internet. On accessing it through the browser the attacker can run commands on the compromised system through the interface provided by the file, browse files and directories etc. Or it could just be a malicious program which opens a port on the victim machine and allows access to the hacker from the outside. In some cases the victim machine could be running a firewall that monitors incoming and outgoing packets. For e.g. the firewall may not allow outgoing TCP connections from the compromised host, but allow other generic protocols like DNS (it is mandatory on most networks) and ICMP. The attacker could then use a technique called TUNNELING. Tunneling is the process of encapsulating one protocol into another protocol. The protocol which is encapsulated is called the payload protocol and the protocol which encapsulates this protocol is called the delivery protocol. Tunneling can be used for bypassing firewall restrictions and can even be used for providing a secure path over untrusted networks. They are also used for bypassing hotspot access controls.For e.g. one can encapsulate FTP traffic (which is plain text and can be seen by an eavesdropper) onto the HTTPS protocol, thereby making it unobservable to the listener. TCP protocol can also be encapsulated into DNS or ICMP protocol, thereby bypassing the firewall restrictions. Hence the techniques used in the maintaining access step of a penetration test can broadly be classified into three tags.

1.  Protocol Tunneling
2.  Web Backdoors
3.  OS Backdoors

In this article we will be discussing each of them in detail.

1)**Protocol Tunneling**- Protocol Tunneling is the process of encapsulating one protocol into another protocol .We will cover 2 types of Tunneling in this section.

## a)DNS Tunneling

In this exercise we will use Iodine (available by default in Backtrack 5) to encapsulate IPv4 traffic into DNS Traffic .But first let's understand how DNS Tunneling works.

![DNS Tunneling]({{site.baseurl}}/images/posts/Maintaining-Access//DNS-Tunneling.png)

1) The client (in this case our victim) uses a program (in this case Iodine) to encapsulate Ipv4 traffic into DNS traffic and sends the entire traffic to a subdomain (for e.g tunnel.example.com) controlled by us. Let's say the client(or victim) had tried to contact our control machine, through which we control the victim. Note that the control machine and the subdomain to which traffic is being sent are entirely different. The client sends the traffic to the control machine through our subdomain. Also, it is important to note that the subdomain should be running a DNS server.

2) After the traffic has reached tunnel.example.com, the iodine program running on our server will convert it into normal traffic and forward it to the queried domain (the control machine).

3) Once the iodine server gets a response back, it again encapsulates the normal traffic into DNS and sends it through the tunnel.The DNS traffic is allowed by the firewall and is able to reach the client. In this way, we are able to communicate with the victim machine.

## LAB SETUP

We will need a client running iodine (which in our case will be the victim , and will be one end of the tunnel), and a server running iodined (which will be the other end of the tunnel). The traffic between the client and server will appear to be DNS traffic (thereby bypassing restrictions set by the firewall), as we will see soon. Since Iodine comes preinstalled with Backtrack 5, we will be using 2 Backtrack 5 machines as our client and server. In this case we will be testing the DNS tunnel on a LAN.

The first step is to check if DNS traffic is actually allowed to pass through or not. For this just go to the terminal and try pinging any domain from the victim machine. If we get an IP-address in the reply section, then it means that DNS packets are allowed, because the query went to the DNS server and it returned back the IP-Address of the domain.

![Ping]({{site.baseurl}}/images/posts/Maintaining-Access//ping.png)

Ok Cool , so DNS packets are allowed on the network. The next step is to start the Iodine server on our server machine. Note that this machine acts as an intermediate between the victim and the control machine. The traffic is encapsulated onto DNS only between between the victim and the intermediate machine. For e.g let's say the victim is set to send requests to tunnel.infosecinstitute.com . The query will first reach the infosecinstitute.com server. If it has a A record for tunnel.infosecinstitute.com , the request will reach the tunnel.infosecinstitute.com server. Note that in order to direct the request for tunnel.infosecinstitute.com to your server, we should have an NS record in the zone file of infosecinstitute.com server. This server is the one running the Iodine program. This DNS server will then forward our requests to whichever domain we had queried for, get the response back, encapsulate it into DNS and then send the response back to the victim.

Type in the following command to start Iodine server

![1]({{site.baseurl}}/images/posts/Maintaining-Access//1.png)

a) We use ./iodined as we want to start the iodine server

b) -f specifies Iodine to run in foreground,"10.0.1.2" is used to set the IP-ADDRESS of the dns0 interface, basically Iodine sets up a virtual interface named dns0, through which the communication between the client and server will take place. Note that for testing inside a LAN, choose a different IP address if you are on the same subnet as 10.0.1.2\. For e.g if you are on the same subnet as 10.0.1.2 in the LAN, you could use the IP 192.168.1.2 .

c)Finally we need to enter the subdomain which will act as the other end of the tunnel. In this case it is tunnel.infosecinstitute.com.

d)Iodine will then ask for a password. This is the password that is used to authenticate the client. It therefore adds an extra layer of security.

Type "ifconfig dns0" to check that a new virtual interface has been created with the IP 10.0.1.2 .

![3]({{site.baseurl}}/images/posts/Maintaining-Access//3.png)

It is now time to set up the client. To do that go to your client and type the following command as shown in the figure below.

![4]({{site.baseurl}}/images/posts/Maintaining-Access//4.png)

a)We use ./iodine as we want to start the iodine client.

b) "-f" specifies Iodine to run in foreground, "-r 192.168.0.135" specifies the IP-address of the server running iodined, in a real world scenario, this should be a public IP address, which should be directly accessible from the internet. This is so that our client can communicate with it from anywhere.

c) It will then ask for the subdomain we want to query. In this case it is tunnel.infosecinstitute.com.

d)Finally we enter the same password we entered while setting up the server. Once the authentication is successful, Iodine client will communicate with the server and set up the tunnel.

All right, let's now do a quick "ifconfig dns0" on the client, we will see that a virtual interface named dns0 has been created on the client side as well and assigned the IP-Address 10.0.1.1 ,which is in the same subnet as the server IP( 10.0.1.2).

![6]({{site.baseurl}}/images/posts/Maintaining-Access//6.png)

To check if the tunnel is working, let's do a ping to the server (10.0.1.2) from our client. As we can see that we get a response, this means the tunnel works.

![7]({{site.baseurl}}/images/posts/Maintaining-Access//7.png)

If we see the packets in Wireshark, we will see that all the packets appear as DNS, even though right now ICMP ping requests and responses are being sent. This proves that all the traffic going through the tunnel appears as DNS traffic.

![8]({{site.baseurl}}/images/posts/Maintaining-Access//8.png)

## b)Ping Tunneling

Ping Tunneling is the process of encapsulating different protocols into ICMP protocol. Hence all the traffic appears to be in the form of ICMP Requests and ICMP Response. We will be using Ptunnel for creating a ping tunnel as it comes preinstalled with Backtrack. Ptunnel tunnels TCP packets through the ping tunnel, and hence they appear as ping requests and replies. This could be used in scenarios in which the firewall allows us to send ping requests and receive replies but do not allow TCP traffic for connections. Ptunnel also requires a client to be set up on the victim machine and a server on another machine which forms the other end of the tunnel.

To set up the ptunnel server, just go to the ptunnel directory in Backtrack and type in ./ptunnel .

![9]({{site.baseurl}}/images/posts/Maintaining-Access//9.png)

This sets up the server side of the ping Tunnel. To set up the client side type in the following command as shown in the figure below.

![10]({{site.baseurl}}/images/posts/Maintaining-Access//10.png)

a) -p 192.168.0.135 - This specifies the IP-address of the server running the ptunnel server.

b)-lp 8000-This opens up port 8000 on the client and starts listening for incoming connections.

c) -da and -dp specify the destination address and port we want to forward all the incoming requests to.

Once this is done, we need to connect to port 8000 via ssh. To do this we need to configure ssh to run on port 8000\. To do this edit the file "/etc/ssh/sshd_config" to run the ssh service on port 8000.

![11]({{site.baseurl}}/images/posts/Maintaining-Access//11.png)

Once this is done, restart the ssh service and check to see if ssh is listening for connections on port 8000.

![12]({{site.baseurl}}/images/posts/Maintaining-Access//12.png)

Finally connect to the ssh service from the localhost by typing "ssh -p 8000 root@localhost" .Type in the root password ("toor" by default) and we get the screen as shown below.

![13]({{site.baseurl}}/images/posts/Maintaining-Access//13.png)

SUCCESS! We just set up a ping tunnel ! Once the data flow has started, you can see the notification both on the server side and on the client side.

![14]({{site.baseurl}}/images/posts/Maintaining-Access//14.png)

**2)OS Backdoors**-The next category of tools are OS Backdoors .These backdoors when installed on the victim system can be used to create connections between the victim and the attacker machine. It can also be used to download files, execute commands and carry numerous other tasks.

1)Netcat-Netcat is one of the most popular tools available for creating backdoors. Netcat opens up a port on the victim machine, and the attacker can then connect to that port. Netcat can also be used to transfer files.It can also be configured for running a program on the victim machine on successful connection from the hacker machine. For e.g in this case we are going to open up port 5555 on the victim machine, and configure it to present a shell to the hacker once the hacker successfully connects.

Here are the commands on the victim machine

![15]({{site.baseurl}}/images/posts/Maintaining-Access//15.png)

a) -l tells it to listen for connections.

b)-p 5555 specifies the port that is open and allows the connection.

c)-e /bin/bash asks it to present a shell to the connecting machine.

d) -v asks netcat to be more verbose.

and here's the configuration on the attacker machine

![16]({{site.baseurl}}/images/posts/Maintaining-Access//16.png)

To connect, we just need to specify the IP-address and the port. As we can see once we are connected we can execute commands on the victim machine and also see the results. In this case we run the commands "ls" and "cat /etc/passwd" on the victim.

2)Sbd-Sbd is another program that can be used to create connections between the client and the sever. Some of the key features about sbd is that it can use encryption to encrypt the traffic.

In this case we will be using sbd to transfer files from the client to the server.Here are the commands on the victim side. I would recommend you to explore the different options available in sbd to create covert connections.

![17]({{site.baseurl}}/images/posts/Maintaining-Access//17.png)

Here are the results obtained when we connect to the victim machine. As we can see that we see a dump of the file.

![18]({{site.baseurl}}/images/posts/Maintaining-Access//18.png)

3) Msfpayload and Msfencode - Msfpayload is a part of the Metasploit framework. It allows us to create custom backdoors. These backdoors when executed on the victim can allow us to open up a connection between the victim and the attacker machine. However it is important for the backdoor to evade detection from the antivirus. For this we use MsfEncode which encodes the payload so that it is not detected by the antivirus. For Msfencode to encode the payload, we need to give it the file created by Msfpayload in Raw Format (by using the R option). In the diagram below we can see an example of creating an encoded executable file which can act as a backdoor.

![Msf]({{site.baseurl}}/images/posts/Maintaining-Access//msf.png)

a) windows/meterpreter/bind_tcp is the payload we want to use. A payload is the code which is executed on the victim machine upon successful completion of the exploit.

b)RHOST=192.168.0.134 is the IP-Address of the victim machine.

c)"R" specifies that we want the output in Raw format because Msfencode requires the file to be in Raw format for encoding it properly.

d)"|" specifies that we are piping the result to some other program (in this case msfencode).

e) -e x86/shikata_ga_nai specifies the encoder that we want to use.

f)-c 5 specifies the no of times we want to reencode the file, because the more no of times we encode it, the less probable it becomes for the antivirus to detect it.

g)-o specifies the name of the output file that we want.

h)-t exe specifies the format in which we want to output the file.

Once this file is run on the victim machine, all we need to do is run a handler on our system and we will get a session on the victim machine.

3) Web Backdoors - Web backdoors are another category of backdoors which could be installed on webservers. They can be webshells or can also be custom backdoors created using Msfpayload and Msfencode. Either way they allow us access either through an open port or through a web interface in the browser. For e.g. if the attacker gets access to the server, he can drop a malicious file say Backdoor.php onto the server. Later he can access the file through a browser and the web interface will allow the malicious hacker to perform various tasks, like traverse through directories, run system commands etc. Backtrack 5 ships in with some web backdoors, we will not be discussing creating backdoors using Msfencode and Msfpayload as we already discussed that in the previous section.

We will be using backtrack as the victim server in this case, for this we need to install a backdoor in it's "/var/www" folder and then we will start the apache server. Backtrack already comes with some backdoors that are located in "/pentest/backdoors/web/webshells" directory. To do these things, type in the commands as shown in the figure below.

![20]({{site.baseurl}}/images/posts/Maintaining-Access//20.png)

Once we have set up the backdoor, migrate to it from the browser

![21]({{site.baseurl}}/images/posts/Maintaining-Access//21.png)

As we can see it allows us to carry out remote commands on the system, let's try the command "cat /etc/passwd". We will see that we get a dump of the whole file.

![22]({{site.baseurl}}/images/posts/Maintaining-Access//22.png)

Let's try this out for another backdoor named "php-backdoor.php". First let's move it to the /var/www folder.

![23]({{site.baseurl}}/images/posts/Maintaining-Access//23.png)

And now let's migrate to the file from the browser. As we can see that it provides us with a beautiful interface, we can execute commands, browse to directories, upload files, and even execute mysql queries.

![24]({{site.baseurl}}/images/posts/Maintaining-Access//24.png)

## Conclusion

In this article we discussed different ways of maintaining access on a victim once it is compromised. There could be different scenarios, the victim may be behind a firewall, or could just be running a simple web server directly accessible from the internet. We learnt about 3 different tags of tools a) Tunneling tools, which can help us encapsulate one protocol onto another, thereby bypassing the firewall b)OS Backdoor- Netcat , sbd , creating custom backdoors using Msfpayload and then encode it using Msfencode c)Web backdoors- Could be some kind of custom backdoors, or could be webshells which could allow us to execute commmands, browse directories on the remote server etc.

This article was originally published on the [resources](http://resources.infosecinstitute.com/) page at [Infosec Institute](http://infosecinstitute.com/). For more information, please visit my author [page](http://resources.infosecinstitute.com/author/prateek/).