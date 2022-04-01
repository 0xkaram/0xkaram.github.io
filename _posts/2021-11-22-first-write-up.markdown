---
layout: post
title:  "first Write up!"
date:   2021-11-22 00:00:37 +0200
categories: jekyll update
---
You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.


1. Nmap
{% highlight ruby %}
nmap -A -vv -p-1000 -Pn -oA nmap/10.200.120.33 10.200.120.33

Nmap scan report for 10.200.120.33
Host is up, received user-set (0.22s latency).
Scanned at 2021-11-26 19:38:25 EET for 62s
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 a8:cf:2f:61:59:3f:a6:1d:6c:62:de:8a:6a:70:a0:d4 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCwkjm7PUANDx4Hwpr7S1LFzwyVt8FIfFJ1JMpWiXLOyLcQITYOvAxDqu40+loUBid2h2Vo9q8yT4/jmDposJ3BdFM0aiRFk0WoqW/lHijcg9yrV4ee12YbQy6T5q38R7qHZ2aySI3ZIYNIqJCoEwJRgUIRbi00smuyGTN3hS37HXMwgFQcad6dGDo/IdSUCh5ZRSAuY5SpKxJPmBvH/nIA7zVwUUXi7DG7wzcXviMvGf3hvHvOXigp/8LrZApr7DNJOMYfpw2Od2bfYEfed+D04EjCmN04XtYsfPtNUqvPvZGNjS6iTjgrrKhr1cQL7m09cGXAKhP+06Bo0IfyoRnb/N0ZNS96ISAtsYK1eMpWk04XzpzejSZpaVJ02TViYol0weznOixAhuzzT5YSDa1QJqWbxo3Ayy8yYX7YAF9Mb3zC9aYpfN0oBD7ECNt64xdmf23oba4Qm1FxBsnTzL3LRWCaL/MvADaFvHJk/1Zrm4kPQUWtUs8zUtA/i8mGZBU=
|   256 6f:65:0a:bc:90:d0:85:40:35:04:d4:fb:8d:fe:ab:8b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBMvJdZ4Jm8mtOYUmormXOtQxsXF1NLPIVc6QOcjbFAPAAswjhymGgEKqV3KhXT+x6p7aSiXR0Jmz1ivC6VV0FA=
|   256 ad:8a:22:fa:b0:5c:e9:06:81:a9:cc:8d:d2:13:cc:e7 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOl9DyApbKViswhvEOg8jOVmh1+C/eilBeYgqYuu6TPe
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.29 ((Ubuntu))
|_http-title: holo.live
| http-robots.txt: 21 disallowed entries
| /var/www/wordpress/index.php
| /var/www/wordpress/readme.html /var/www/wordpress/wp-activate.php
| /var/www/wordpress/wp-blog-header.php /var/www/wordpress/wp-config.php
| /var/www/wordpress/wp-content /var/www/wordpress/wp-includes
| /var/www/wordpress/wp-load.php /var/www/wordpress/wp-mail.php
| /var/www/wordpress/wp-signup.php /var/www/wordpress/xmlrpc.php
| /var/www/wordpress/license.txt /var/www/wordpress/upgrade
{% endhighlight %}
