---
description: >-
  Hyper-Text Transport Protocol (HTTP(s)) is a communications protocol that
  allows data to be fetched and received from remote Internet sources.
---

# 🕸 80, 443/HTTP(s)

{% content-ref url="../../web-page-methodology.md" %}
[web-page-methodology.md](../../web-page-methodology.md)
{% endcontent-ref %}

## Web Methodology:

1. Start by identifying the technologies used by the web server. Look for tricks to keep in mind once you identify key pieces of tech.
   1. Is there a known vulnerability of the version of technology?
   2. Is there any useful trick to extract additional information?
   3. Can you utilize a specialized scanner? i.e. wpscan.
2.  Launch general purpose scanners such as nikto.

    ```python
    nikto -h <URL>

    whatweb -a 4 <URL>

    wapiti -u <URL>

    w3af
    ```
3. Check robots, sitemap, 404 errors, etc.
4. Run a directory bruteforce.
5.  Once you have identified the web technology, use the following source to find vulnerabilities:

    [80,443 - Pentesting Web Methodology](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web#web-tech-tricks)
6.  CMS Scanning:

    cmsmap

    wpscan

    joomscan

    joomlavs.rb
7. Visual Inspection:

* While you have a form of recon going on in the background, visit the site and click around. See what you can discover.

There are 2 main ways of enumerating a web server: directory bruteforcing and web technology enumeration.

## VHOST Discovery (FFUF):

```python
ffuf -c -ac -w ~/tools/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H 'Host: Fuzz.forget.htb' -u <http://forge.htb>
```

## Directory Bruteforcing:

Gobuster

* This tool is used for enumerating DNS and web server directories via bruteforcing a pre-defined wordlist.

Directory

```
gobuster dir --url <http://example.com> --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

DNS

```
gobuster dns --domain example.com --wordlist /path/to/list
```

## Web Technology Enumeration:

Banner Grabbing:

```python
nc -v domain.com 80 # GET / HTTP/1.0
openssl s_client -connect domain.com:443 # GET / HTTP/1.0
```

Wappalyzer- Browser Extension

* Check version numbers
  * Google versions for exploits
* What programming language is being utilized on the web server/web app?

Nikto

* Generalization tool for enumeration.

## 401 & 403 Error Bypasses:

[403 & 401 Bypasses](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/403-and-401-bypasses)

[https://github.com/carlospolop/fuzzhttpbypass](https://github.com/carlospolop/fuzzhttpbypass)

### FuzzHTTPBypass:

```python
./fuzzhttpbypass.py -f notcontains,403 -u <http://example.com/index.php>

./fuzzhttpbypass.py -f notcontains,240 -u <http://example.com/index.php>

./fuzzhttpbypass.py -f notcontains,Invalid -u <http://example.com/index.php>
```

### HTTP Headers Fuzzing:

```python
# Attempt to add the following to the GET request:

X-Originating-IP: 127.0.0.1
X-Forwarded-For: 127.0.0.1
X-Forwarded: 127.0.0.1
Forwarded-For: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-ProxyUser-Ip: 127.0.0.1
X-Original-URL: 127.0.0.1
Client-IP: 127.0.0.1
True-Client-IP: 127.0.0.1
Cluster-Client-IP: 127.0.0.1
X-ProxyUser-Ip: 127.0.0.1
Host: localhost
```

If the path is protected, you can do the following to attempt to bypass the path protection:

```python
X-Original-URL: /admin/console
X-Rewrite-URL: /admin/console
```

### Path Fuzzing (If /path is blocked):

* Try using /%2e/path _(if the access is blocked by a proxy, this could bypass the protection). Try also_\*\* /%252e\*\*/path (double URL encode).
* Try Unicode bypass: /%ef%bc%8fpath (The URL encoded chars are like "/") so when encoded back it will be //path and maybe you will have already bypassed the /path name check Other path bypasses:

```python
site.com/secret –> HTTP 403 Forbidden
site.com/SECRET –> HTTP 200 OK
site.com/secret/ –> HTTP 200 OK
site.com/secret/. –> HTTP 200 OK
site.com//secret// –> HTTP 200 OK
site.com/./secret/.. –> HTTP 200 OK
site.com/;/secret –> HTTP 200 OK
site.com/.;/secret –> HTTP 200 OK
site.com//;//secret –> HTTP 200 OK
site.com/secret.json –> HTTP 200 OK (ruby)
```

Use all this list in the following situations:

```python
/FUZZsecret
/FUZZ/secret
/secretFUZZ
```

### Other API bypasses:

```python
/v3/users_data/1234 --> 403 Forbidden
/v1/users_data/1234 --> 200 OK
{“id”:111} --> 401 Unauthriozied
{“id”:[111]} --> 200 OK
{“id”:111} --> 401 Unauthriozied
{“id”:{“id”:111}} --> 200 OK
{"user_id":"<legit_id>","user_id":"<victims_id>"} (JSON Parameter Pollution)
user_id=ATTACKER_ID&user_id=VICTIM_ID (Parameter Pollution)
```

## Automation Tools (For Bypass)

[https://github.com/lobuhi/byp4xx](https://github.com/lobuhi/byp4xx)

Installation:

```python
pip install git+https://github.com/lobuhi/byp4xx.git
```

Example:

```python
python3 byp4xx.py <https://www.google.es/test>
```

## wpscan (with API key)

{% embed url="https://wpscan.com/register" %}

* Register user
* Use API key from your user
* Use fake information

```
wpscan --url http://www.holo.live --api-token <API-key-here> -e u,vp
```

Conduct a brute force:

```
wpscan --url http://www.holo.live --api-token <API-key-here> -w /usr/share/wordlists/rockyou.txt -u Admin
```

* Check robots.txt

## Vhosts and Subdomains

* Vhosts and subdomains look very similar
* Vhosts are a cluster of different hosts behind one IP address
* Subdomains are additional parts to your domain
* If you already see that you have a subdomain and had to add to /etc/hosts, you should enumerate further to rule out the possibility of there being more

Gobuster

```
gobuster vhost -u http://holo.live -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 30
```

* Add findings to /etc/hosts

dnsmap

```
dnsmap example.com -w /usr/share/wordlists/dnsmap.txt
```
