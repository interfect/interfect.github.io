# Meshnet Captive Portal

As a Meshnet Enthusiast, I've grabbed a couple of WiFi routers (including a nice Ubiquiti Nanostation that can happily live outside) and built myself a corner of the [fc00 cjdns meshnet](https://www.fc00.org/). The design goals were:

* Anyone should be able to come by and connect and get access to the meshnet, without installing anything.
* Nobody should be able to get access to the Internet in such a way that their actions might reflect poorly on me.
* People who connect should get an informative message about what they have connected to and why it isn't the Internet.
* If someone has [cjdns](https://github.com/cjdelisle/cjdns) software installed and is already a meshnet node, my node should peer with their node.

This document describes how I achieved these goals.

##Design Compromises

To make the whole thing easier to build and run, I made some compromises.

First, the WiFi setup consists of Access Points and Clients; I'm not trying to run it in the poorly-supported, poorly-documented Ad-Hoc mode which in theory would make things so much nicer but in practice makes them so much worse. The upshot is that Android clients can connect, but the downside is that you can't just stand up more clients and let them extend the network. If you want to make it bigger, you have to do manual configuration, and separate out the network that the APs use to talk to each other from the network that they use to talk to the clients.

Second, there's no WiFi- or IP-level mesh involved; everything is one big bridged network. In practice it's a bunch of Wi-Fi stuff, but in theory it all works like a bunch of daisy-chained Ethernet switches. Any "mesh" that may or may not be in this "meshnet" is provided by cjdns running over the top of everything. Nodes that can't run cjdns see a boring, centralized, single-router network with a few switches in it.

##Hardware Components

The system I have has three main pieces of hardware: a meshnet node on a server under my desk, a Ubiquiti Nanostation WiFi access point outside, and a WRT54G to attach them together.

###Meshnet Node

The most important physical piece of the system is the full meshnet node. It lives under my desk, runs the [meshnet software](#cjdns-mesh-metworking-software), and manages the network. It hands out IPv4 and private IPv6 addresses to clients when they connect, proxies their traffic through to the meshnet, *refuses* to proxy their traffic through to the Internet or the rest of my network, and does the web serving, DNS serving, and packet manipulation required to display the captive portal page that explains what the whole thing is about.

The meshnet node server has a wired connection to the rest of my network, and a USB WiFi dongle with big beefy antennas plugged in to connect it to the wireless part of the system. Although it hands out IP addresses and routes traffic, the meshnet node is *not* the access point; its WiFi interface is operating as a client.

###Access Point

The main access point in the system is a [Ubiquiti Nanostation locoM2](https://www.amazon.com/gp/product/B004EGI3CI/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&tag=interfect-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B004EGI3CI&linkId=23dade7a674144fe22776a7f5bc40fdf). (My blog is a participant in the Amazon Services LLC Associates Program, an affiliate advertising program designed to provide a means for sites to earn advertising fees by advertising and linking to amazon.com. Gimme your money.) The access point is running, obviously, in Access Point mode, and broadcasts the main SSID that clients are supposed to connect to.

The device itself is pretty cool; it sits outside and is attached to a wooden post with [a plastic mount](https://www.amazon.com/gp/product/B004EHUR8U/ref=as_li_tl?ie=UTF8&tag=interfect-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B004EHUR8U&linkId=b4930600028832c48681acc008c02f0a) that lets you aim it. The whole assemblage screws in with three tiny screws, so when you eventually take it down there won't be big holes in your wooden post.

One downside with the Ubiquiti setup was that it was damn near impossible to get the AP to slot into the plastic mount like it was supposed to. I had to pry it apart with a Japanese coin, which was the only one I could get to wedge in and hold the bracket open while I slid in the AP. I am not sure that their tolerances on the mounting bracket manufacturing are tight enough.

The other downside is that the Nanostation can only be powered by power-over-Ethernet, and although it comes with an injector there's no Ethernet cable in the box to connect the two together. Moreover, according to [the manual](https://dl.ubnt.com/guides/NanoStation_M/NanoStation_M_Loco_M_QSG.pdf), you're supposed to use shielded Ethernet cable, with shielded end connectors to ground it to the ground in the injector. I ended up having to buy [a bunch of shielded cable](https://www.amazon.com/gp/product/B00P5DGEJ8/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&tag=interfect-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B00P5DGEJ8&linkId=1a77367ee4ec1fb08c9c16114e8d6e23) and [a bunch of shiny metal-encased end connectors that look really cool](https://www.amazon.com/gp/product/B008XP94BA/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&tag=interfect-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B008XP94BA&linkId=a186a45efabe64df5f21aecb67cce063) to make shielded cable to run out to the AP from inside where the power outlet lives. In the course of that process, I learned that Ethernet sockets are designed to make electrical contact with the grounded outer shielding of these fancy connectors, and that my crimper can only sort of push in the pins on these fancy connectors. I did manage to make a cable that satisfied my tester, after about 3 attempts on each end, but I won't vouch for it's reliability. It may be that I have a bad crimper, or some kind of unshielded-only crimper, or it may just be that I'm not a real network engineer.

In the end, I finally got the thing mounted and hooked up, broadcasting its SSID and bridging it with its Ethernet interface.

###Extra Router

After putting the main AP outside, facing away from my house, and then connecting to it from the other side of the house, the setup worked, but I got really low througput. I attributed this to being exactly where the directional antenna in the AP wasn't pointing, and so I added an extra WRT54G router I had lying around to cover the inside of my house. The router has one of its LAN ports hooked up to the data port on the PoE injector, and bridges from Ethernet to another AP-mode WiFi SSID that I connect to from inside the house. Once I set that up, throughput improved dramatically.

##Software Components

The Nanostation AP runs the stock Ubiquiti firmware, and the WRT54G runs a horribly old build of Tomato. All the software that really makes the system work runs on the meshnet node server itself.

###cjdns Mesh Networking Software

###radvd IPv6 Router Advertisment Daemon

###DHCP Server

###DNS Server

###iptables Forwarding, Blocking, and Captive Portal-ing
