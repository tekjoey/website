---
title: Dipping my toes into High Availability
date: 2024-12-12
tags: []
categories:
  - Homelab
draft: true
---
I recently followed a tutorial put out by TechnoTim called "[PostgreSQL Clustering the Hard Way](https://www.youtube.com/watch?v=RHwglGf_z40&t=2457s&pp=ygUJdGVjaG5vdGlt)". It is a fantastic video, he does a good job of walking you step by step though the instructions all while explaining what each thing is doing and what each line of configuration means.

This post will be 1) a reflection on this video and my experience setting up my first HA cluster and 2) my own quasi-tutorial as I use the tools Tim showed off to set up a much tamer HA webserver.


# Setting up my own HA webserver

---
**TODO: add requirements section for those who haven't watched the video**
---

I choose a web server because i figured it would be the easiest to get going. Webservers were one of the first things that needed to be highly available, so all the tools that exist can easily support them. Plus, it's easy to see if it's working though a web browser, instead of having to stand up a separate application.

To have a highly available webserver, one needs multiple webservers. I reused the same VMs I had setup for TechnoTims tutorial and just installed Apache2 alongside PostgreSQL.

```
sudo apt install apache2
```

Then, just to make sure it installed correctly, I navigated to the respective IP addresses in my web browser. On both I saw the default page, so I knew the installation had succeeded. Ordinarily, these two servers would have the same content, but I changed the default `index.html` page to include the server number so I could differentiate between the two. I put the following in `/var/www/html/index.html`, changing the `svr-0x` number appropriately.

```
<html>
<body>

<!-- Change this number depending on the web server -->
<h1> This is svr-01</h1>

</body>
</html>
```

I checked both once again, and they both showed my simple page. Now onto `HAProxy`. This configuration I was able to figure out without even looking at the documentation. I put the following in `/etc/haproxy/haproxy.cfg`

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

Then a quick restart for `HAProxy`

```
sudo systemctl restart haproxy
```

Then navigating to the IP address of the `HAProxy` VM I saw the page for SVR-01. Refreshing the page a bunch of times quickly forced it to load balance and so I also saw the page for SVR-02! Everything was working! 

The last thing to get working is the Virtual IP using `keepalived`, so that `HAProxy` doesn't become a single point of failure. This took a little bit more investigation. I am thankful for the information at [this site](https://packetpushers.net/blog/vrrp-linux-using-keepalived-2/).

Still on the HAProxy-01 VM, add the following to `/etc/keepalived/keepalived.conf`

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

Copy the same to HAProxy-02, changing the `state` to `BACKUP` and the `priority` to any number lower than 100. Then restart `keepalived` and navigate to the virtual IP

```
sudo systemctl restart keepalived
```

Navigating to `192.168.100.60` and quickly refreshing the page leads to the same result as earlier. This means keepalived is routing the IP to HAProxy, and HAProxy is load balancing to the two different webservers. We can test this even further by shutting down the HAProxy node that is the master. the Virtual IP should still be routed correctly to the two webservers. 

## HA Wacky Webserver
While doing the above, I realized that `keepalived` has not interest in port numbers, it is Layer 3 only, and HAProxy has definitions for port numbers for each server declaration. Would it be possible then to have two webservers on different ports (say, 9080 and 9081) get routed though a virtual IP that was itself listening for traffic on different port (say 9090). Only one way to find out!

First I needed to make some web service available on a seperate port. I looked into the APache2 documentation and saw they you can define multiple Listen interfaces, as well as multiple VirtualHosts. 

So, in `/etc/apache2/ports.conf` I added a second Listen directive

```
Listen *:80
Listen *:9080
```

Then in `/etc/apache2/sites-available/000-default.conf` I added a second VirtualHost

```
<VirtualHost *:9080>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html2
        ErrorLog ${APACHE_LOG_DIR}/error2.log
        CustomLog ${APACHE_LOG_DIR}/access2.log combined
</VirtualHost>
```

Then, to create the new html2 directory and index file...

```
sudo mkdir /var/www/html2
nano /var/www/html2/index.html
```

```
<html>
<body>

<!-- Change these numbers depending on the web server -->
<h1> This is wacky-svr-01 port 9080-</h1>

</body>
</html>
```

I did similar things on SVR-02, excpet everywhere I just put `9080` for the port number, I put `9081`. I was then able to access each new page at their respective IP:port combos.

Then, in HAProxy, I just had to add one new frontend and backend

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

Accessing HAProxy's IP address and port `9090` in my web browser brought be to the webpage for wacky-01. Once more, spamming the refresh button also showed the wacky-02 page. I was able to load balance between two different machines using three different ports! Pretty cool!

# Conclusion
I hope this has been an interesting adventure. I certainly learned a lot (from actual load balancing, to Apache2 configuration), and had what I already knew firmed up and cemented by doing this wacky project. 