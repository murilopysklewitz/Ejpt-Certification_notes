## Enumeração HTTP / Apache com Metasploit

Objetivo: identificar diretórios, métodos HTTP inseguros, autenticação e possíveis vetores de exploração.

Ferramenta principal: Metasploit Framework

---

# Fase 1 — Verificar conectividade

ping -c 5 victim-1

Se responder → host acessível.

---

# Fase 2 — Abrir Metasploit

msfconsole -q

---

# Fase 3 — Identificar versão do servidor

Module: `http_version`

use auxiliary/scanner/http/http_version  
set RHOSTS victim-1  
run

Retorna:

- versão Apache
- linguagem backend
- headers

---

# Fase 4 — robots.txt

Module: `robots_txt`

use auxiliary/scanner/http/robots_txt  
set RHOSTS victim-1  
run

Pode revelar:

- diretórios ocultos
- áreas administrativas

---

# Fase 5 — HTTP headers

use auxiliary/scanner/http/http_header  
set RHOSTS victim-1  
run

Testando diretório específico:

set TARGETURI /secure  
run

Procura:

- autenticação
- cookies
- redirects

---

# Fase 6 — Brute force de diretórios

Module: `brute_dirs`

use auxiliary/scanner/http/brute_dirs  
set RHOSTS victim-1  
run

---

# Fase 7 — Directory scanner com wordlist

use auxiliary/scanner/http/dir_scanner  
set RHOSTS victim-1  
set DICTIONARY /usr/share/metasploit-framework/data/wordlists/directory.txt  
run

Detecta:

- /admin
- /backup
- /uploads
- /data

---

# Fase 8 — Directory listing

use auxiliary/scanner/http/dir_listing  
set RHOSTS victim-1  
set PATH /data  
run

Verifica se o servidor permite listar arquivos.

---

# Fase 9 — Enumeração de arquivos

use auxiliary/scanner/http/files_dir  
set RHOSTS victim-1  
set VERBOSE false  
run

Procura arquivos comuns:

- backup.zip
- config.php
- db.sql

---

# Fase 10 — Exploração HTTP PUT

Module: `http_put`

Escrever arquivo no servidor:

use auxiliary/scanner/http/http_put  
set RHOSTS victim-1  
set PATH /data  
set FILENAME test.txt  
set FILEDATA "Welcome To AttackDefense"  
run

Download para verificar:

wget http://victim-1:80/data/test.txt  
cat test.txt

Se baixar → servidor vulnerável a upload.

---

# Deletar arquivo

set ACTION DELETE  
run

Testar novamente:

wget http://victim-1:80/data/test.txt

Erro 404 esperado.

---

# Fase 11 — Brute force login HTTP

use auxiliary/scanner/http/http_login  
set RHOSTS victim-1  
set AUTH_URI /secure/  
set VERBOSE false  
run

Testa:

- basic auth
- default credentials

---

# Fase 12 — Enumeração userdir Apache

use auxiliary/scanner/http/apache_userdir_enum  
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt  
set RHOSTS victim-1  
set VERBOSE false  
run

Descobre:

/~admin  
/~john  
/~test

---

# Fluxo completo

Ping  
 ↓  
http_version  
 ↓  
robots.txt  
 ↓  
headers  
 ↓  
dir brute  
 ↓  
dir scanner  
 ↓  
file listing  
 ↓  
HTTP PUT  
 ↓  
login brute  
 ↓  
userdir enum

---

# Possíveis Exploits após enumeração

|Descoberta|Exploit possível|
|---|---|
|HTTP PUT|upload webshell|
|Directory listing|download config|
|Backup file|credenciais|
|Userdir|acesso usuário|
|Login page|brute force|

---

# Classificação

|Técnica|Tipo|
|---|---|
|HTTP enum|Network-based|
|Directory brute|Network-based|
|HTTP PUT|Exploit|
|Login brute|Credential attack|

---

# Vetor mais crítico deste lab

HTTP PUT habilitado → permite:

- upload webshell
- execução remota
- takeover do servidor