### ** 2. Findings and Remediations**
##### 2.1 Throwback-FW01: 10.200.154.138

###### 2.1.1 Critical Default Credentials

2.1.1.1 Infected url: [http://10.200.154.138/](http://10.200.154.138/)

    > username: admin
    > password: pfsense

###### 2.1.2 RCE as root from web page.
2.1.2.1 Infected url: [http://10.200.154.138/diag_command.php](http://10.200.154.138/diag_command.php)

    Methods to get RCE:
    > python
    > PHP
    > File upload
    > Edit SSH files

##### 2.2  Throwback-MAIL: 10.200.154.232

###### 2.2.1 Guest Credentials leaked.
2.2.1.1 Infected url: [http://10.200.154.232/src/login.php](http://10.200.154.232/src/login.php)

    > username: tbhguest
    > password: WelcomeTBH1!
