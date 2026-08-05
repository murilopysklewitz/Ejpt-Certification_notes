# 📡 Host Discovery

> Identificar quais dispositivos estão **ativos** em uma rede.
> É a fase inicial do network mapping — você precisa saber **quem está vivo** antes de escanear portas.

---

## 🧠 Conceito Central

Host discovery funciona enviando **probes** (pequenos testes) e analisando respostas.

```
Resposta recebida  →  host está ativo ✅
Sem resposta       →  pode estar inativo OU protegido por firewall
```

> ⚠️ **Silêncio ≠ inexistência.** Um host pode ignorar ICMP e ainda estar ativo.

---

## 🔧 Técnicas Principais

### 1️⃣ ICMP Ping Sweep (Camada 3)
```bash
nmap -sn 192.168.1.0/24
ping 192.168.1.1
```
**Limitação:** firewalls frequentemente bloqueiam ICMP.

---

### 2️⃣ ARP Scan (Camada 2) — rede local
```bash
nmap -sn 192.168.1.0/24   # usa ARP automaticamente em LAN
arp-scan 192.168.1.0/24
```
**Por que é confiável:** ARP é essencial para comunicação interna. Bloqueá-lo quebraria a rede. Praticamente impossível de filtrar dentro da LAN.

---

### 3️⃣ TCP Ping — quando ICMP está bloqueado
```bash
# SYN ping nas portas 80 e 443
nmap -sn -PS80,443 192.168.1.0/24

# ACK ping
nmap -sn -PA80 192.168.1.0/24
```
Se receber SYN-ACK ou RST → host está vivo.

---

### 4️⃣ UDP Ping
```bash
nmap -sn -PU53 192.168.1.0/24
```
Menos confiável, mas útil para serviços específicos como DNS.

---

## 📊 Comparação das Técnicas

| Técnica | Camada OSI | Melhor Para | Limitação |
|---------|-----------|------------|-----------|
| ICMP | L3 | Internet | Bloqueado por firewalls |
| ARP | L2 | Rede local | Só funciona na LAN |
| TCP SYN | L4 | Quando ICMP bloqueado | Mais visível |
| UDP | L4 | Serviços UDP específicos | Pouco confiável |

---

## 🔧 Comandos Práticos

```bash
# Rede local (mais comum em labs)
nmap -sn 192.168.1.0/24

# Com verbose (mais detalhes)
nmap -sn -v 192.168.1.0/24

# Se ICMP bloqueado
nmap -sn -PS80,443 192.168.1.0/24

# Forçar scan mesmo sem resposta ao ping
nmap -Pn 192.168.1.10

# Ferramenta alternativa
arp-scan 192.168.1.0/24
netdiscover -i eth0 -r 192.168.1.0/24
```

---

## ⚠️ Problemas Comuns

**Firewall bloqueando ICMP** → parece que não há hosts → use TCP ou ARP.

**NAT** → pode mascarar dispositivos internos.

**IDS/IPS** → ping sweep agressivo pode ser detectado.

**Silent hosts** → host pode ignorar ICMP mas estar ativo com portas abertas.

---

## 🔁 Fluxo Real em Pentest

```
1. Identificar faixa de rede
2. Host discovery (quem está vivo)
3. Confirmar gateway
4. Mapear padrão de IPs ativos
5. Só depois → port scanning nos ativos
```

---

## 🧠 Modelo Mental

> Host Discovery não é "pingar tudo".
> É formular uma hipótese → escolher o probe certo → interpretar o silêncio corretamente.

---

## 📌 Relacionados

- [[Network Mapping]]
- [[Nmap — Host Discovery]]
- [[Network Layer — IP & ICMP]]
- [[Firewall — Conceito]]

#recon/ativo #ferramenta/nmap #protocolo/icmp
