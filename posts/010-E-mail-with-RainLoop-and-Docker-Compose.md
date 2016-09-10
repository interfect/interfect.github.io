# E-mail with RainLoop and Docker Compose

Recently Google decided to update Gmail's interface to [Material Design 2](https://9to5google.com/2018/04/26/what-is-material-design-2-examples-launch-io/), with cool rounded buttons and a new logo.

![Gmail's new UI features more rounded things](/images/010/newui.png)

Unfortunately, there are two problems with this redesign that make it not work for me. One of them is that the new interface is much more **animation-heavy**; it significantly raises the minimum system requirements for a quick and responsive e-mail experience, and one of the PCs I use Gmail on can no longer keep up. Even on a good computer, it now takes two or three seconds more on the critical path from navigating to https://mail.google.com to getting a new message window in response to your "Compose" button click

The other major problem with the new Gmail is the inclusion of **"Confidential Mode"**.

## Confidential Mode

![Confidential mode tries to make e-mail do things that e-mail cannot do](/images/010/confidentialmode.png)

Confidential Mode replaces e-mails that you send from Gmail with links back to Google, so that Google can choose to dispense or not dispense the content of your messages to the recipients. After a certain date, or when you ask Google to rescind the message, Google will stop handing over the message contents and the message will be "expired".

### Not a Mode

That's all well and good, but that's not what this feature is being sold as. It's being marketed by Google as a "mode" that ordinary e-mail can be in, but it's really more akin to an e-mail substitute. Gmail users get a message rendered in-line, but the thing actually sent by e-mail is a *link* to the message, and that isn't made clear. They don't advertise that recipients are going to have to agree to Google's terms of service and privacy policy, for example, before being allowed to read your message. The way it is treated makes me think that **Google is quite happy to have people conflate Gmail and e-mail**, and I don't like that at all.

### Not Confidential

What Google does talk about is the advanced security features that Confidential Mode messages come with, which prevent the recipient from saving or printing the messages they receive. These security measures are, naturally, implemented client-side in the untrusted recipient's web browser. Less naturally, they are implemented in CSS, and are [trivially defeatable with the Firefox Style Editor](https://grokprivacy.org/2018/06/24/archiving-self-destructing-gmail-with-firefox/). Although I haven't found one yet, a browser extension could do it even more easily. How does Google disclose this shortcoming to Confidential Mode users?

> Recipients who have malicious programs on their computer may still be able to copy or download your messages or attachments.

You heard it here first, folks! Firefox, along with every other piece of software that is willing to put the interests of its immediate user first, is a **malicious program**! If you don't view the content we sent you in the way we told you to view it, you're doing Bad Hacker Crimes and should be sent to jail! This is because we are Google, the makers of e-mail.

## Alternatives

Between the poor performance of the new web interface, and the apparent preparation for a hostile takeover of the federated e-mail network, I decided that I would rather not use the new Gmail. So I went and fired up Ubuntu's default e-mail client, Thunderbird, and logged it into my Gmail account. Presto! A new, fully-spec-compliant e-mail client for Gmail, right?

Well, not so much.

### Thunderbird

![Thunderbird deals poorly with this much mail](/images/010/thunderbird.png)

Thunderbird was my e-mail client of choice back in the pre-Gmail days of the early-to-mid 2000s. It's the same basic program today, and it isn't designed for the **210,487 e-mails** that 11 years of archiving everything have left me with. It wants to download headers for all of them and display them all in a big list with a tiny scrollbar, and has no concept of pagination. Note how, in the screenshot, it seems to have gotten stuck on message 9 of the 182,302 messages that it still hasn't managed to get headers for.

Also, in the last decade I've worked out that [it's not really OK to murder a bunch of people, steal their continent, and then borrow cool stuff from their religion to name your software](http://ohwitchplease.ca/2016/12/episode-18-roaring-anachronisms-and-where-to-find-them/), so it bothers me every time I look at the title bar now.

### RainLoop

[RainLoop](https://www.rainloop.net/) is an AGPLv3-licensed web-based e-mail client. You run the client on your web server, and you interact with it through your browser. You "log in" to RainLoop with your e-mail account credentials, and it logs in to the actual IMAP server behind the scenes and presents you with a view of your e-mail in your browser.

Because it's a web-based client, it doesn't suffer from the same shortcomings that Thunderbird has with regard to immediately trying to download and display massive amounts of mail. And because the interface is in the browser, while the backend is running as its own program, if it does have to sit around thuinking about hundreds of thousands of messages, it has a limited impact on UI responsiveness.

Here's a screenshot from RainLoop's official online demo, to which people are happily uploading potentially malicious .exe files.

![RainLoop has a sidebar, a message list, and a message view](/images/010/rainloop.png)

Note the support for pagination of the message list, and for large numbers of pages. There's also a handy "Mark all as read" feature for dealing with unread messages in huge folders.

I decided that RainLoop looked interesting, but it's not as simple to install as Thunderbird, because it needs a web server. Moreover, if I'm going to go through all the trouble of setting up a web server, I want to be able to get to my e-mail from the Internet.

## Installing RainLoop

Luckily, there's a nice Docker image of RainLoop available: [hardware/rainloop](https://hub.docker.com/r/hardware/rainloop/).

To use it, you will need Docker installed. For Ubuntu, that looks like:

```
sudo apt install docker
```

You can then get RainLoop going right away with:

```
mkdir webmail-host
cd webmail-host
mkdir rainloop
docker run --name rainloop -e LOG_TO_STDOUT='true' -v ./rainloop:/rainloop/data hardware/rainloop
```

This sets up a container running an Nginx web server with RainLoop installed on it, listening on the container's port 8888. You can stop it with `Ctrl+C`.

### Docker and VPNs

Note that, if you are using a VPN or otherwise have routes covering a lot of the IP space, Docker will be upset that it can't find a block of un-routeable IP addresses to use for its `docker0` virtual bridge network, and the `docker` daemon will refuse to start up. In that case, you will have to make the bridge manually, with something like [this](https://github.com/moby/moby/issues/31546#issuecomment-284263284):

```
sudo brctl addbr docker0
sudo ip addr add 192.168.42.1/24 dev docker0
sudo ip link set dev docker0 up
ip addr show docker0
sudo systemctl restart docker
```

### Configuring RainLoop

When we started RainLoop, we didn't expose the container's port 8888 to the world; you can only access it from the computer it is running on. To do that, get ahold of the container's IP address:

```
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' rainloop
```

I got `172.17.0.3`, but you may get something different. You can then go to http://172.17.0.3:8888/?admin (substituting in the address you got), log in with the default credentials of `admin` and `12345`, and configure your new e-mail client:

1. Go to "Security" on the left and **set a new admin password**. Make sure to remember it.
2. Also under "Security", make sure to enable "Require verification of SSL certificate used (IMAP/SMTP)". I have no idea why this is off by default; it is essential that you not just hand your e-mail password over to people with self-signed certificates.
3. Under "Domains", make sure that your e-mail domain appears in the list of domains that can use RainLoop. By default, `gmail.com` is enabled already. Also, go into "Whitelist" and put your username (without `@gmail.com`) on the whitelist. This will mean that only you (and not every Gmail user) can use your RainLoop instance.
4. Under "Contacts", you should check "Enable contacts" and also "Allow contacts sync (with external CardDAV server)". This will allow you to have your Google contacts pre-populate when composing e-mail.

Once you've done the initial configuration, you can sign in on the main login page, http://172.17.0.3:8888/ (again substituting your container's IP if it is different), with your Gmail e-mail address and password. (If you are using 2-factor authentication, you will need an [app password](https://support.google.com/accounts/answer/185833?hl=en).) Then, once logged in, click on the person icon in the upper right, and select "Settings", so you can configure your user settings:

1. Under "Folders" on the left, hit "System Folders", and set "Spam" to "Do not use". This will prevent RainLoop nagging you about how many unread spam messages you have with a little badge.
2. Under "contacts" on the left, check "Enable remote synchronization". Enter `https://google.com` as the "Addressbook URL", and use your Gmail e-mail address and password as the "User" and "Password". This will make RainLoop periodically download and update your Google address book.

**Congratulations!** You now have a working e-mail client. If that is all you want, you can stop now. But if you want your client to start on boot and be accessible from anywhere, you can move on to...

## Using Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) is a feature of Docker that allows multiple containers to be run together as an application, coordinated by a `docker-compose.yml` file. That file lists what container images to run as part of the application, what volume paths they should use, what ports each should expose, and how they should interact with each other. It also makes setting up Docker virtual networks pretty easy. Perhaps most importantly, **Docker Compose takes care of starting up all the containers on boot**; anything you set up will persist across reboots automatically, with no need for a systemd service or anything in `/etc/rc.local`.

If you don't have it already, you can install it on Ubuntu with:
```
sudo apt install docker-compose
```
The command will then be accessible as `docker-compose`.

If we want RainLoop to be accessible from the Internet, we're going to have to secure it with an SSL certificate. So in addition to the RainLoop container, we're going to add [Traefik](https://traefik.io/), a proxy which can sit in front of RainLoop and receive SSL connections. Traefik also has built-in support for automatically obtaining SSL certificates from [Let's Encrypt](https://letsencrypt.org/), without you having to do anything. The Docker image name is just `traefik`.

To get the SSL certificate and to make access from the Internet practical, we need a DNS name. You could go out and register a whole domain name, or you could use  For this I recommend [Affraid.org's FreeDNS service](https://freedns.afraid.org/), an alternative to commercial services like DynDNS, which has a wide selection of domains whose owners are willing to hand out free subdomains. To interface with it, we can use [ddclient](https://sourceforge.net/p/ddclient/wiki/Home/), a dynamic DNS client that supports several services. There are a few different Docker images for this tool, but I have selected `rycus86/ddclient`.

### Setting Up the Application

1. If you have been following along with the tutorial, Ctrl+C out of the running RainLoop that we started earlier.
2. Get your hostname set up at https://freedns.afraid.org/, or from wherever else you plan to obtain it.
3. If you are behind a firewall router, set up [port forwarding](https://superuser.com/questions/284051/what-is-port-forwarding-and-what-is-it-used-for) for ports 80 and 443 to the machine that will be hosting your application. Also make sure to open the ports in your system's firewall.
4. In the `webmail-host` directory, create two more folders to hold configuration and runtime data for the Traefik and ddclient containers:
    ```
    mkdir traefik
    mkdir ddclient
    ```
5. Make a file `ddclient/ddclient.conf` based on the following template:
    ```
    daemon=5m
    use=web, web=dyndns
    timeout=10
    syslog=no
    pid=/var/run/ddclient.pid
    ssl=yes

    server=freedns.afraid.org
    protocol=freedns
    login=YOUR_USER_HERE
    password=YOUR_PASSWORD_HERE
    YOUR_DOMAIN_HERE
    ```
Make sure to fill in `YOUR_USER_HERE`, `YOUR_PASSWORD_HERE`, and `YOUR_DOMAIN_HERE` with the information for the domain you are using, and change the `server` and `protocol` if you are using another dynamic DNS provider.
6. Mostly, Traefik is intended to Just Work without any configuration, and a lot of the things you can do with the config file can also be done with labels on Docker containers, if you start it with `--docker`. But we're going to make a config file for it anyway, in `traefik/traefik.toml`, using this template:
    ```
    debug = false

    logLevel = "ERROR"
    defaultEntryPoints = ["https","http"]

    [entryPoints]
      [entryPoints.http]
      address = ":80"
        [entryPoints.http.redirect]
        entryPoint = "https" # Redirect HTTP traffic to HTTPS
      [entryPoints.https]
      address = ":443"
      [entryPoints.https.tls]

    [retry]

    [docker]
    # Talk to Docker and set up proxying for containers we find
    endpoint = "unix:///var/run/docker.sock"
    domain = "YOUR_DOMAIN_HERE"
    watch = true
    exposedByDefault = false # But only containers that ask for it

    [acme]
    email = "YOUR_EMAIL_HERE"
    storage = "/etc/traefik/acme.json"
    entryPoint = "https"
    onHostRule = true
    [acme.httpChallenge]
      entryPoint = "http"
    ```
Make sure to replace `YOUR_DOMAIN_HERE` with the domain name that you are using, and `YOUR_EMAIL_HERE` with the e-mail you want Let's Encrypt to associate with your certificate.
7. Finally, make the `docker-compose.yml` file, using this template:
    ```
    version: '2'

    services:
      reverse-proxy:
        image: traefik
        ports:
          - "80:80"
          - "443:443"
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock:ro
          - ./traefik:/etc/traefik
        networks:
          - webmail-net
        depends_on:
          - ddclient
        restart: always # Restart even after underlying Docker host reboots
        
      rainloop:
        image: hardware/rainloop
        labels:
          - "traefik.enable=true"
          - "traefik.port=8888"
          - "traefik.frontend.rule=Host:YOUR_DOMAIN_HERE"
        volumes:
          - ./rainloop:/rainloop/data
        environment:
          - UID=1000
          - GID=1000
          - LOG_TO_STDOUT=true
        networks:
          - webmail-net
        restart: always
      
      ddclient:
        image: rycus86/ddclient
        command: --daemon=300 --ssl --debug
        volumes:
          - ./ddclient/ddclient.conf:/etc/ddclient.conf
        networks:
          - webmail-net
        restart: always
        
    networks:
      webmail-net:
        ipam:
          config:
            - subnet: 172.16.238.0/24
              gateway: 172.16.238.1
    ```
Make sure to replace `YOUR_DOMAIN_HERE` in the `traefik.frontend.rule` label on the `rainloop` container with the domain that you are using.

The `docker-compose.yml` file defines how the application fits together. Each "service" describes one of the containers we want, including the image it should be started from, the "volumes" (which expose host files to the container) that it should have access to, and the virtual network that it should be attached to. There are also "labels", which is how Traefik does most of its configuration, and which it reads via its access to `/var/run/docker.sock`. (Note that this access also means that the Traefik container isn't *really* contained; it can [escape its container](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/) even with the read-only access we give it.)

#### Adventures in Networking

We use compose file format version 2, and define explicitly the subnet that we want our virtual network to use (as only compose file version 2 allows), because if you are on a VPN or otherwise have routes covering private IP space, Docker can't come up with a free block of private IP addresses to use on its own for the virtual network. So we have to tell Docker explicitly that we want it to use this subnet, even if the system thinks it has a route to it elsewhere.

Because I want my Docker containers to send their outbound traffic over my eth0 interface, instead of through any tunnels, I had to create an alternative routing table and direct traffic from the virtual network to use it. Also, because Linux blocks any traffic that isn't coming *from* where it would route traffic *to* for that traffic's source address, I had to turn down the [reverse path filter](https://www.slashroot.in/linux-kernel-rpfilter-settings-reverse-path-filtering). I added the following to /etc/rc.local:

```
# Set up this one Docker subnet as exempt from VPN tunneling, and use the router at 10.1.0.1
ip route add default dev eth0 via 10.1.0.1 table 1001
ip rule add fwmark 1001 lookup 1001
iptables --wait -t mangle -A PREROUTING -s 172.16.238.0/24 -j MARK --set-mark 1001
# And don't require packets to come back from the best route for them, to make that work both ways
echo 2 >/proc/sys/net/ipv4/conf/eth0/rp_filter
```

This works by using the `iptables` firewall to "mark" traffic from the Docker subnet with a number (1001), setting up an additional routing table (table number 1001) with just a default route in it, and saying that we want to use table 1001 to route traffic with mark 1001. Then we turn down the reverse path filter, so that traffic arriving at eth0 from the Internet will be accepted, even if we have a better route to the Internet over another interface.

If you aren't using a VPN or tunnel, or your incoming traffic comes over that tunnel, you won't have to mess around with any of this.

### Running the Application

Once you have your `webmail-host/docker-compose.yml` written, and the new `webmail-host/ddclient/` and  `webmail-host/traefik/` directories (along with a `webmail-host/rainloop/` directory if you don't have one from the first part of the tutorial), you can start up your application from the `webmail-host/` directory with:

```
docker-compose up
```

This will bring up all the containers in the application, in dependency order, and keep you attached to their output, so you can see if any of them complain. If everything goes right, you should see something like:

```
Creating network "webmailhost_webmail-net" with the default driver
Creating webmailhost_rainloop_1 ... 
Creating webmailhost_ddclient_1 ... 
Creating webmailhost_rainloop_1
Creating webmailhost_ddclient_1 ... done
Creating webmailhost_reverse-proxy_1 ... 
Creating webmailhost_reverse-proxy_1 ... done
Attaching to webmailhost_rainloop_1, webmailhost_ddclient_1, webmailhost_reverse-proxy_1
rainloop_1       | [INFO] Logging to stdout activated
```

If things go wrong (for example, because an SSL certificate can't be obtained for your domain), you will see other messages in the console.

Check to make sure that your service is available at the domain you are using, and that it is using a valid SSL certificate. Note that it can take a couple minutes for the certificate to be issued.

Whether you are happy or unhappy with the results, the next step is to stop the application with `Ctrl+C`, either to tinker with the configuration or to set it up to run on its own.

#### Set and Forget

Once you are happy with your configuration, you will want to start up your application in "detached" mode, so you can have it run without your terminal window. To do this, just run (from the `webmail-host/` directory):

```
docker-compose up -d
```

After the application starts up, it will run in the background forever, restarting itself after reboots. To stop it again, in order to change the configuration or because you no longer want to have it, you have to run (again from the `webmail-host/` directory):

```
docker-compose down
```

## Conclusion

By assembling RainLoop, Traefik, and ddclient using Docker Compose, you can create a modern webmail solution that can replace both the Gmail web interface and Mozilla Thunderbird while combining the best aspects of both. Moreover, since Traefik supports [routing requests for many services presented at different hostnames and paths](https://docs.traefik.io/basics/), and Docker is the current standard for server software distribution, it would be easy to extend this basic structure with additional self-hosted services.

![A diagram showing the three containers, their presence in the virtual network, and how traffic goes from the Internet through Traefik to Rainloop, and then back the way it came. Also shown is ddclient making its DNS updates, and the fact that traffic between the Internet and Traefik is secure.](/images/010/diagram.png)









