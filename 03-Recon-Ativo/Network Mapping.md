# 🗺️ Network Mapping

> Processo de descobrir e documentar a **estrutura completa de uma rede**.
> Quais hosts existem, portas abertas, serviços rodando e como tudo se conecta.
> É a espinha dorsal da fase de reconhecimento ofensivo.

---

## 🎯 As 4 Perguntas Fundamentais

```
1. Quais máquinas existem?        → Host Discovery
2. Quais portas estão abertas?    → Port Scanning
3. Quais serviços estão rodando?  → Service Enumeration
4. Como os sistemas se conectam?  → Topologia
```

Sem responder isso, qualquer exploração é adivinhação.

---

## 🔁 Etapas do Network Mapping

### 1️⃣ Host Discovery
```bash
nmap -sn 192.168.1.0/24
```
→ Lista de máquinas vivas.

---

### 2️⃣ Port Scanning
```bash
sudo nmap -sS IP
sudo nmap -p- IP   # todas as 65535 portas
```
→ Portas abertas, fechadas, filtradas.

---

### 3️⃣ Service Enumeration
```bash
sudo nmap -sV IP
```
→ O que está rodando em cada porta.
```
80/tcp  open  http    Apache 2.4.49
22/tcp  open  ssh     OpenSSH 8.2
```

---

### 4️⃣ OS Detection
```bash
sudo nmap -O IP
```
→ Sistema operacional inferido.

---

### 5️⃣ Mapeamento de Topologia
```bash
traceroute IP
nmap --traceroute IP
```
→ Roteadores intermediários, estrutura da rede.

---

## 🛠️ Ferramentas

| Ferramenta | Uso |
|-----------|-----|
| [[Nmap — Visão Geral]] | Ferramenta central de tudo |
| `netdiscover` | Descoberta de hosts na LAN |
| `arp-scan` | ARP scan eficiente na LAN |
| `masscan` | Scan extremamente rápido |
| `wireshark` | Análise de tráfego |

---

## 📊 Tipos de Scan

| Tipo | Objetivo |
|------|---------|
| Horizontal | Muitos hosts, mesma porta |
| Vertical | Um host, muitas portas |
| Internal | Dentro da rede |
| External | A partir da internet |

---

## 🔁 Workflow Profissional

```bash
# 1. Descobrir hosts
nmap -sn 192.168.1.0/24

# 2. Scan completo de portas nos ativos
sudo nmap -p- -T4 IP

# 3. Versão dos serviços apenas nas portas abertas
sudo nmap -sV -O -p 22,80,443 IP

# 4. Scripts NSE para detalhar
sudo nmap -sV --script=default,vuln IP
```

---

## 🔥 Conceito Avançado

Mapear não é só portas. É entender:
- Segmentação de rede e VLANs
- Onde estão os firewalls
- IDS/IPS na rota
- NAT mascarando hosts internos

Uma rede pode parecer pequena externamente, mas ser enorme internamente.

---

## 🧠 Modelo Mental

> Network mapping não é "escanear tudo".
> É formular hipótese → coletar evidência → ajustar o mapa.
> É ciência aplicada.

---

## 📌 Relacionados

- [[Host Discovery]]
- [[Nmap — Port Scanning]]
- [[Nmap — Service & OS Detection]]
- [[Firewall — Conceito]]
- [[IDS & IPS]]

#recon/ativo #ferramenta/nmap #conceito
