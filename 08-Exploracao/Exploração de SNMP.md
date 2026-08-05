## Exploração de SNMP

O **SNMP** (Simple Network Management Protocol) roda normalmente na porta **UDP 161** e é usado para monitoramento de dispositivos. Configurações fracas (ex: community string `public`) permitem enumeração sensível.

---

# Fase 1 — Descoberta

### 1. Scan inicial

nmap -sU -p161 target

Ou mais detalhado:

nmap -sU -sV -p161 target

Se aparecer:

161/udp open snmp

→ SNMP ativo.

---

# Fase 2 — Enumeração com community strings

Strings comuns:

- public
- private
- manager
- admin

### 2. Teste rápido com snmpwalk

snmpwalk -v1 -c public target

ou:

snmpwalk -v2c -c public target

Se retornar dados → acesso permitido.

---

# Fase 3 — Brute force de community strings

### 3. Com onesixtyone

onesixtyone target -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt

Resultado exemplo:

target [public] Linux server

---

# Fase 4 — Dump completo SNMP

snmpwalk -v2c -c public target

Informações comuns:

- usuários
- processos
- interfaces
- rotas
- software instalado

---

# Fase 5 — Enumeração útil

### 1. Informações do sistema

snmpwalk -v2c -c public target 1.3.6.1.2.1.1

### 2. Processos rodando

snmpwalk -v2c -c public target 1.3.6.1.2.1.25.4.2.1.2

### 3. Usuários

snmpwalk -v2c -c public target 1.3.6.1.4.1.77.1.2.25

### 4. Interfaces de rede

snmpwalk -v2c -c public target 1.3.6.1.2.1.2.2.1.2

---

# Fase 6 — Ferramenta snmp-check

Mais organizado:

snmp-check target -c public

---

# Fase 7 — Exploração indireta

SNMP normalmente não dá shell direto, mas fornece:

- usuários
- caminhos
- versões vulneráveis
- shares
- serviços

Depois disso você:

- brute force SSH
- brute force SMB
- explorar serviços descobertos

---

# Exemplo fluxo completo

nmap -sU -p161 target  
snmpwalk -v2c -c public target  
snmp-check target -c public  
snmpwalk -v2c -c public target 1.3.6.1.2.1.25.4.2.1.2

Descobre:

apache  
mysql  
backup.sh

→ vetor de ataque.

---

# Brute force SNMP com Hydra

hydra -P wordlist.txt target snmp

---

# Pivot comum com SNMP

1. Descobrir usuários
2. Descobrir senha em script
3. SSH login
4. Escalada local

---

# OIDs importantes

|Informação|OID|
|---|---|
|System info|1.3.6.1.2.1.1|
|Processos|1.3.6.1.2.1.25.4.2|
|Usuários|1.3.6.1.4.1.77.1.2.25|
|Interfaces|1.3.6.1.2.1.2.2|
|Rotas|1.3.6.1.2.1.4.21|

---

# Fluxo resumido

Scan UDP 161  
     ↓  
Test community string  
     ↓  
Brute force community  
     ↓  
Dump SNMP  
     ↓  
Coletar usuários / serviços  
     ↓  
Exploração indireta

---

# Classificação

|Tipo|Categoria|
|---|---|
|SNMP enumeração|Network-based|
|Uso dos dados|Post-exploitation|
|Escalada posterior|Host-based|

SNMP é **principalmente enumeração**, não exploit direto.