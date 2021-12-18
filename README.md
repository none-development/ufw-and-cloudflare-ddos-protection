# UFW DDoS Protection

<p align="center">
  <img src="https://res.cloudinary.com/practicaldev/image/fetch/s--pnF87Zxf--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vfiufomeykte9vrqrfri.png">
</p>

----

# About

Uses Uncomplicated Firewall to rate-limit requests on spicule ports. 

In addition, here are some Cloudflare and UFW settings that can lead to improve the visibility of the server and the protection against DDoS attacks.

If you like this please like it or

[Follow me on GitHub](https://github.com/none-development/)

**LETS START**

----

## Setup

Install UFW:

```
sudo apt install ufw -y
```

After install go to the UFW Config. You can use any text editor in this example I use nano.

```
sudo nano /etc/ufw/before.rules
```

To create a Rate Limit on Port **80** and **443**

Add this on the Top under `*filter`
```
:ufw-http - [0:0]
:ufw-http-logdrop - [0:0]
```

<p align="center">
  <img src="https://cdn.discordapp.com/attachments/857885707183325185/921745003838066688/unknown.png">
</p>

Go down between the 2 commands add this:

```
# START RATE LIMIT SETTINGS

-A ufw-before-input -p tcp --dport 80   -j ufw-http
-A ufw-before-input -p tcp --dport 443  -j ufw-http
-A ufw-http -p tcp --syn -m connlimit --connlimit-above 50 --connlimit-mask 24 -j ufw-http-logdrop
-A ufw-http -m state --state NEW -m recent --name conn_per_ip --set
-A ufw-http -m state --state NEW -m recent --name conn_per_ip --update --seconds 10 --hitcount 20 -j ufw-http-logdrop
-A ufw-http -m recent --name pack_per_ip --set
-A ufw-http -m recent --name pack_per_ip --update --seconds 1  --hitcount 20  -j ufw-http-logdrop
-A ufw-http -j ACCEPT
-A ufw-http-logdrop -j DROP

# END RATE LIMIT SETTINGS
```
<p align="center">
  <img src="https://cdn.discordapp.com/attachments/857885707183325185/921745360181940344/unknown.png">
</p>

To protect your server from being found by bots you can use these settings

Search for the comment 

`# ok icmp codes for INPUT`

Now DENY all theres Settings:
```
-A ufw-before-input -p icmp --icmp-type destination-unreachable -j DROP
-A ufw-before-input -p icmp --icmp-type time-exceeded -j DROP
-A ufw-before-input -p icmp --icmp-type parameter-problem -j DROP
-A ufw-before-input -p icmp --icmp-type echo-request -j DROP
```

Now the UFW Config should look like this:

<p align="center">
  <img src="https://cdn.discordapp.com/attachments/857885707183325185/921746064288137246/unknown.png">
</p>

Now safe the File.

Don't forget to allow your SSH port with the command (Standard Port)

```
ufw allow 22/tcp
```

Now Start UFW
```
ufw enable
```
----

# Cloudflare Settings

These settings go into the Free Plan and can be set easily

Go to the Point `Firewall` and select `Firewall Rules`

<p align="center">
  <img src="https://cdn.discordapp.com/attachments/857885707183325185/921747025287073792/unknown.png">
</p>

Now add this 5 Rules to your Page: 

Rule 1, Block Countrys: 

`(not ip.geoip.country in {"GB" "US" "FR" "ES" "DE" "IT" "CA" "SE" "DK" "FI" "NZ" "AT" "AU" "IE" "CH" "BE" "CY"})`

Rule 2 (Use this only if you don't need any other request methods except POST and GET.):

`(http.request.method eq "GET" and http.request.method eq "POST")`

Rule 3 Block HTTP:

`(not ssl)`

Rule 4 Thread Score:

`(cf.threat_score eq 3 and cf.threat_score gt 2)`

Rule 5 No know Bots(Optional):

`(cf.client.bot)`
