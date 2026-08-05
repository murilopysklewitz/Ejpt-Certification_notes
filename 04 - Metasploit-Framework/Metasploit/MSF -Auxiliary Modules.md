Quando se fala em **Port Scanning dentro do Metasploit**, muita gente pensa apenas em exploits. Só que o **Metasploit Framework** também possui diversos módulos **auxiliary** voltados para **reconhecimento e enumeração**. Entre eles estão scanners de portas que funcionam como uma alternativa ao **Nmap**, mas integrados ao banco de dados do próprio Metasploit.

Essa integração muda o fluxo de trabalho. Em vez de escanear, salvar resultados e depois importar manualmente, o próprio Metasploit já armazena tudo automaticamente.

---

# Port Scanning com MSF Auxiliary Modules

Os scanners ficam na categoria:

auxiliary/scanner/portscan/

Alguns módulos importantes:

|Módulo|Função|
|---|---|
|`auxiliary/scanner/portscan/tcp`|TCP connect scan|
|`auxiliary/scanner/portscan/syn`|SYN scan (semi-open)|
|`auxiliary/scanner/portscan/ack`|ACK scan (descobrir regras de firewall)|
|`auxiliary/scanner/portscan/xmas`|Técnica stealth com flags TCP|

---

# 1. Usando um scanner TCP

Dentro do console do Metasploit:

use auxiliary/scanner/portscan/tcp

Ver opções do módulo:

show options

Configuração típica:

set RHOSTS 192.168.1.10  
set PORTS 1-1000  
set THREADS 20  
run

Explicando rapidamente:

|Parâmetro|Significado|
|---|---|
|RHOSTS|alvo|
|PORTS|range de portas|
|THREADS|número de conexões paralelas|

Esse módulo realiza um **TCP connect scan**, semelhante ao `nmap -sT`.

---

# 2. SYN Scan (mais stealth)

use auxiliary/scanner/portscan/syn

Configuração:

set RHOSTS 192.168.1.10  
set PORTS 1-1000  
run

Isso executa um **half-open scan**, equivalente ao:

nmap -sS

O scan envia **SYN → recebe SYN/ACK → encerra conexão**, sem completar o handshake.

Firewalls e logs de sistema detectam menos esse tipo de atividade.

---

# 3. ACK Scan (detecção de firewall)

use auxiliary/scanner/portscan/ack

Objetivo:

- identificar **filtragem de firewall**
    
- descobrir **se portas estão filtradas ou não**
    

Equivalente ao:

nmap -sA

O pacote enviado contém apenas a flag **ACK**, o que gera respostas diferentes dependendo da política do firewall.

---

# 4. Xmas Scan

use auxiliary/scanner/portscan/xmas

Essa técnica envia um pacote com várias flags ligadas:

FIN + PSH + URG

Parece uma árvore de natal de bits acesos — daí o nome **Xmas scan**.

Alguns sistemas respondem apenas se a porta estiver fechada, permitindo inferir estados.

---

# 5. Escaneando múltiplos alvos

Metasploit permite ranges:

set RHOSTS 192.168.1.0/24

ou

set RHOSTS 192.168.1.10-20

Depois:

run

---

# 6. Salvando resultados no banco do Metasploit

Se o **database estiver ativo**, portas encontradas são armazenadas automaticamente.

Ver hosts descobertos:

hosts

Ver portas abertas:

services

Isso permite que você use exploits diretamente contra serviços descobertos.

Exemplo:

services -p 445

Para identificar máquinas com SMB aberto.

---

# 7. Workflow realista em pentest

Fluxo comum usando o **Metasploit Framework**:

db_nmap -sS 192.168.1.0/24

ou

auxiliary/scanner/portscan/syn

Depois:

services

Depois buscar exploits:

search smb

E então:

use exploit/windows/smb/ms17_010_eternalblue

Tudo dentro do mesmo ambiente.

---

# Observação interessante

Pentesters experientes raramente usam o Metasploit como scanner principal.

Normalmente fazem:

Nmap → importação → exploração no Metasploit

Porque o **Nmap possui motores de detecção muito mais avançados**.

Mas o scanner interno do Metasploit é útil para:

- **quick scans**
    
- **pivoting dentro de redes comprometidas**
    
- **escaneamento através de sessões Meterpreter**
    

---

Um detalhe técnico fascinante: quando você compromete uma máquina e usa **pivoting com Meterpreter**, o Metasploit consegue **escanear a rede interna a partir da máquina comprometida**. Nesse cenário, os módulos `auxiliary/scanner` viram uma espécie de **sonda exploratória dentro da rede da vítima** — algo que ferramentas externas não conseguem fazer diretamente.

## 📌 Relacionados

- [[Metasploit — Fundamentos e Arquitetura]]
- [[Nmap — Output Formats]]
- [[SMB — Enumeração e Comprometimento]]
- [[Cheatsheet — Portas Importantes]]

#ferramenta/metasploit #exploração #database