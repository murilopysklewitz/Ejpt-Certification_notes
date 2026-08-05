# 🧪 Lab Report — SMTP Enumeration e Reconhecimento com Postfix

> **Plataforma:** INE
> **Tema central:** Reconhecimento de servidor SMTP — banner grabbing, enumeração manual de usuários via VRFY, capacidades do servidor, enumeração automatizada e envio de email falso
> **Alvo:** `demo.ine.local` / `openmailbox.xyz`

---

## 🎯 Objetivo

Mapear completamente um servidor SMTP Postfix — identificar versão e banner, enumerar usuários válidos manualmente e com ferramentas automatizadas, descobrir capacidades do servidor e demonstrar o envio de email com remetente falso (email spoofing).

---

## 🧠 O Que é SMTP e Por Que é um Alvo

SMTP (Simple Mail Transfer Protocol) é o protocolo responsável pelo **envio** de emails entre servidores. Opera na porta 25 (servidor-para-servidor) ou 587 (cliente-para-servidor autenticado).

**Por que SMTP é relevante em pentest:**

| Vetor | Técnica | Impacto |
|-------|---------|---------|
| **VRFY** | Verificar se usuário existe | Enumeração de usuários do sistema |
| **EXPN** | Expandir listas de email | Revelar membros de grupos |
| **Open Relay** | Enviar email por qualquer servidor | Spam, phishing com domínio legítimo |
| **Email Spoofing** | Falsificar remetente | Phishing/engenharia social convincente |
| **Banner** | Versão do software | CVEs específicos do servidor |
| **Auth Brute Force** | Força bruta de credenciais SMTP | Acesso a conta de email |

> 💡 SMTP mal configurado é um vetor clássico de **engenharia social** — você pode enviar emails que parecem vir de dentro da organização.

---

## 📋 Sumário de Etapas

| # | Objetivo | Ferramenta | Resultado |
|---|---------|-----------|-----------|
| 1 | Banner e versão | `nmap -sV --script banner` | Postfix, `openmailbox.xyz` |
| 2 | Hostname do servidor | `nc` (netcat) | `openmailbox.xyz` |
| 3 | Verificar usuário `admin` | `VRFY` via netcat | ✅ Existe |
| 4 | Verificar usuário `commander` | `VRFY` via netcat | ❌ Não existe |
| 5 | Capacidades do servidor | `HELO`/`EHLO` via telnet | Extensões suportadas |
| 6 | Enumeração em massa | `smtp-user-enum` | 8 usuários válidos |
| 7 | Enumeração via Metasploit | `smtp_enum` | 22 usuários válidos |
| 8 | Envio de email falso | `telnet` manual | Email enviado como root |
| 9 | Envio com ferramenta | `sendemail` | Email enviado via CLI |

---

## 🔬 Execução Passo a Passo

### Step 1 — Banner e Versão do Servidor SMTP

```bash
nmap -sV --script banner demo.ine.local
```

**Por que `--script banner`:** O script NSE `banner` conecta em cada porta aberta e captura a mensagem de boas-vindas do serviço sem autenticar. Para SMTP, isso revela o software (Postfix, Sendmail, Exim) e frequentemente informações do servidor.

**Resultado:**
```
25/tcp open  smtp  Postfix smtpd
| banner: 220 openmailbox.xyz ESMTP Postfix: Welcome to our mail server.
```

**O que extrair do banner:**

| Campo | Valor | Relevância |
|-------|-------|-----------|
| Código `220` | Servidor pronto | Confirmação que SMTP está ativo |
| `openmailbox.xyz` | Hostname real do servidor | Domínio para usar nas queries VRFY |
| `ESMTP` | Extended SMTP | Suporta extensões (AUTH, STARTTLS, etc) |
| `Postfix` | Software | Buscar CVEs do Postfix |

> 💡 O hostname revelado no banner (`openmailbox.xyz`) é essencial — o VRFY precisa do formato `usuario@dominio`. Sem o banner, você não saberia qual domínio usar.

---

### Step 2 — Conexão Manual e Hostname via Netcat

```bash
nc demo.ine.local 25
```

**O que é netcat neste contexto:** Netcat abre uma conexão TCP "crua" — você vê exatamente o que o servidor envia e pode digitar comandos SMTP manualmente. É a forma mais direta de entender o protocolo.

**Output ao conectar:**
```
220 openmailbox.xyz ESMTP Postfix: Welcome to our mail server.
```

O hostname `openmailbox.xyz` é confirmado diretamente pelo banner do servidor.

---

### Step 3 e 4 — Enumerar Usuários com VRFY (Manual)

```bash
nc demo.ine.local 25

# Verificar usuário que existe
VRFY admin@openmailbox.xyz

# Verificar usuário que não existe
VRFY commander@openmailbox.xyz
```

**O que é o comando VRFY:**

`VRFY` (Verify) é um comando SMTP criado originalmente para verificar se um endereço de email é válido antes de enviar. Em servidores mal configurados, funciona como um **oráculo de enumeração de usuários** — você pergunta se o usuário existe e o servidor responde.

**Interpretando as respostas:**

| Código | Resposta | Significado |
|--------|---------|------------|
| `252` | `Cannot VRFY user, but will accept message` | ✅ Usuário existe |
| `250` | `openmailbox.xyz` | ✅ Usuário existe (confirmação direta) |
| `550` | `User unknown` | ❌ Usuário não existe |
| `502` | `Command not implemented` | VRFY desabilitado |

**Resultado:**
```
VRFY admin@openmailbox.xyz
252 2.0.0 admin          ← existe

VRFY commander@openmailbox.xyz
550 5.1.1 commander      ← não existe
```

> ⚠️ VRFY habilitado é uma misconfiguration de segurança. O RFC 2821 recomenda desabilitá-lo ou limitar seu uso. A maioria dos servidores modernos retorna `252` para todos os usuários (não revela se existe ou não) — aqui está mal configurado.

---

### Step 5 — Descobrir Capacidades com HELO/EHLO

```bash
telnet demo.ine.local 25

HELO attacker.xyz
EHLO attacker.xyz
```

**Por que usar telnet aqui em vez de netcat:** Ambos funcionam para SMTP. Telnet tem suporte a escape sequences que podem ser úteis. Na prática, para SMTP simples, são intercambiáveis.

**Diferença entre HELO e EHLO:**

| Comando | Protocolo | Retorno |
|---------|-----------|---------|
| `HELO dominio` | SMTP básico | Só `250 OK` |
| `EHLO dominio` | ESMTP (Extended) | Lista de extensões suportadas |

**Por que `EHLO` é mais útil em pentest:**

O `EHLO` retorna todas as extensões que o servidor suporta:

```
250-openmailbox.xyz
250-PIPELINING
250-SIZE 10240000
250-VRFY                ← VRFY habilitado = enumeração possível
250-ETRN
250-STARTTLS            ← suporta criptografia TLS
250-AUTH PLAIN LOGIN    ← métodos de autenticação → brute force
250-AUTH=PLAIN LOGIN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
```

**O que procurar no output do EHLO:**

| Extensão | Relevância |
|----------|-----------|
| `VRFY` | Enumeração de usuários possível |
| `AUTH PLAIN LOGIN` | Autenticação básica → brute force viável |
| `STARTTLS` | Criptografia disponível (mas não obrigatória) |
| `SIZE` | Limite de tamanho dos emails |

---

### Step 6 — Enumeração em Massa com smtp-user-enum

```bash
smtp-user-enum -U /usr/share/commix/src/txt/usernames.txt -t demo.ine.local
```

**O que faz:** Automatiza o processo manual do Step 3 — testa cada usuário da wordlist via `VRFY` (ou `EXPN`/`RCPT TO`) e reporta quais existem.

**Flags do smtp-user-enum:**

| Flag | Valor | Função |
|------|-------|--------|
| `-U` | wordlist | Arquivo com lista de usuários |
| `-t` | alvo | IP ou hostname |
| `-m` | método | VRFY (padrão), EXPN ou RCPT |
| `-p` | porta | Padrão 25 |
| `-v` | — | Verbose |

**Métodos de enumeração:**
```bash
# VRFY — direto, mais ruidoso
smtp-user-enum -M VRFY -U users.txt -t IP

# RCPT TO — menos bloqueado por filtros
smtp-user-enum -M RCPT -U users.txt -t IP -D dominio.com

# EXPN — expande listas de email
smtp-user-enum -M EXPN -U users.txt -t IP
```

**Resultado:** 8 usuários válidos encontrados na wordlist do commix.

---

### Step 7 — Enumeração via Metasploit

```bash
msfconsole -q

use auxiliary/scanner/smtp/smtp_enum
set RHOSTS demo.ine.local
exploit
```

**Por que usar Metasploit além do smtp-user-enum:**
- Wordlist padrão diferente (`unix_users.txt`) — mais abrangente para sistemas Unix
- Resultado salvo automaticamente no banco de dados (`hosts`, `notes`)
- Pode ser encadeado com outros módulos

**Wordlist padrão:** `/usr/share/metasploit-framework/data/wordlists/unix_users.txt`

**Resultado:** 22 usuários válidos — mais do que com a wordlist anterior, porque `unix_users.txt` é mais completo para sistemas Linux/Unix.

> 💡 A diferença de resultado (8 vs 22) mostra que a **wordlist importa tanto quanto a ferramenta**. Use múltiplas wordlists para cobertura máxima.

---

### Step 8 — Enviar Email Falso via Telnet (Manual)

```bash
telnet demo.ine.local 25

HELO attacker.xyz
MAIL FROM: admin@attacker.xyz
RCPT TO: root@openmailbox.xyz
DATA
Subject: Hi Root
Hello,
This is a fake mail sent using telnet command.
From,
Admin
.
```

**O que cada comando faz:**

| Comando SMTP | Função |
|-------------|--------|
| `HELO`/`EHLO` | Apresentação do cliente ao servidor |
| `MAIL FROM:` | Define o remetente (**pode ser falso**) |
| `RCPT TO:` | Define o destinatário |
| `DATA` | Inicia o corpo do email |
| `.` (linha só com ponto) | Termina o corpo e envia o email |

**Por que isso é perigoso — Open Relay:**

Um servidor SMTP que aceita emails de qualquer remetente para qualquer destinatário é chamado de **Open Relay**. Permite:
- Enviar emails como qualquer pessoa (`admin@empresa.com`, `ceo@empresa.com`)
- Usar o servidor como intermediário para spam
- Phishing com domínio legítimo da empresa

```
MAIL FROM: ceo@empresa.com    ← completamente falso
RCPT TO: funcionario@empresa.com
DATA
Por favor, transfira R$ 50.000 para...
.
```

> ⚠️ Isso não quebra criptografia. O campo `From:` no email é texto livre — não é verificado por padrão. SPF, DKIM e DMARC existem justamente para mitigar isso, mas muitos servidores ainda não os configuram corretamente.

---

### Step 9 — Enviar Email Falso via sendemail

```bash
sendemail -f admin@attacker.xyz \
          -t root@openmailbox.xyz \
          -s demo.ine.local \
          -u "Fakemail" \
          -m "Hi root, a fake from admin" \
          -o tls=no
```

**Flags do sendemail:**

| Flag | Valor | Função |
|------|-------|--------|
| `-f` | remetente | From — endereço falso |
| `-t` | destinatário | To |
| `-s` | servidor SMTP | Qual servidor usar para enviar |
| `-u` | assunto | Subject |
| `-m` | mensagem | Corpo do email |
| `-o tls=no` | sem TLS | Não usar criptografia (para servidores sem TLS) |

**Diferença telnet vs sendemail:** Telnet é manual e didático — você entende o protocolo SMTP passo a passo. `sendemail` é uma ferramenta que automatiza o processo e é mais prática para uso real em pentest.

---

## 📊 Resultado Final

| Informação | Valor |
|-----------|-------|
| Servidor | Postfix |
| Banner | `openmailbox.xyz ESMTP Postfix` |
| Porta | 25/TCP |
| VRFY habilitado | ✅ Sim — misconfiguration |
| Open Relay | ✅ Sim — aceita qualquer remetente |
| Usuários encontrados | 22 (via unix_users.txt) |
| Email spoofing | ✅ Possível |

---

## 🧠 Conceitos Consolidados

### Os 3 Protocolos de Email

| Protocolo | Função | Porta |
|-----------|--------|-------|
| **SMTP** | Envio de email | 25 (servidor), 587 (cliente auth) |
| **POP3** | Receber email (baixa e remove) | 110 / 995 (SSL) |
| **IMAP** | Receber email (sincroniza) | 143 / 993 (SSL) |

SMTP é o único relevante para **envio** e **enumeração de usuários**.

### VRFY vs EXPN vs RCPT TO

| Método | Comando | Detectabilidade | Eficácia |
|--------|---------|----------------|---------|
| VRFY | `VRFY usuario@dominio` | Alta (log direto) | Alta se habilitado |
| EXPN | `EXPN lista` | Alta | Revela membros de grupos |
| RCPT TO | `RCPT TO: usuario@dominio` | Média | Funciona mesmo com VRFY desabilitado |

> `RCPT TO` é o método mais evasivo — simula o início de um envio de email real e é menos provável de ser filtrado.

### SPF, DKIM e DMARC — Por Que Email Spoofing Ainda Funciona

| Proteção | O Que Faz | Quando Falha |
|---------|----------|-------------|
| **SPF** | Lista quais IPs podem enviar pelo domínio | Não configurado ou muito permissivo |
| **DKIM** | Assinatura criptográfica no email | Não configurado |
| **DMARC** | Política de como tratar falhas SPF/DKIM | `p=none` (só monitora, não rejeita) |

Quando nenhum desses está configurado, qualquer servidor pode enviar email fingindo ser qualquer domínio.

---

## ⚠️ Red Flags em Enumeração SMTP

| Achado | Risco | Recomendação |
|--------|-------|-------------|
| VRFY habilitado | 🟡 Médio | Desabilitar no `main.cf`: `disable_vrfy_command = yes` |
| Open Relay | 🔴 Crítico | Configurar `mynetworks` corretamente |
| Sem SPF/DKIM/DMARC | 🔴 Alto | Configurar registros DNS de proteção |
| Banner com versão | 🟡 Baixo | Remover versão do banner: `smtpd_banner = $myhostname ESMTP` |
| AUTH PLAIN sem TLS | 🔴 Alto | Credenciais em texto claro na rede |

---

## 🔁 Próximos Passos Lógicos

```
22 usuários enumerados via smtp_enum
        ↓
Lista de usuários → SSH brute force
use auxiliary/scanner/ssh/ssh_login
set USER_FILE usuarios_smtp.txt
        ↓
VRFY confirmou admin existe
        ↓
Testar admin em outros serviços (FTP, SMB, MySQL)
        ↓
Open Relay confirmado
        ↓
Phishing interno via email com remetente legítimo
sendemail -f ceo@openmailbox.xyz -t funcionario@openmailbox.xyz ...
```

---

## 📌 Relacionados

- [[FTP Enumeration com Metasploit]]
- [[SSH Enumeration e Brute Force com Metasploit]]
- [[SMB Brute Force e Acesso a Shares]]
- [[Nmap — NSE]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Cheatsheet — Portas Importantes]]

#lab #recon/ativo #protocolo/smtp #ferramenta/metasploit #email #enumeracao
