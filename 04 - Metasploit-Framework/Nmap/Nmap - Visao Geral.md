# 🔬 Nmap — Visão Geral

> **Network Mapper** — a ferramenta mais usada em pentest para reconhecimento de redes.
> Vai muito além de port scanning: é uma plataforma completa de recon.

---

## 🧠 O Que o Nmap Faz

```
Host Discovery        →  Quem está vivo?
Port Scanning         →  O que está aberto?
Service Detection     →  O que está rodando?
OS Detection          →  Qual sistema?
NSE Scripts           →  O que posso descobrir/explorar?
Evasão de Firewall    →  Como passar despercebido?
```

---

## 🗂️ Notas Detalhadas

| Tópico | Nota |
|--------|------|
| Descoberta de hosts | [[Nmap — Host Discovery]] |
| Port scanning | [[Nmap — Port Scanning]] |
| Service e OS detection | [[Nmap — Service & OS Detection]] |
| NSE (scripts) | [[Nmap — NSE]] |
| Evasão e detecção de firewall | [[Nmap — Evasão & Firewall]] |

---

## ⚡ Comandos Mais Usados

```bash
# Host discovery (sem port scan)
nmap -sn 192.168.1.0/24

# SYN scan (o padrão em pentest)
sudo nmap -sS IP

# Scan completo: versão + OS + scripts
sudo nmap -T4 -sS -sV -O --osscan-guess -p- IP

# Scan agressivo (lab)
sudo nmap -A IP

# Só as portas mais comuns, rápido
nmap -F IP
```

---

## 📊 Estados das Portas

| Estado | Significado |
|--------|-------------|
| `open` | Porta aceita conexão — serviço ativo |
| `closed` | Porta responde mas não tem serviço |
| `filtered` | Firewall bloqueando — sem resposta |
| `open\|filtered` | Nmap não consegue determinar |

---

## 🔁 Workflow Ideal

```bash
# 1. Descobrir hosts ativos
nmap -sn 192.168.1.0/24

# 2. Scan completo de portas
sudo nmap -p- -T4 IP

# 3. Versão + OS só nas portas abertas (eficiente)
sudo nmap -sV -O -p 22,80,443 IP

# 4. Scripts NSE nos serviços encontrados
sudo nmap --script=vuln -p 22,80,443 IP
```

---

## ⚠️ Importante

- `-sS` precisa de `sudo` (raw sockets)
- `-A` é barulhento — só em labs
- `-p-` demora mas vale para encontrar serviços fora das 1000 portas padrão
- Serviços interessantes frequentemente estão em portas altas

---

## 📌 Relacionados

- [[Cheatsheet — Nmap Flags]]
- [[Host Discovery]]
- [[Network Mapping]]
- [[IDS & IPS]]

#ferramenta/nmap #recon/ativo
