# ⚡ Cheatsheet — Portas Importantes

> Referência rápida das portas mais relevantes em pentest.

---

## 🔢 Well-Known Ports (0–1023)

| Porta | Protocolo | Serviço | Relevância em Pentest |
|-------|-----------|---------|----------------------|
| **21** | TCP | FTP | Login anônimo, upload de arquivos |
| **22** | TCP | SSH | Brute force, credenciais padrão |
| **23** | TCP | Telnet | Credenciais em texto claro |
| **25** | TCP | SMTP | Email, relay aberto, enumeração de usuários |
| **53** | TCP/UDP | DNS | Zone transfer, enumeração, tunelamento |
| **67/68** | UDP | DHCP | Rogue DHCP, starvation |
| **69** | UDP | TFTP | Sem autenticação |
| **79** | TCP | Finger | Enumeração de usuários |
| **80** | TCP | HTTP | Web attacks (SQLi, XSS, LFI, RFI…) |
| **88** | TCP | Kerberos | Pass-the-ticket, AS-REP roasting |
| **110** | TCP | POP3 | Brute force, emails |
| **111** | TCP/UDP | RPC | Enumeração NFS |
| **119** | TCP | NNTP | — |
| **135** | TCP | MSRPC | Enumeração Windows, relay |
| **139** | TCP | NetBIOS | SMB legacy, enumeração |
| **143** | TCP | IMAP | Brute force, emails |
| **161/162** | UDP | SNMP | Enumeração com community string padrão |
| **389** | TCP | LDAP | Enumeração AD, null bind |
| **443** | TCP | HTTPS | Web attacks sobre TLS |
| **445** | TCP | SMB | EternalBlue, relay, shares, enum usuários |
| **512-514** | TCP | RSH/Rexec/Rlogin | Credenciais sem criptografia |
| **587** | TCP | SMTP (submission) | Relay autenticado |
| **631** | TCP | CUPS (IPP) | Impressoras |
| **636** | TCP | LDAPS | LDAP sobre TLS |
| **873** | TCP | rsync | Acesso a arquivos sem auth |

---

## 🔢 Registered Ports (1024–49151)

| Porta | Protocolo | Serviço | Relevância |
|-------|-----------|---------|-----------|
| **1080** | TCP | SOCKS proxy | Proxy para pivoting |
| **1194** | UDP | OpenVPN | VPN |
| **1433** | TCP | MSSQL | Injeção SQL, xp_cmdshell |
| **1521** | TCP | Oracle DB | Injeção SQL |
| **2049** | TCP/UDP | NFS | Montar shares sem auth |
| **2375/2376** | TCP | Docker | RCE via API exposta |
| **3306** | TCP | MySQL | Injeção SQL, credenciais |
| **3389** | TCP | RDP | Brute force, BlueKeep |
| **4444** | TCP | Metasploit default | Callback de reverse shell |
| **5432** | TCP | PostgreSQL | Injeção SQL |
| **5900** | TCP | VNC | Sem auth, brute force |
| **5985/5986** | TCP | WinRM | PSRemoting, evil-winrm |
| **6379** | TCP | Redis | Sem auth por padrão |
| **8080** | TCP | HTTP alt | Painéis admin, proxies |
| **8443** | TCP | HTTPS alt | Admin panels |
| **8888** | TCP | Jupyter Notebook | RCE via notebook |
| **9090** | TCP | Prometheus / Cockpit | — |
| **9200/9300** | TCP | Elasticsearch | Dados expostos sem auth |
| **27017** | TCP | MongoDB | Sem auth por padrão |

---

## 🎯 Agrupamentos por Tema

### Windows / Active Directory
```
88    Kerberos
135   MSRPC
139   NetBIOS
389   LDAP
445   SMB
3389  RDP
5985  WinRM
```

### Bancos de Dados
```
1433  MSSQL
1521  Oracle
3306  MySQL
5432  PostgreSQL
6379  Redis
27017 MongoDB
```

### Web
```
80    HTTP
443   HTTPS
8080  HTTP alternativo
8443  HTTPS alternativo
8888  Jupyter
```

### File Transfer
```
21    FTP
69    TFTP
873   rsync
2049  NFS
445   SMB
```

---

## 🔎 Comandos de Verificação Rápida

```bash
# Verificar serviço em porta específica
nc -nv IP PORTA
curl -i http://IP:PORTA
nmap -sV -p PORTA IP

# Banner grabbing manual
echo "" | nc -nv IP 21    # FTP
echo "" | nc -nv IP 22    # SSH
echo "" | nc -nv IP 25    # SMTP
```

---

## 📌 Relacionado

- [[Cheatsheet — Nmap Flags]]
- [[Cheatsheet — Recon Workflow]]
- [[Transport Layer — TCP & UDP]]

#cheatsheet #protocolo/tcp #protocolo/udp
