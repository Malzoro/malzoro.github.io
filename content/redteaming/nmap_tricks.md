+++
title = 'NMAP Red-Teaming Tips and Tricks'
description = 'In this post, I will be going over a few nmap tips and tricks that i regularly use when doing red-team operations.'
tags = ["nmap", "tips", "tricks"]
date = 2023-09-23T09:08:38-04:00
draft = false
+++
Stealth may be an important factor when scanning a certain machine. Nmap IDLE scan is perfect for this, all we need to do is find "zombie hosts" which are basically just idle machines (machine not currently being used by anyone) that have an incremential IP ID

**Find Zombies** :
```
nmap -iR 1000 -p80 --script ipidseq
```

**Then you can use the found zombies to scan a target :**
```
nmap -sI <zombie ip or domain> ... <target>
```

Another cool stealth trick you can use with nmap is to use tor proxies, you don't need to use the proxychains command with nmap, instead you can use `--proxies` like the following.

**Use Nmap with Proxies :**
```
nmap -sT -Pn -v -sV -O -A --proxies socks4://127.0.0.1:9050
```

You can also trace the route of your packets using the `--trace-packet switch` like the following.

**Use Nmap with Proxies but trace the packets :**
```
nmap -sT -Pn -v -sV -O -A --proxies socks4://127.0.0.1:9050 --packet-trace
```

**You can generate random IP addresses using the following command :**
```
nmap -iR 1000 -sn --proxies socks4://127.0.0.1:9050
```

**You can use NCrack to crack most services :**
```
ncrack ssh://<ipaddress>:<port>
```

**You can specify a user with NCrack :**
```
ncrack -U admin ssh://<ipaddress>:<port>
```

**You can perform Geolocation with one of the following scripts :**
```
nmap --script ip-geolocation-maxmind, ip-geolocation-ipinfodb, ip-geolocation-geoplugin
```

**You can get WHOIS records for a domain using the following scripts :**
```
nmap -sn --script whois-* <target>
```

## Querying Shodan to obtain target information
The NSE script shodan-api needs an API key before it can be used, Shodan offers a free developer API that you can obtain [here](https://developer.shodan.io).
```
nmap -sn -Pn -n --script shodan-api --script-args shodan-api.apikey=<apikey> <target>
```

## Checking Whether a host is flagged by Google Safe Browsing for Malicious Activities
Nmap allows us to systematically check whether a host is known for distributing malware or being used in phishing attacks, with some help from the Google Safe Browsing API.

The http-google-malware script depends on google's safe browsing service, and it requires you to [register](https://developers.google.com/safe-browsing/?csw=1) to get an API key.
```
nmap -p80 --script http-google-malware --script-args http-google-malware.api=<API> <target>
```

## Collecting valid e-mail accounts and IP addresses from web servers
You can try and collect valid e-mail accounts and IP addresses from web servers using the http-grep script. The http-grep script crawls a web application and matches patterns to extract interesting information from all pages. The script will search for e-mail and IP addresses by default, but there are other built-in patterns for things such as social security or credit card numbers.

**Default :**
```
nmap -p443 --script http-grep nmap.org
```

The script http-grep can select different patterns for extraction by setting the script argument http-grep.builtins. The built-in patterns are the following.
- E-mail
- Phone
- Mastercard
- Discover
- VISA
- Amex
- SSN
- IP address

**With Patterns :**
```
nmap -p443 --script http-grep --script-args http-grep.builtins={"mastercard", "discover", "ssn", "visa", "e-mail", "ip address"}
```

## Discovering hostnames pointing to the same IP Address
Web servers return different content depending on the hostname used in the HTTP request. By discovering new hostnames, penetration testers can access new target web applications that were inaccessible using the server's IP.
```
nmap -sn --script hostmap-* <target>
```

## Discovering hostnames by bruteforcing DNS records
DNS records hold a surprising amount of host information, and by bruteforcing them we can reveal additional targets. DNS entries often give away information; for example, a DNS record type A named mail obviously indicates that we are dealing with a mail server, or Cloudflare's default DNS entry named direct most of the time will point to the IP that they are trying to protect.
```
nmap --script dns-brute <target>
```

## Checking whether a web server is an open proxy
HTTP proxies are used to make requests through their addresses, therefore hiding out real IP address from the target. Detecting them is important if you are a system administrator who needs to keep the network secure or as an attacker looking to spoof your real origin.
```
nmap --script http-open-proxy -p8080 <target>
```

## Discovering interesting files in a web server
You can discover interesting files in a web server by using the http-enum script.
```
nmap --script http-enum -p80 <target>
```

## Detecting Web Application Firewalls
Web servers are often protected by packet filtering systems that drop or redirect suspected malicious packets. Web penetration testers benefit from knowing that there is a traffic filtering system between them and the target application.
```
nmap -p80 --script http-waf-detect, http-waf-fingerprint <target>
```

## Finding web applications with default credentials
Default credentials are often forgotten in web applications and devices, such as webcams, printers, VoIP systems, video conference systems, and other applicances. There is a very useful NSE script to automate the process of testing default credentials in the network.
```
nmap -p80 --script http-default-accounts <target>
```

## Scrapping e-mail accounts from web servers
Finding valid e-mail accounts is an important task during a penetration test. E-mail accounts are often used as usernames in some systems and web applications. Attackers often target the highly sensitive information that is stored in them.
```
nmap -p80 --script http-grep --script-args http-grep.builtins=e-mail <target>
```

## Obtaining System Information from SMB
SMBv1 has an interesting feature that has been abused for years, that is that SMBv1 servers return system information without authentication. The information includes Windows version, build number, NetBIOS computer name, workgroup, and exact system time.
```
nmap -p139,445 --script smb-os-discovery <target>
```

## Detecting SMB vulnerabilities
```
nmap -p445 --script smb-vuln-* <target>
```

## Retrieving the NetBIOS name and MAC address of a host
NetBIOS name resolution is enabled in most of Windows clients today and even a debugging utility called nbtstat is shipped with Windows to diagnose name resolution problems with NetBIOS over TCP/IP.
```
nmap -sU -p137 --script nbtstat <target>
```

## Enumerating user accounts of Windows hosts
User enumeration allows attackers to conduct dictionary attacks against systems and reveals information about who has access to the. Against Windows systems, there are two known techniques to enumerate the user in the system: **SAMR enumeration** and **LSA bruteforcing**.
```
nmap -p139,445 --script smb-enum-users <target>
```

## Enumerating Shared Folders
Shared folders in organizations are very common and bad practices among users present a major risk.
```
nmap -p139,445 --script smb-enum-shares --script-args smbusername=Administrator,smbpassword=<password> <target>
```

## Enumerating SMB Sessions
SMB Sessions reflect people connected to file shares or making RPC calls and they can provide invaluable information that can be used to profile users and machines. The SMB session information includes usernames, origin IP addresses, and even idle time.
```
nmap -p445 --script smb-enum-sessions <target>
```

## Finding Domain Controllers
Domain controllers are the most important systems in Microsoft Windows networks using the AD technologyy as they control all the machines in the network and host critical services for the organization's operations such as DNS resolution.
```
nmap -p389 -sV <target>
```

Domain controllers will show port 389 running the Microsoft Windows AD LDAP service :
```
PORT STATE SERVICE VERSION
389/tcp open ldap Microsoft Windows AD LDAP (Domain:TESTDOMAIN, Site: TEST)
```
