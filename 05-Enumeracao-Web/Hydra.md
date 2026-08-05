# ⚡ Hydra — Guia Completo de Brute Force

> Ferramenta de brute force de credenciais online mais usada em pentest.
> Suporta dezenas de protocolos e permite paralelismo agressivo.

---

## 🧠 O Que é Hydra

O Hydra (THC-Hydra) é um **cracker de login online** — testa combinações de usuário e senha diretamente contra serviços ativos em tempo real. Diferente de hashcat/john (offline), o Hydra ataca o serviço enquanto ele está rodando.

**Criado por:** van Hauser / The Hacker's Choice (THC)
**Linguagem:** C — extremamente rápido e eficiente

---

## 🔑 Conceito Fundamental: L/l e P/p

A confusão mais comum com o Hydra:

| Flag | Tipo | Função |
|------|------|--------|
| `-l` | minúsculo | Um **único** usuário (valor direto) |
| `-L` | MAIÚSCULO | **Lista** de usuários (arquivo) |
| `-p` | minúsculo | Uma **única** senha (valor direto) |
| `-P` | MAIÚSCULO | **Lista** de senhas (arquivo) |

```bash
# Um usuário, lista de senhas
hydra -l admin -P passwords.txt ftp://IP

# Lista de usuários, uma senha
hydra -L users.txt -p password ftp://IP

# Lista de usuários, lista de senhas (matriz completa)
hydra -L users.txt -P passwords.txt ftp://IP

# Um usuário, uma senha (só para validar)
hydra -l admin -p password ftp://IP
```

---

## 🔢 Flags Principais

| Flag | Função |
|------|--------|
| `-l USER` | Usuário único |
| `-L FILE` | Arquivo de usuários |
| `-p PASS` | Senha única |
| `-P FILE` | Arquivo de senhas |
| `-C FILE` | Arquivo `usuario:senha` (um par por linha) |
| `-s PORTA` | Porta não-padrão |
| `-t N` | Threads paralelas (padrão: 16) |
| `-f` | Parar após **primeiro** sucesso por host |
| `-F` | Parar após **qualquer** sucesso |
| `-vV` | Verbose — mostrar cada tentativa |
| `-V` | Mostrar tentativas (menos verboso que -vV) |
| `-d` | Debug |
| `-o FILE` | Salvar credenciais encontradas em arquivo |
| `-e nsr` | Testes extras: `n`=sem senha, `s`=user=pass, `r`=reverso |
| `-w N` | Timeout por conexão em segundos (padrão: 30) |
| `-W N` | Tempo entre tentativas (para ser mais lento/furtivo) |
| `-u` | Loop por usuário primeiro (padrão: por senha) |
| `-I` | Ignorar arquivo de restore e começar do zero |
| `-R` | Retomar sessão anterior (restore) |
| `-x MIN:MAX:CHARSET` | Geração de senhas on-the-fly |

---

## 🌐 Protocolos Suportados

### Mais Usados em Pentest

```bash
ftp://IP:PORTA          # FTP
ssh://IP:PORTA          # SSH
smb://IP                # SMB/Samba
mysql://IP:PORTA        # MySQL
mssql://IP              # Microsoft SQL Server
rdp://IP                # Remote Desktop
vnc://IP                # VNC
telnet://IP             # Telnet
```

### Web

```bash
http-get://IP/path           # HTTP Basic Auth (GET)
http-post-form://IP/path     # Formulário HTML (POST)
https-get://IP/path          # HTTPS Basic Auth
https-post-form://IP/path    # Formulário HTTPS
```

### Outros

```bash
pop3://IP               # Email POP3
smtp://IP               # Email SMTP
imap://IP               # Email IMAP
ldap://IP               # LDAP
postgresql://IP         # PostgreSQL
oracle-listener://IP    # Oracle
sip://IP                # VoIP
redis://IP              # Redis
```

### Ver Todos os Módulos Disponíveis
```bash
hydra -h | grep "Supported services"
hydra -U ftp    # ajuda específica de um módulo
```

---

## 🔧 Comandos por Protocolo

### FTP
```bash
# Porta padrão
hydra -L users.txt -P passwords.txt ftp://IP

# Porta não-padrão
hydra -L users.txt -P passwords.txt ftp://IP:5554

# Usuário único
hydra -l admin -P passwords.txt -f ftp://IP
```

---

### SSH
```bash
hydra -L users.txt -P passwords.txt ssh://IP

# Reduzir threads (SSH tem limites de conexão)
hydra -L users.txt -P passwords.txt -t 4 ssh://IP

# Com verbose para ver tentativas
hydra -l root -P /usr/share/wordlists/rockyou.txt -vV ssh://IP
```

---

### SMB
```bash
hydra -L users.txt -P passwords.txt smb://IP

# Com domínio
hydra -L users.txt -P passwords.txt smb://IP -m DOMINIO
```

---

### MySQL
```bash
hydra -l root -P passwords.txt mysql://IP

# Porta não-padrão
hydra -l root -P passwords.txt mysql://IP:3307
```

---

### RDP
```bash
hydra -L users.txt -P passwords.txt rdp://IP -t 4
```

---

### HTTP Basic Auth
```bash
hydra -L users.txt -P passwords.txt http-get://IP/admin/

# HTTPS
hydra -L users.txt -P passwords.txt https-get://IP/secure/
```

---

### HTTP Form (Login Web)
```bash
# Estrutura: URL:CAMPOS_POST:MENSAGEM_DE_FALHA
hydra -L users.txt -P passwords.txt \
  http-post-form://IP/login.php:"username=^USER^&password=^PASS^:Invalid credentials"

# ^USER^ e ^PASS^ são os placeholders que Hydra substitui
# A última parte é o texto que aparece quando o login FALHA
```

**Como descobrir os parâmetros do formulário:**
```bash
# Inspecionar no browser (DevTools → Network → ver POST)
# ou
curl -v -X POST http://IP/login.php -d "username=test&password=test"
```

---

## ⚡ Combinações Práticas

### Descoberta Rápida (usuário e senha prováveis)
```bash
# Testar sem senha, user=pass, e reverso
hydra -l admin -e nsr ftp://IP
```

### Máxima Velocidade (lab/CTF)
```bash
hydra -L users.txt -P passwords.txt -t 64 -f ftp://IP
```

### Furtivo (evitar detecção/lockout)
```bash
# 1 thread, 3 segundos entre tentativas
hydra -L users.txt -P passwords.txt -t 1 -W 3 ssh://IP
```

### Salvar e Retomar
```bash
# Salvar progresso automaticamente (hydra.restore)
hydra -L users.txt -P passwords.txt ssh://IP

# Retomar onde parou
hydra -R
```

### Geração de Senhas On-the-Fly
```bash
# Senhas de 4-6 caracteres só com letras minúsculas
hydra -l admin -x 4:6:a ftp://IP

# Minúsculas + números, 6-8 chars
hydra -l admin -x 6:8:aA1 ftp://IP
# a = minúsculas, A = maiúsculas, 1 = números
```

---

## 📁 Wordlists Recomendadas

| Wordlist | Tamanho | Melhor Para |
|---------|---------|------------|
| `/usr/share/wordlists/rockyou.txt` | 14M senhas | Padrão de mercado |
| `/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt` | ~1000 senhas | Senhas Unix comuns, rápido |
| `/usr/share/metasploit-framework/data/wordlists/common_users.txt` | ~100 usuários | Usuários padrão de sistemas |
| `/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt` | 1000 senhas | Rápido e eficiente |
| `/usr/share/seclists/Usernames/top-usernames-shortlist.txt` | ~17 usuários | Reconhecimento inicial |

---

## ⚠️ Erros Comuns e Soluções

### "Invalid target definition"
```bash
# ❌ ERRADO — sem o protocolo
hydra -L users.txt -P pass.txt 192.168.1.1

# ✅ CORRETO
hydra -L users.txt -P pass.txt ftp://192.168.1.1
```

### "Too many connection errors"
```bash
# Causa: porta errada, serviço não responde, ou threads demais
# Solução: verificar porta e reduzir threads
hydra -L users.txt -P pass.txt -t 4 ftp://IP:PORTA_CORRETA
```

### SSH muito lento
```bash
# SSH tem proteção contra conexões paralelas
# Reduzir threads para 4
hydra -L users.txt -P pass.txt -t 4 ssh://IP
```

### Resultados inconsistentes em SMB
```bash
# SMB moderno pode bloquear Hydra
# Usar Metasploit como alternativa
use auxiliary/scanner/smb/smb_login
```

---

## 🔁 Workflow Completo

```bash
# 1. Identificar o serviço e porta
nmap -sV -p- IP

# 2. Ter os usuários (banner, enum4linux, OSINT)
echo -e "admin\nroot\nguest" > users.txt

# 3. Escolher wordlist adequada
# Rápida para lab: unix_passwords.txt
# Completa para tudo: rockyou.txt

# 4. Rodar com protocolo e porta correta
hydra -L users.txt -P passwords.txt -f -vV PROTOCOLO://IP:PORTA

# 5. Credencial encontrada → validar
ftp IP PORTA
ssh usuario@IP -p PORTA

# 6. Registrar no banco do Metasploit
creds add user:USUARIO password:SENHA host:IP
```

---

## ⚔️ Hydra vs Outras Ferramentas de Brute Force

| Ferramenta | Tipo | Protocolos | Velocidade | Uso Principal |
|-----------|------|-----------|-----------|--------------|
| **Hydra** | Online | 50+ | Alta | Geral — múltiplos protocolos |
| **Medusa** | Online | 30+ | Alta | Alternativa ao Hydra |
| **Ncrack** | Online | 6 | Média | SSH, RDP, FTP, SMB |
| **MSF smb_login** | Online | SMB | Média | SMB específico |
| **hashcat** | Offline | — | Muito Alta | Quebrar hashes |
| **john** | Offline | — | Alta | Quebrar hashes |

> 💡 **Online** = ataca o serviço em tempo real. **Offline** = quebra hashes sem comunicação com o alvo.

---

## 🧠 Modelo Mental

```
Tenho um serviço → Hydra é minha primeira opção online

Tenho usuários confirmados?
    SIM → -L users.txt (wordlist pequena e direcionada)
    NÃO → -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt

Conheço a senha provável?
    SIM → -p senha (testar direto)
    NÃO → -P passwords.txt (wordlist de senhas)

Há risco de lockout?
    SIM → -t 1 -W 3 (1 thread, 3s entre tentativas)
    NÃO → -t 16 -f (16 threads, parar no primeiro sucesso)

Porta não-padrão?
    SEMPRE → -s PORTA ou protocolo://IP:PORTA
```

---

## 📌 Relacionados

- [[enum4linux]]
- [[FTP — File Transfer Protocol]]
- [[SMB — Server Message Block]]
- [[FTP Porta Nao-Padrao e Hydra Brute Force]]
- [[FTP Enumeration com Metasploit]]
- [[Metasploit — Fundamentos e Arquitetura]]

#ferramenta/hydra #brute-force #exploração #credenciais
