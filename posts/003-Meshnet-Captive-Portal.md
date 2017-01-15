# Meshnet Captive Portal

As a Meshnet Enthusiast, I've grabbed a couple of WiFi routers (including a nice Ubiquiti Nanostation that can happily live outside) and built myself a corner of the [fc00 cjdns meshnet](https://www.fc00.org/), also known as Hyperboria. The design goals were:

* Anyone should be able to come by and connect and get access to the meshnet, without installing anything.
* Nobody should be able to get access to the Internet in such a way that their actions might reflect poorly on me.
* People who connect should get an informative message about what they have connected to and why it isn't the Internet.
* If someone has [cjdns](https://github.com/cjdelisle/cjdns) software installed and is already a meshnet node, my node should peer with their node.

This document describes how I achieved these goals.

## Design Compromises

To make the whole thing easier to build and run, I made some compromises.

First, the WiFi setup consists of Access Points and Clients; I'm not trying to run it in the poorly-supported, poorly-documented Ad-Hoc mode which in theory would make things so much nicer but in practice makes them so much worse. The upshot is that Android clients can connect, but the downside is that you can't just stand up more clients and let them extend the network. If you want to make it bigger, you have to do manual configuration, and separate out the network that the APs use to talk to each other from the network that they use to talk to the clients.

Second, there's no WiFi- or IP-level mesh involved; everything is one big bridged network. In practice it's a bunch of WiFi stuff, but in theory it all works like a bunch of daisy-chained Ethernet switches. Any "mesh" that may or may not be in this "meshnet" is provided by cjdns running over the top of everything. Nodes that can't run cjdns see a boring, centralized, single-router network with a few switches in it.

## Hardware Components

The system I have has three main pieces of hardware: a meshnet node on a server under my desk, a Ubiquiti Nanostation WiFi access point outside, and a WRT54G to attach them together. Here is an aggressively terrible diagram of my network setup, made using MS Paint.

![A diagram showing the meshnet, my meshnet server node, a WRT54G, a PoE injector, and a Ubiquiti Nanostation connected together in that order.](/images/003/aggressively-terrible-diagram.png)

### Meshnet Node

The most important physical piece of the system is the full meshnet node. It lives under my desk, runs the [meshnet software](#cjdns-mesh-networking-software), and manages the network. It hands out IPv4 and private IPv6 addresses to clients when they connect, proxies their traffic through to the meshnet, *refuses* to proxy their traffic through to the Internet or the rest of my network, and does the web serving, DNS serving, and packet manipulation required to display the captive portal page that explains what the whole thing is about.

The meshnet node server has a wired connection to the rest of my network, and a USB WiFi dongle with big beefy antennas plugged in to connect it to the wireless part of the system. Although it hands out IP addresses and routes traffic, the meshnet node is *not* the access point; its WiFi interface is operating as a client.

### Access Point

The main access point in the system is a [Ubiquiti Nanostation locoM2](http://www.newegg.com/Product/Product.aspx?Item=9SIA1N84EE5480). The access point is running, obviously, in Access Point mode, and broadcasts the main SSID that clients are supposed to connect to.

The device itself is pretty cool; it sits outside and is attached to a wooden post with [a plastic mount](http://www.newegg.com/Product/Product.aspx?Item=0ED-0005-00030) that lets you aim it. The whole assemblage screws in with three tiny screws, so when you eventually take it down there won't be big holes in your wooden post.

One downside with the Ubiquiti setup was that it was damn near impossible to get the AP to slot into the plastic mount like it was supposed to. I had to pry it apart with a Japanese coin, which was the only one I could get to wedge in and hold the bracket open while I slid in the AP. I am not sure that their tolerances on the mounting bracket manufacturing are tight enough.

The other downside is that the Nanostation can only be powered by power-over-Ethernet, and although it comes with an injector there's no Ethernet cable in the box to connect the two together. Moreover, according to [the manual](https://dl.ubnt.com/guides/NanoStation_M/NanoStation_M_Loco_M_QSG.pdf), you're supposed to use shielded Ethernet cable, with shielded end connectors to ground it to the ground in the injector. I ended up having to buy [a bunch of shielded cable](https://www.amazon.com/gp/product/B00P5DGEJ8) and [a bunch of shiny metal-encased end connectors that look really cool](https://www.amazon.com/gp/product/B008XP94BA) to make shielded cable to run out to the AP from inside where the power outlet lives. In the course of that process, I learned that Ethernet sockets are designed to make electrical contact with the grounded outer shielding of these fancy connectors, and that my crimper can only sort of push in the pins on these fancy connectors. I did manage to make a cable that satisfied my tester, after about 3 attempts on each end, but I won't vouch for it's reliability. It may be that I have a bad crimper, or some kind of unshielded-only crimper, or it may just be that I'm not a real network engineer.

In the end, I finally got the thing mounted and hooked up, broadcasting its SSID and bridging it with its Ethernet interface.

### Extra Router

After putting the main AP outside, facing away from my house, and then connecting to it from the other side of the house, the setup worked, but I got really low throughput. I attributed this to being exactly where the directional antenna in the AP wasn't pointing, and so I added an extra WRT54G router I had lying around to cover the inside of my house. The router has one of its LAN ports hooked up to the data port on the PoE injector, and bridges from Ethernet to another AP-mode WiFi SSID that I connect to from inside the house. Once I set that up, throughput improved dramatically.

## Software Components

The Nanostation AP runs the stock Ubiquiti firmware, and the WRT54G runs a horribly old build of Tomato. All the software that really makes the system work runs on the meshnet node server itself.

### cjdns Mesh Networking Software

The most important sofware component is [cjdns](https://github.com/cjdelisle/cjdns#cjdns), which provides the mesh networking capability. Cjdns is used to connect the main meshnet node server to other systems on the Internet, as part of a mesh network. There are several useful services on this meshnet, including an IRC server, a free e-mail service, and [IPFS](https://ipfs.io). All addressing is IPv6, in the `fc00::/8` block reserved for [unique local addresses without a specified allocation method](https://en.wikipedia.org/wiki/IPv6_address#Unique_local_addresses). Cjdns allocates addresses by public key hash; all communication is end-to-end encrypted.

Cjdns is configured for Ethernet auto-peering on all interfaces, with this `ETHInterface` stanza in the `interfaces` section of `/etc/cjdroute.conf`:

```
"ETHInterface":
[
    {
        // Bind to this device.
        "bind": "all", 
        "beacon": 2
     }
]
```

This configures the cjdns daemon to send out raw broadcast Ethernet frames attempting to peer with other nodes on the local network, and accept such peering requests itself. This means that if any other similarly-configured cjdns nodes (like my laptop) connect to the meshnet WiFi network, they will automatically peer with the meshnet server node and have meshnet connectivity.

### radvd IPv6 Router Advertisement Daemon

Clients that don't run cjdns can still connect to meshnet hosts, by using the meshnet server as a NAT-ing IPv6 router. To this end, the system advertises a private IPv6 subnet of `fdfc::/64` to everyone who connects to the WiFi network.

This is accomplished with the following stanza in `/etc/radvd.conf`:

```
interface wlan0 {
	AdvSendAdvert on;
	MaxRtrAdvInterval 30;
	prefix fdfc::/64
	{
		AdvOnLink on;
		AdvAutonomous on;
	};
};
```

### WiFi Connectivity

The meshnet server needs to actually be on the WiFi, with an address on the `fdfc::/64` subnet, in order to do anything. So here's `/etc/network/interfaces`:

```
auto wlan0
iface wlan0 inet static
	wpa-driver wext
	wpa-conf /etc/wpa_supplicant.conf
	address 10.2.0.2
	netmask 255.255.255.0
iface wlan0 inet6 static
	address fdfc::2
	netmask 64
```

I also gave it an IPv4 address, to run the captive portal.

Here's the `/etc/wpa_supplicant.conf` that tells it to actually connect to the correct network (in this case, `SantaCruzMeshnet2`, which is broadcast by the WRT54G). It's a bit odd to use wpa_supplicant to connect to an open network, but that's how I got it working, and once I got it working I didn't want to poke it.

```
network={
	ssid="SantaCruzMeshnet2"
	scan_ssid=1
	key_mgmt=NONE
}
```

### DHCP Server

In order to explain to people what sort of network they're connected to, I need some kind of captive portal, which will redirect web pages (over IPv4) to a welcome page. Unlike real captive portals, I don't eventually end up granting Internet access, so there's no need to make the captivating functionality turn on and off.

I also need to hand out IPv4 addresses to weird clients that will be upset if they can't get an IPv4 address and only have IPv6.

For this, I am using `isc-dhcp-server`, which is configured with this in `/etc/dhcp/dhcpd.conf`:

```
subnet 10.2.0.0 netmask 255.255.255.0 {
	range 10.2.0.10 10.2.0.254;
	option routers 10.2.0.2;
	option domain-name-servers 10.2.0.2;
}
```

There's also this in `/etc/default/isc-dhcp-server` to actually turn it on:

```
INTERFACES="wlan0"
```

### DNS Server

The DHCP server refers clients back to the same host as a DNS server, and I need a DNS server that can answer requests if I want the captive portal to work. Additionally, a lot of meshnet services keep their hostnames in normal DNS, and don't necessarily work if accessed just by IPv6 in the URL bar, so meshnet clients are going to want access to Internet DNS too.

I just installed `bind`, in its default configuration, to act as a DNS server and proxy requests through to the system's configured DNS servers.

### Web server

The captive portal needs to involve a web page that people see when they connect. I installed `apache2` and set up a simple `index.html` explaining what the meshnet is and why people shouldn't just immediately disconnect and look for a real Internet network.

### iptables Forwarding, Blocking, and Captive Portal-ing

The meshnet server is set up to route traffic between a local IPv6 subnet and the cjdns meshnet, using IPv6 NAT. This is accomplished by turning on IPv6 forwarding (by setting `net.ipv6.conf.all.forwarding=1` in `/etc/sysctl.conf`) and then placing the following in `/etc/rc.local`:

```
# Start IPv6 NAT for Hyperboria
ip6tables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
```

Here `tun0` is the virtual interface that cjdns creates.

I also have `/etc/rc.local` commands to set up the captive portal. They look like this:

```
# Mark all packets coming in for routing from 10.2.0.*
iptables -t mangle -A PREROUTING -s 10.2.0.0/24 -j MARK --set-mark 99

# Steal all web traffic being routed from the 10.2.0.* subnet
iptables -t nat -A PREROUTING -m mark --mark 99 -p tcp --dport 80 -j DNAT --to-destination 10.2.0.2

# Drop anything else getting forwarded
iptables -t filter -A FORWARD -m mark --mark 99 -j DROP
```

Basically, I'm taking all packets that come in over IPv4 on the WiFi subnet and mark them with mark number 99. Then, any marked packets to port 80 get re-written to go to the meshnet server, while any other marked packets get dropped. This effectively man-in-the-middles my server between clients and whatever web server they were trying to access, and so they get my page instead of the page they wanted.

This only works if `net.ipv4.ip_forward=1` is set in `/etc/sysctl.conf`, to enable IPv4 routing.

It's worth noting, also, that the meshnet server does not have Internet IPv6 connectivity. If it did, I would want to drop IPv6 packets between the WiFi network and the Internet as well.

## Shortcomings and Improvements

Right now, the system mostly works: people will get IPv4 and IPv6 addresses when they connect, they can access meshnet sites through the IPv6 NATing proxy if they don't have cjdns, they get cjdns auto-peering if they do have cjdns, and if they get confused there's an obvious web page explaining what's going on. However, there are still a few places where the system could be improved:

* **Firewall Security**: I'm not confident that there's no clever way around my iptables rules.
* **Captive Portal Redirect**: Right now the captive portal doesn't do any HTTP redirects, so requests for `http://example.com` will wind up with the captive portal page displayed, but appearing to come from the `example.com` domain. Also, requests for things other than the root of the domain, like `http://example.com/thingy.html` will 404, because there's no `thingy.html` on my server. Both of these ought to redirect to `http://10.2.0.2/index.html`.
* **Android Experience Smoothing**: Potentially relatedly, Android doesn't display the little "log into WiFi network" prompt it displays for normal captive-portal-protected sites. This may be due to the lack of redirect, or it may be because it detects there's no log in form somehow.
* **DNS Tunneling Prevention**: Because I expose a normal DNS server, it's possible to tunnel out to the Internet with something like [iodine](http://code.kryo.se/iodine/). However, since anyone tunneling over iodine is going to have to go through a proxy they control at the other end, I don't find it likely that I will be blamed for any nefarious crimes being tunneled over the DNS system, and so finding a way to prevent it isn't a priority for me.
* **Missing `iptables` Rules**: I have had my `iptables` rules added by `/etc/rc.local` go missing on me during the time the system is up. I've unistalled the `ufw` package (`sudo apt-get remove ufw`), which is installed by default on Ubuntu and wants to manage your iptables rules for you. Perhaps that will solve the problem.
