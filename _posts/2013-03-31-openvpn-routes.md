---
layout:     post
title:      "OpenVPN to route all / selective traffic to a client"
date:       2013-03-31 10:00:00 +0200
categories: blog
---

This post is inspired from my urge to watch Macedonian TV (for free). Broadcast of Macedonian television is actually available on internet via Web MaxTV.mk, however it is limited to Macedonian IP addresses only. Since this scenario for tv broadcast or any kind of services, which are limited to a particular IP country block, applies to many other people living abroad, I decided to make my nightly research/experiment public.

The standard way to use services that operates in one or several countries only is to use proxy servers. However, in some cases there are either no proxies available in the home country, the services that one is planing to use can not be bypassed with a simple proxy server, the proxy services are slow to transfer broadcasts, or one just does not want to pay for a proxy service.

![MaxTV](/img/maxtv.png)

Obviously from the picture above, I have already resolved the problem, by making my home computer (the one in Macedonia) an internet gateway. This is almost a trivial task when the home computer has a public IP address, but in most cases, this simply does not apply (such as my Macedonian IP). Assuming that one can get his hands on a computer with a public IP address, a cheap (almost free) solution is routing via OpenVPN. What you need is the following:

A computer with a public IP address (the server), located anywhere in the world. 
A computer in the home country (the gateway), with a decent internet connection.
One or several clients (the client), with an Internet connection
As I currently live in Switzerland, I am more than happy to use the services of upc cablecom ISP, especially since they are so nice to provide me a public IP address. Therefore, I can simply setup an OpenVPN server in Switzerland, and use it as a tunnel to redirect traffic to the home computer. My current setup looks like this:

![MaxTV](/img/openvpn.png)

From the drawing above you can infer that the server is ch-server, the gateway is mk-gateway and finally the client is astojanov-mac. The gray lines and black labels represent the physical connections, and the red lines and red labels specify the OpenVPN network. 

72.xxx.xxx.xxx is the public address in Switzerland, and the router behind it has port-forwarding to ch-server, port 1194 to the ch-server having local IP address 192.168.1.50
ch-server and astojanov-mac are behind the same router and are part of the 192.168.1.0/24 network. Note that the client astojanov-mac can access the OpenVPN server from any network node on the Internet. Thus the route to access the ch-server goes through the Internet cloud.
mk-gateway is part of the 192.168.0.0/24 local network in Macedonia and has no public IP address attached on the router.
The OpenVPN overlaid network is represented with 192.168.2.0/24. The server has a static ip address: 192.168.2.1, as well as the gateway 192.168.2.250. The client astojanov-mac as every other OpenVPN client are assigned dynamic ip address.
The first step is installing and setting up OpenVPN. In order to run the server, a very modest computer can fulfill the needs of a VPN server if less than 10 VPN connections are anticipated. Personally I am using Intel(R) Pentium(R) M processor 1200MHz, with 1.6 GB of RAM. OpenVPN is cross platform and has no OS requirements. ch-server runs Debian GNU/Linux 6.0.6 (squeeze) and I base my instruction on this distribution. Note that to run OpenVPN server / client there are many alternatives with less power consumption requirements. For example the Raspberry Pi, which has 700 Mhz ARM CPU and costs about $25, or your router if you can set up DD-WRT on it.

Installing OpenVPN sever and setting up server / client keys & certificates
---------------------------------------------------------------------------

Installing and setting up OpenVPN has almost cross-platform steps. Also there are many tutorial available out there. My setup is based on the following tutorial, but you can also find additional tutorial on Linux, Windows and Mac OS X. First install OpenVPN and OpenSSL:

apt-get install openvpn openssl
Now we need to create server certificates. Before that we edit the variables of the certificates we are about to create.

{% highlight powershell %}
cd /etc/openvpn
cp -r /usr/share/doc/openvpn/examples/easy-rsa/2.0 ./easy-rsa
vim easy-rsa/vars
{% endhighlight %}

Edit accordingly:

{% highlight powershell %}
export KEY_COUNTRY="CH" 
export KEY_PROVINCE="ZH"
export KEY_CITY="Zurich" 
export KEY_ORG="AlenBlog" 
export KEY_EMAIL="me@myhost.mydomain"
{% endhighlight %}

Save and quit. Load the variables, clean, and build the server key and certificate:

{% highlight powershell %}
source ./easy-rsa/vars
./easy-rsa/clean-all
./easy-rsa/build-ca
./easy-rsa/build-key-server ch-server
{% endhighlight %}

As soon as the server ca.crt and ch-server.key are set, we can proceed towards creating the client keys. Note that we will have one special client, namely the mk-gateway client. This step must be repeated for ever clients you want to allow on your VPN. Note that the clients are identified by the ‘Common Name’.

{% highlight powershell %}
./easy-rsa/build-key mk-gateway
./easy-rsa/build-key astojanov-mac
{% endhighlight %}

Now let’s create Diffie Hellman parameters:

{% highlight powershell %}
./easy-rsa/build-dh
{% endhighlight %}

After this is done, we are ready to setup the OpenVPN configuration file.

OpenVPN server configuration
----------------------------

Since the main goal is to watch Macedonian TV, the router will be configured such that I do not have to turn on / off the VPN connection whenever I want to watch the TV stream. However, it was is also useful to have the opportunity to transfer the whole traffic to the mk-gateway. Therefore the client will make the decision of what routes will be redirected to the mk-gateway. In other words the OpenVPN will route complete or selective trafic to a client. The server configuration file is as simple as possible.

{% highlight powershell %}
dev tun
proto udp
port 1194
ca /etc/openvpn/ca.crt
cert /etc/openvpn/ch-sever.crt
key /etc/openvpn/ch-server.key
dh /etc/openvpn/dh1024.pem
user nobody
group nogroup
server 192.168.2.0 255.255.255.0
persist-key
persist-tun
route 192.168.0.0 255.255.255.0
status /var/log/openvpn-status.log
ifconfig-pool-persist /etc/openvpn/ipp.txt
client-config-dir /etc/openvpn/ccd
verb 3
client-to-client
{% endhighlight %}

The server uses UDP as a transport protocol (although less reliable than TCP, it is quite faster than TCP), running on port 1194. For in-depth knowledge for the rest of the directives, I strongly advise the OpenVPN manpage.

Note the client-config-dir directive. It provides the flexibility to add specific configurations to the clients. We configure the mk-gateway here.

{% highlight powershell %}
mkdir -p /etc/openvpn/ccd
vim /etc/openvpn/ccd/mk-gateway
{% endhighlight %}

In order to make mk-gateway route any specific traffic, we use the iroute directive. Ideally we would like to route 0/1 to the client and set something like:

{% highlight powershell %}
iroute 0.0.0.0 128.0.0.0
{% endhighlight %}

However, THIS DOES NOT WORK in OpenVPN. Good news is that instead of using one general route, we can set routes from 1.0.0.0 to 255.0.0.0 using netmask 255.0.0.0 (thanks to fuzzie’s post for his insights) which will do exactly the same. We also like a static IP for the mk-gateway:

{% highlight powershell %}
ifconfig-push 192.168.2.250 192.168.2.249
iroute 1.0.0.0 255.0.0.0
iroute 2.0.0.0 255.0.0.0
iroute 3.0.0.0 255.0.0.0
. . . . . . 

iroute 254.0.0.0 255.0.0.0
iroute 255.0.0.0 255.0.0.0
{% endhighlight %}

By now, the configuration for the OpenVPN server is complete. The server can be started with:

{% highlight powershell %}
/etc/init.d/openvpn start
{% endhighlight %}

Setting up the remote gateway (mk-gateway). Optional DD-WRT setup.
------------------------------------------------------------------

Initially I configured the gateway to run on Windows 7 machine. Since this machine will be forwarding packets, the OS must be configured to enable forwarding. According to Bebop’s post, the following tweaks should do the job:

Start -> Right-click My Computer -> Manage -> Services. Right-click Routing and Remote Access -> Properties -> Automatic. Right-click Routing and Remote Access -> Start
Control Panel -> Network and Sharing Center -> Local Area Connection -> Properties -> Sharing. Tick the box “Allow other network users to connect through this computer’s Internet connection”. From the drop-down list select “Local Area Connection 2”, or whatever is the connection name of your TUN connection.
Run regedit. Navigate to HKEY_LOCAL_MACHINE and then to SYSTEM\CurrentControlSet\Services\Tcpip\Parameters. Change value of IPEnableRouter to 1.
OpenVPN GUI for Windows is a decent OpenVPN client for Windos, including GUI, as mentioned in its title. In order to set it up, download it, install it and copy the files /etc/openvpn/ca.crt, /etc/openvpn/mk-gateway.crt and /etc/openvpn/mk-gateway.key into C:\Program Files\Open VPN\config\ and finally create the config file config.opvn

{% highlight powershell %}
dev tun
proto udp
remote ch-sever 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert mk-gateway.crt
key mk-gateway.key
verb 3
route-method exe
route-delay 2
{% endhighlight %}

The GUI client will enable / disable the tun device and setup the routes in the system. Therefore in Windows 7 / Vista it need administrator permissions. Make sure that you right-click on the GUI executable in C:\Program Files\OpenVPN and check the ‘Run as administrator’ checkbox.

OpenVPN client running on DD-WRT
--------------------------------

Having a computer running 24/7 just for routing is not really desirable. With a decent router having OpenVPN support, one can bypass the need for an extra computer. I personally have WRT54GL in Macedonia. It is possible to install DD-WRT on the router and ‘unleash’ the OpenVPN client support in the router itself. Setting up DD-WRT in some cases is router specific, and its installation has been well documented on the DD-WRT wiki.

In order to setup DD-WRT on a router, one needs to flush the current firmware, and replace it with a DD-WRT, which might be a risky business. Note that the first step is flushing the router with the ‘mini firmware’ available at the DD-WRT website, and then the next step is installing the OpenVPN supported firmware. When the final firmware is installed, setting up the OpenVPN client can be done via the Web interface in Services -> VPN. The files /etc/openvpn/ca.crt, /etc/openvpn/mk-gateway.crt and /etc/openvpn/mk-gateway.key can simply be copy-pasted into the corresponding fields:

![MaxTV](/img/ddwrt.png)

Depending on the router CPU, you can enable or disable the LZO Comporession. I have actually disabled it in my final configuration since WRT54GL has 200MHz CPU.

The above creates a connection to the OpenVPN server ch-server as soon as the router is rebooted. Before rebooting, packet forwarding within the router must be enabled. This can be done in Administration -> Commands, by setting the following commands for the Firewall.

{% highlight powershell %}
iptables -I FORWARD 1 --source 192.168.2.0/24 -j ACCEPT
iptables -I FORWARD -i br0 -o tun0 -j ACCEPT
iptables -I FORWARD -i tun0 -o br0 -j ACCEPT
{% endhighlight %}

By clicking Save Firewall, the commands will allow packets to flow to/from the ch-sever to the mk-gateway.

Setting up the client such that the whole traffic is redirected to the remote gateway

The client `astojanov-mac`, runs Mac OS X. I am using Tunnelblick as an OpenVPN client GUI. To set it up, the generated key & certificates: /etc/openvpn/ca.crt, /etc/openvpn/astojanov-mac.crt and /etc/openvpn/astojanov-mac.key must be copied to the client, and config.opvn file must be created:

{% highlight powershell %}
client
dev tun
proto udp
remote ch-server 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert astojanov-mac.crt
key astojanov-mac.key
verb 3
redirect-gateway def1
dhcp-option DNS 8.8.8.8
{% endhighlight %}

Note the redirect-gateway def1 directive. This directive forces the client to change its default gateway and redirect it to the OpenVPN server. Since the mk-gateway takes all the routes from 1.0.0.0 to 255.0.0.0, the whole traffic will be redirected to mk-gateway.

Setting up the client to route selective traffic via a remote gateway

For this scenario, I use most of the previous settings for redirecting the whole traffic and Tunnelblick, with a modified config.opvn file. In order to perform selective routing, instead of redirecting the gateway, we need to rewrite the routing rules to the specific selective trafic that we are planning to redirect. I personally wanted scenario where all Macedonian web sites hosted in Macedonia will be redirected through the mk-gateway. To identify all Macedonian hosts, I used the NirSoft country based IP blocks. The blocks are given by their price ranges, and the number of hosts for each block. With a simple math, it is easy to derive the netmasks for each block taking into considerations the number of nodes.

Finally the config file looks like the following:

{% highlight powershell %}
client
dev tun
proto udp
remote ch-server 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert astojanov-mac.crt
key astojanov-mac.key
verb 3
route 31.11.64.0 255.255.192.0 # Cabletel DOOEL Skopje
route 78.157.0.0 255.255.224.0 # Cabletel DOOEL Skopje
route 92.53.0.0 255.255.192.0 # Cabletel DOOEL Skopje
route 188.44.0.0 255.255.224.0 # GIV Ivan LTD Gostivar
route 212.120.0.0 255.255.224.0 # NETCETERA DOOEL Skopje
route 95.86.0.0 255.255.192.0 # Inel Dooel Kavadarci
route 79.141.112.0 255.255.240.0 # INFEL-KTV DOO
route 213.135.160.0 255.255.224.0 # KDS-Kabel Net DOOEL
route 212.110.64.0 255.255.224.0 # Macedonia On-Line
route 46.217.0.0 255.255.0.0 # Makedonski Telekom
route 62.162.0.0 255.255.0.0 # Makedonski Telekom
route 62.220.192.0 255.255.224.0 # Makedonski Telekom
route 77.28.0.0 255.254.0.0 # Makedonski Telekom
route 79.125.128.0 255.255.128.0 # Makedonski Telekom
route 195.26.128.0 255.255.224.0 # Makedonski Telekom
route 89.205.0.0 255.255.128.0 # MEGANET
route 94.100.96.0 255.255.240.0 # Miksnet
route 217.196.192.0 255.255.240.0 # Miksnet
route 80.77.144.0 255.255.240.0 # NEOTEL Skopje
route 88.85.96.0 255.255.224.0 # NEOTEL Skopje
route 92.55.64.0 255.255.192.0 # NEOTEL Skopje
route 95.180.128.0 255.255.128.0 # NEOTEL Skopje
route 79.126.128.0 255.255.128.0 # ONE Telecom
route 85.30.64.0 255.255.192.0 # ONE Telecom
route 212.158.176.0 255.255.240.0 # ONE Telecom
route 217.16.64.0 255.255.240.0 # ONE Telecom
route 217.16.80.0 255.255.240.0 # ONE Telecom
route 151.236.240.0 255.255.240.0 # PET NET DOO Gevgelija
route 31.12.16.0 255.255.240.0 # T-Mobile Makedonija
route 95.156.0.0 255.255.192.0 # T-Mobile Makedonija
route 146.255.64.0 255.255.224.0 # TELESMART TELEKOM DOO
route 89.185.192.0 255.255.224.0 # TRD " Net Kabel"
route 212.13.64.0 255.255.224.0 # UltraNet d.o.o.
route 194.149.128.0 255.255.224.0 # "Sv. Kiril i Metodij"
{% endhighlight %}


Linux clients
--------------------------------

For the linux users, particularly, the linux clients, setting up openvpn in a client mode is straight forward. We use the same keys and certificates as explained above. The content of the config file remains the same and its renamed to client.conf. All the files should be placed into /etc/openvpn and the client is started with:

{% highlight powershell %}
/etc/init.d/openvpn start client
{% endhighlight %}