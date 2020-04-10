# Hack The Box - Traverxec - Writeup by <a href="https://montyshyama.me/">Monty Shyama</a>

<p align="center">
  <img src="screenshots/1.png" width="738">
</p>

# Reconnaissance

Run a Full Nmap Scan to find all the open ports & the services associated with them.

```
nmap -sC -sV -p- -oA full-port-scan 10.10.10.160 -vvv
```
