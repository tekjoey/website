---
title: Dipping my toes into High Availability
date: 2024-12-20
tags: 
categories:
  - Homelab
draft: false
---
I recently followed a tutorial put out by TechnoTim called "[PostgreSQL Clustering the Hard Way](https://www.youtube.com/watch?v=RHwglGf_z40&t=2457s&pp=ygUJdGVjaG5vdGlt)". It is a fantastic video, he does a good job of walking you step by step though the instructions all while explaining what each thing is doing and what each line of configuration means. After following his tutorial and having my own HA PostgreSQL cluster, I wondered if I could do the same with some webservers. Certainly it should be easy, right? After all, a simple webservers should be the easiest thing to configure because they've been around as long as the internet has.

>***My methods in this post are not production ready. This is a quick-and-dirty project I did as a proof of concept; a real HA webserver should have TLS and proper Apache2 sites configured, as well as probably some other things I don't know about.***

## Prerequisite Configuration

Some prerequisites before we get started. I have four servers setup as VMs on a Proxmox cluster. They are all fresh installs of Debian.

| Server | IP             | application         |
| ------ | -------------- | ------------------- |
| SVR-01 | 192.168.100.10 | apache2             |
| SVR-02 | 192.168.100.11 | apache2             |
| HA-01  | 192.168.100.20 | HAProxy, keepalived |
| HA-02  | 192.168.100.21 | HAProxy, keepalived |

The way this will work is `keepalived` will create a virtual IP (VIP) address that our clients can connect to, in my setup that will be `192.168.100.60`. The MAC address that this IP will resolve to (and therefore the machine that our clients should send their requests to) will change depending on who `keepalived` determines is the "MASTER", at first this will be HA-01, but if it goes down, then HA-02 will become the master. `haproxy` on whichever node is the "MASTER" will then equally balance the load between SVR-01 and SVR-02. 
## Setting up my own HA webserver

To have a highly available webserver, one needs multiple webservers. I reused the same VMs I had setup for TechnoTims tutorial (SVR-01 and 02) and just installed Apache2 alongside PostgreSQL.

```
sudo apt install apache2
```

Then, just to make sure it installed correctly, I navigated to the respective IP addresses for SVR-01 and SVR-02 in my web browser. On both I saw the default Apache2 page, so I knew the installation had succeeded. 

Ordinarily, these two servers would have the same content (synced using `rsync` periodically perhaps), but I changed the default `index.html` page to include the server name so I could differentiate between the two. I put the following in `/var/www/html/index.html`, changing the `svr-0x` number appropriately.

```
<html>
<body>

<!-- Change this number depending on the web server name -->
<h1> This is svr-01</h1>

</body>
</html>
```

I checked both once again, and they both showed my simple page. 

Now onto `haproxy`. It can be installed through the package manager as well.

`sudo apt install haproxy`

Once it was installed we can add the configuration. I was able to figure out by just resuming most of TechnoTim's configuration. I put the following in `/etc/haproxy/haproxy.cfg` on HA-01

``` 
frontend apache_frontend
    bind *:80
    mode http
    default_backend apache_backend

backend apache_backend
    mode http
    server apache-01 192.168.100.10:80 check
    server apache-02 192.168.100.11:80 check
```

The `frontend` section creates the receiving part, this is the port and configuration that HAProxy is listening for. The backend section is then what HAProxy will do with that incoming request. In this case, simply forward it on to one of the defined servers. The `check` parameter at the end of each server definition tells `haproxy` to periodically health check the endpoint. More information in that is available on [HAProxy's Blog](https://www.haproxy.com/blog/how-to-enable-health-checks-in-haproxy#active-health-checks)

Then a quick restart for `HAProxy`

```
sudo systemctl restart haproxy
```

Then navigating to the IP address of HA-01 I saw the page for SVR-01. Refreshing the page quickly a bunch of times forced it to load balance and so I also saw the page for SVR-02! Everything was working! I then copied the same configuration to HA-02.

The last thing to get working is the Virtual IP using `keepalived`, so that `haproxy` doesn't become a single point of failure. This took a little bit more investigation. I am thankful for the information at [this site](https://packetpushers.net/blog/vrrp-linux-using-keepalived-2/).

First, install `keepalived` on HA-01

```
sudo apt install keepalived
```

Then add the following to `/etc/keepalived/keepalived.conf`

```
vrrp_instance web_server {
        state MASTER
        interface eth0
        virtual_router_id 20
        priority 100
        advert_int 1
        virtual_ipaddress {
                192.168.100.60
        }
}
```

Copy the same to HAProxy-02, changing the `state` to `BACKUP` and the `priority` to any number lower than 100. Then restart `keepalived` on both HA-01 and HA-02.

```
sudo systemctl restart keepalived
```

Navigating to `192.168.100.60` and quickly refreshing the page leads to the same result as earlier. 
This means our VIP was correctly resolved to HA-01's MAC address, which took our request and balanced it between SVR-01 and SVR-02. We can test this even further by shutting down the `haproxy` node that is the master, in this case HA-01. `keepalived` on HA-02 should notice the drop out and assign itself to be the new "MASTER" and `haproxy` on HA-02 will then proxy the request to the webservers. If you had more than two `keepalived` nodes, then the highest `priority` value always becomes the new "MASTER". 

## HA Wacky Webserver
While doing the above, I realized that `keepalived` has no interest in port numbers, it is Layer 3 only, and `haproxy` has definitions for port numbers for each server declaration. Would it be possible then to have two webservers on different ports (say, 9080 and 9081) get routed though a virtual IP that was itself listening for web traffic on a different port (say 9090)? Only one way to find out!

First I needed to make some web service available on a separate port. I looked into the APache2 documentation and saw they you can define multiple Listen interfaces, as well as multiple VirtualHosts. 

So, on SVR-01 in `/etc/apache2/ports.conf` I added a second Listen directive. Now this VM is listening on both port `80` and port `9080`.

```
Listen *:80
Listen *:9080
```

Then in `/etc/apache2/sites-available/000-default.conf` I added a second VirtualHost. This defines what should happen when a request comes in on a particular `IP:Port` combination. Here we are telling Apache that a request that comes in on any IP address defined on this host over port `9080` should receive the contents in `/var/www/html2`

```
<VirtualHost *:9080>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html2
        ErrorLog ${APACHE_LOG_DIR}/error2.log
        CustomLog ${APACHE_LOG_DIR}/access2.log combined
</VirtualHost>
```

Then, to create the new `html2` directory and index file

```
sudo mkdir /var/www/html2
sudoedit /var/www/html2/index.html
```

```
<html>
<body>

<!-- Change these numbers depending on the web server -->
<h1> This is wacky-svr-01 port 9080-</h1>

</body>
</html>
```

I did similar things on SVR-02, except everywhere I put `9080` for the port number on SVR-01, I put `9081` on SVR-02. I was then able to access each new page at their respective IP:port combos.

Then, on HA-01, I just had to add one new frontend and backend

```
frontend wacky_frontend
    bind *:9090
    mode http
    default_backend wacky_backend

backend wacky_backend
    mode http
    server wacky-01 192.168.100.10:9080 check
    server wacky-02 192.168.100.11:9081 check
```

Notice how I defined the `wacky_frontend` to be listening on port `9090`. 

Accessing HA-01's IP address and port `9090` in my web browser brought be to the webpage for wacky-01. Once more, spamming the refresh button also showed the wacky-02 page. I was able to load balance between two different machines using three different ports! Pretty cool!

Of course this could also be extended to include HA-02 and a new `keepalived` IP address. I decided not to do that.

# Conclusion
I have known about load balancing and Virtual IPs for a while now and always kind of assumed it was difficult or complicated to setup. I'm glad to see that for simple projects like this, it really wasn't that difficult. This gives me the courage to add Virtual IPs to more services in my homelab, such as DNS.

**Challenge:** Can you configure `haproxy` to not load balance equally? Maybe change it from the default of (Ibelive) 50/50 to something like 90/10. Or change it to simply failover when one webserver goes down.