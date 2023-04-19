---
description: 04/18/2022
---

# Spectra (Easy)

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

## Information Gathering

Scanned all TCP ports:

<pre><code>PORT     STATE SERVICE VERSION
<strong>22/tcp   open  ssh     OpenSSH 8.1 (protocol 2.0)
</strong>80/tcp   open  http    nginx 1.17.4
3306/tcp open  mysql   MySQL (unauthorized)
</code></pre>

Enumerated UDP ports:

```
PORT     STATE         SERVICE  VERSION
68/udp   open|filtered dhcpc
5353/udp open|filtered zeroconf
```

Notes:



## Enumeration

### Port 80 - HTTP (nginx 1.17.4)

* Wordpress is running at <mark style="color:yellow;">spectra.htb/main</mark>

#### Directory Bruteforce

dirsearch:

```
dirsearch -u http://spectra.htb

Target: http://spectra.htb/

[22:05:12] Starting: 
[22:05:30] 200 -  283B  - /index.html
[22:05:32] 301 -  169B  - /main  ->  http://spectra.htb/main/
[22:05:33] 200 -   25KB - /main/
[22:05:41] 301 -  169B  - /testing  ->  http://spectra.htb/testing/
```

ffuf:

```
ffuf -c -w /usr/share/wordlists/seclists/Discovery/Web-Content/quickhits.txt -u http://spectra.htb/testing/FUZZ -ac
[Status: 200, Size: 19915, Words: 3331, Lines: 385, Duration: 34ms]
    * FUZZ: /license.txt

[Status: 200, Size: 7278, Words: 740, Lines: 98, Duration: 33ms]
    * FUZZ: /readme.html

[Status: 200, Size: 2888, Words: 425, Lines: 91, Duration: 33ms]
    * FUZZ: /wp-config.php.save

[Status: 200, Size: 69, Words: 15, Lines: 1, Duration: 37ms]
    * FUZZ: /wp-content/plugins/akismet/akismet.php

[Status: 500, Size: 2804, Words: 223, Lines: 121, Duration: 385ms]
    * FUZZ: /wp-admin/setup-config.php
```

* <mark style="color:yellow;">/wp-config.php.save -- These .save files appear when backups are made on the server!</mark>
  * <mark style="color:yellow;">They may contain juicy information!</mark>
  * <mark style="color:yellow;">Since it is a .php file, view the source!</mark>

<mark style="color:red;">MySQL Credentials</mark> found in wp-config.php.save file:

```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'dev' );

/** MySQL database username */
define( 'DB_USER', 'devtest' );

/** MySQL database password */
define( 'DB_PASSWORD', 'devteam01' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );
```

feroxbuster:

```
feroxbuster -u http://spectra.htb -s 200,301,403

200      GET        6l       25w      283c http://spectra.htb/
301      GET        7l       11w      169c http://spectra.htb/main => http://spectra.htb/main/
301      GET        7l       11w      169c http://spectra.htb/testing => http://spectra.htb/testing/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes => http://spectra.htb/main/wp-includes/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content => http://spectra.htb/main/wp-content/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin => http://spectra.htb/main/wp-admin/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/plugins => http://spectra.htb/main/wp-content/plugins/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/themes => http://spectra.htb/main/wp-content/themes/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js => http://spectra.htb/main/wp-includes/js/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/images => http://spectra.htb/main/wp-includes/images/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/css => http://spectra.htb/main/wp-includes/css/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/uploads => http://spectra.htb/main/wp-content/uploads/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/assets => http://spectra.htb/main/wp-includes/assets/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/languages => http://spectra.htb/main/wp-content/languages/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/upgrade => http://spectra.htb/main/wp-content/upgrade/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/blocks => http://spectra.htb/main/wp-includes/blocks/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/fonts => http://spectra.htb/main/wp-includes/fonts/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/languages/themes => http://spectra.htb/main/wp-content/languages/themes/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/languages/plugins => http://spectra.htb/main/wp-content/languages/plugins/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/customize => http://spectra.htb/main/wp-includes/customize/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/images/media => http://spectra.htb/main/wp-includes/images/media/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/includes => http://spectra.htb/main/wp-admin/includes/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/images => http://spectra.htb/main/wp-admin/images/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/css => http://spectra.htb/main/wp-admin/css/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/js => http://spectra.htb/main/wp-admin/js/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/user => http://spectra.htb/main/wp-admin/user/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/widgets => http://spectra.htb/main/wp-includes/widgets/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js/tinymce => http://spectra.htb/main/wp-includes/js/tinymce/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js/jquery => http://spectra.htb/main/wp-includes/js/jquery/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/js/widgets => http://spectra.htb/main/wp-admin/js/widgets/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/network => http://spectra.htb/main/wp-admin/network/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/css/dist => http://spectra.htb/main/wp-includes/css/dist/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js/dist => http://spectra.htb/main/wp-includes/js/dist/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/maint => http://spectra.htb/main/wp-admin/maint/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js/thickbox => http://spectra.htb/main/wp-includes/js/thickbox/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/images/smilies => http://spectra.htb/main/wp-includes/images/smilies/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/certificates => http://spectra.htb/main/wp-includes/certificates/
301      GET        7l       11w      169c http://spectra.htb/main/wp-admin/css/colors => http://spectra.htb/main/wp-admin/css/colors/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/Text => http://spectra.htb/main/wp-includes/Text/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js/swfupload => http://spectra.htb/main/wp-includes/js/swfupload/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/uploads/2020 => http://spectra.htb/main/wp-content/uploads/2020/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/js/crop => http://spectra.htb/main/wp-includes/js/crop/
301      GET        7l       11w      169c http://spectra.htb/main/wp-content/plugins/akismet => http://spectra.htb/main/wp-content/plugins/akismet/
301      GET        7l       11w      169c http://spectra.htb/main/wp-includes/images/crystal => http://spectra.htb/main/wp-includes/images/crystal/

```

#### Subdomain Enumeration

```
ffuf -u http://spectra.htb -H "Host: FUZZspectra.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 283
```

* No findings

#### Fingerprinting

whatweb:

<pre><code>whatweb http://spectra.htb

<strong>http://spectra.htb [200 OK] Country[RESERVED][ZZ], HTTPServer[nginx/1.17.4], IP[10.129.252.10], nginx[1.17.4]
</strong></code></pre>

#### wpscan

```
wpscan --url http://spectra.htb/main --api-token=place_api_token_here -e

WordPress version 5.4.2 identified (Insecure, released on 2020-06-10)

28 vulnerabilities identified:
```

User Identified:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

#### Nikto Web Scan

```
nikto -h http://spectra.htb

+ Target IP:          10.129.252.10
+ Target Hostname:    spectra.htb
+ Target Port:        80
+ Start Time:         2023-04-18 21:55:01 (GMT-4)
---------------------------------------------------------------------------
+ Server: nginx/1.17.4
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ /7T9Hq8ua.php#: Retrieved x-powered-by header: PHP/5.6.40.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ /testing/: Directory indexing found.
+ /testing/: This might be interesting.
+ /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.
```

## Exploitation

### Cleartext Credentials

* Cleartext credentials have been located in view-source:http://spectra.htb/testing//wp-config.php.save
* A case of password reuse has also been detected on [http://spectra.htb/main/wp-login.php](http://spectra.htb/main/wp-login.php)

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* Although it does appear broken, we still successfully authenticated
* Now that we have administrator access on wordpress, we can use the metasploit module <mark style="color:yellow;">wp\_admin\_shell\_upload</mark> to gain a meterpreter reverse shell&#x20;

###

## Privilege Escalation

### Local enumeration

### PrivEsc vector

##
