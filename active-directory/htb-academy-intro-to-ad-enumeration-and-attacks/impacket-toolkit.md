---
description: The coolest toolkit in the world
---

# Impacket Toolkit

## Introduction

* Impacket provides us with many different ways to enumerate, interact, and exploit Windows protocols and find the information we need using Python
* This tool is well maintained and has many contributors

## psexec.py

* This is <mark style="color:yellow;">one of the most useful tools</mark>
* This is a <mark style="color:yellow;">clone of the Sysinternals psexece executable</mark> but works slightly different
* <mark style="color:yellow;">The tool works by creating a remote service by uploading a randomly-named executable to the ADMIN$ share on the host</mark>
* It then registers the service via RPC
* <mark style="color:yellow;">Once established, communication occurs over a pipe, allowing for an interactive shell as SYSTEM on the victim</mark>

```
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125
```

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## wmiexec.py