# 🔍 enum4linux — Guia Completo

> Ferramenta de enumeração SMB/Samba que automatiza múltiplas consultas RPC, NetBIOS e SMB em um único comando.
> É o "canivete suíço" da enumeração de ambientes Windows/Samba — especialmente poderoso em null sessions.

---

## 🧠 O Que é enum4linux

O enum4linux é um **wrapper** escrito em Perl que combina as seguintes ferramentas em sequência automática:

| Ferramenta Base | O Que Faz |
|----------------|-----------|
| `smbclient` | Lista shares, testa null session |
| `rpcclient` | Enumera usuários, grupos, políticas via RPC |
| `nmblookup` | Resolve nomes NetBIOS |
| `net` | Informações de domínio e membros de grupos |

Em vez de rodar cada uma manualmente, o enum4linux executa tudo e apresenta um relatório organizado.

---

## 🔢 Flags e Opções

```bash
enum4linux [opções] IP
```

| Flag | Função | Equivalente Manual |
|------|--------|------------------|
| `-a` | **All** — executa todas as verificações | Todas as abaixo |
| `-U` | Enumerar usuários | `rpcclient → enumdomusers` |
| `-G` | Enumerar grupos | `rpcclient → enumdomgroups` |
| `-S` | Enumerar shares | `smbclient -L` |
| `-P` | Política de senha | `rpcclient → getdompwinfo` |
| `-o` | Informações do SO | `smbclient` OS info |
| `-i` | Informações de impressoras | `rpcclient → enumprinters` |
| `-r` | RID cycling (descoberta de usuários por RID) | Loop com `rpcclient → queryuser` |
| `-R RANGE` | Range de RIDs para cycling | `rpcclient → queryuser 0x500` |
| `-w WORKGROUP` | Definir workgroup manualmente | — |
| `-u USUARIO` | Usuário para autenticação | `-U` no smbclient |
| `-p SENHA` | Senha para autenticação | — |
| `-n` | Fazer nmblookup | `nmblookup -A IP` |
| `-v` | Verbose | — |

---

## 🔧 Comandos

### Enumeração Completa (mais usado)
```bash
enum4linux -a IP
enum4linux -a target.ine.local
```

### Com Credencial
```bash
enum4linux -a -u administrator -p senha IP
enum4linux -a -u admin -p password IP
```

### Verificações Individuais
```bash
# Só usuários
enum4linux -U IP

# Só shares
enum4linux -S IP

# Só grupos
enum4linux -G IP

# Só política de senha
enum4linux -P IP

# RID cycling com range customizado
enum4linux -r -R 500-600 IP

# Usuários + grupos + shares (sem tudo)
enum4linux -U -G -S IP
```

---

## 📋 Interpretando o Output

### Seção: Target Information
```
Target ........... target.ine.local
RID Range ........ 500-550,1000-1050
Username ......... ''          ← vazio = null session
Password ......... ''          ← vazio = null session
```

---

### Seção: Workgroup/Domain
```
[+] Got domain/workgroup name: WORKGROUP
```

| Resultado | Interpretação |
|-----------|--------------|
| `WORKGROUP` | Máquina standalone — sem Active Directory |
| Nome real (ex: `EMPRESA`) | Parte de domínio AD |

---

### Seção: Session Check
```
[+] Server allows sessions using username '', password ''
```
✅ Null session habilitada — toda enumeração seguinte funciona sem credencial.

```
[-] Server doesn't allow session using username '', password ''
```
❌ Null session bloqueada — precisa de credencial válida (`-u` e `-p`).

---

### Seção: Domain SID
```
Domain SID: S-1-5-21-XXXXXXXXX-XXXXXXXXX-XXXXXXXXX
```
SID real → máquina é **membro de domínio AD**.

```
Domain SID: (NULL SID)
```
NULL SID → máquina **standalone** (workgroup), sem domínio.

---

### Seção: Users (RID Cycling)
```
[+] Got SID for domain from server
user:[administrator] rid:[0x1f4]   ← RID 500 em hex
user:[guest] rid:[0x1f5]           ← RID 501
user:[john] rid:[0x3ee]            ← RID 1006
user:[admin] rid:[0x3f2]           ← RID 1010
```

**Como o RID Cycling funciona:**
```
Para cada RID no range (500-550, 1000-1050):
    rpcclient → queryuser 0xRID
    Se retornar nome → usuário existe
    Se retornar erro → RID não usado
```

RIDs fixos do Windows:
| RID (decimal) | RID (hex) | Conta |
|--------------|-----------|-------|
| 500 | 0x1f4 | Administrator |
| 501 | 0x1f5 | Guest |
| 502 | 0x1f6 | krbtgt (só em DCs) |
| 1000+ | 0x3e8+ | Usuários criados manualmente |

---

### Seção: Password Policy
```
[+] Password Info for Domain: TARGET
    Minimum password length: 5
    Password history length: None
    Maximum password age: Not Set
    Password Lockout Threshold: None    ← sem bloqueio
    Lockout duration: Not Set
    Lockout observation window: Not Set
    Password Complexity Flags: 000000   ← sem complexidade
```

**Red flags imediatos:**

| Campo | Valor Perigoso | Implicação |
|-------|---------------|-----------|
| `Lockout Threshold` | `None` / `0` | Brute force irrestrito |
| `Minimum length` | `< 8` | Senhas fracas prováveis |
| `Complexity Flags` | `000000` | Sem requisito de complexidade |
| `Maximum age` | `Not Set` | Senha nunca expira |

---

### Seção: Shares
```
Sharename    Type    Comment
---------    ----    -------
public       Disk
IPC$         IPC     IPC Service
admin        Disk
```

**Combinado com brute force de acesso:**
```
pubfiles EXISTS, Allows access using username: '', password: ''
```
Share acessível via null session — acesso direto sem credencial.

---

### Seção: Groups
```
[+] Getting local groups:
group:[Administrators] rid:[0x220]
group:[Users] rid:[0x221]
group:[Guests] rid:[0x222]

[+] Getting local group memberships:
Group 'Administrators' (RID: 544):
    administrator (SID: S-1-5-21-...-500)
    john (SID: S-1-5-21-...-1006)    ← usuário não-padrão em Admins!
```

Encontrar usuários não-padrão no grupo Administrators é um finding crítico.

---

## 🔁 Workflow Completo com enum4linux

```bash
# 1. Enumeração inicial — null session
enum4linux -a IP

# 2. Analisar output:
#    → null session habilitada?
#    → usuários identificados?
#    → lockout desabilitado?
#    → shares acessíveis?

# 3. Se null session OK → montar lista de usuários
echo -e "john\nadmin\nguest" > users.txt

# 4. Brute force com usuários encontrados
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt smb://IP
# ou
use auxiliary/scanner/smb/smb_login
set USER_FILE users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt

# 5. Com credencial válida → reenumerar autenticado
enum4linux -a -u usuario -p senha IP

# 6. Acessar shares
smbclient //IP/share -U usuario
```

---

## ⚡ enum4linux vs Alternativas

| Ferramenta | Vantagem | Quando Usar |
|-----------|---------|-------------|
| `enum4linux` | Tudo automatizado, output organizado | Recon inicial rápido |
| `enum4linux-ng` | Versão moderna em Python, mais rápida | Ambientes modernos |
| `rpcclient` manual | Controle total de cada query | Quando precisa de query específica |
| `crackmapexec` | Mais completo, suporte a AD | Ambientes corporativos |
| `ldapdomaindump` | Dump completo de AD via LDAP | Domínios Active Directory |

### enum4linux-ng (versão moderna)
```bash
# Instalar
pip3 install enum4linux-ng

# Usar
enum4linux-ng -A IP
enum4linux-ng -A -u usuario -p senha IP
```

---

## ⚠️ Limitações

- **Ambientes modernos** com SMB signing obrigatório podem bloquear algumas consultas
- **Domínios AD modernos** bloqueiam null session por padrão
- **Windows 10/2019+** tem null session desabilitada por padrão
- Funciona melhor contra **Samba/Linux** e **Windows legado**
- Gera logs no servidor alvo — não é silencioso

---

## 🧠 Modelo Mental

```
enum4linux -a IP
        ↓
Null session?
    SIM → mapa completo sem credencial:
          usuários, grupos, shares, política
    NÃO → precisa de credencial
          → brute force primeiro
          → enum4linux -a -u user -p pass IP
        ↓
Usuários encontrados → alimentar brute force
Lockout = 0 → brute force agressivo seguro
Shares acessíveis → smbclient //IP/share
```

---

## 📌 Relacionados

- [[SMB — Server Message Block]]
- [[Samba Recon Completo]]
- [[SMB Enumeration com enum4linux]]
- [[SMB Brute Force e Acesso a Shares]]
- [[Hydra — Brute Force de Credenciais]]
- [[Cheatsheet — Portas Importantes]]

#ferramenta/enum4linux #protocolo/smb #recon/ativo #windows
