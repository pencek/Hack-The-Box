# 信息搜集

端口扫描

```
┌──(kali㉿kali)-[~]
└─$ nmap --min-rate 10000 -p- 10.129.38.11 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-10 01:22 EDT
Nmap scan report for 10.129.38.11
Host is up (7.8s latency).
Not shown: 64776 filtered tcp ports (no-response), 757 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 24.38 seconds
