# 📡 Nmap — Host Discovery

> Descobrir quais hosts estão ativos antes de fazer port scanning.
> O Nmap usa diferentes técnicas dependendo do ambiente.

---

## 🧠 Comportamento do Nmap

**Em rede local (LAN):** usa ARP automaticamente — extremamente confiável.
**Em rede remota:** usa ICMP + TCP por padrão.

> `-sn` = **Ping scan** — descobre hosts sem escanear portas.

---

## 🔧 Comandos

```bash
# Descoberta simples (mais comum)
nmap -sn 192.168.1.0/24

# Com verbose
nmap -sn -v 192.168.1.0/24

# TCP SYN ping (quando ICMP bloqueado)
nmap -sn -PS80,443 192.168.1.0/24

# TCP ACK ping (tenta passar firewall)
nmap -sn -PA80 192.168.1.0/24

# UDP ping
nmap -sn -PU53 192.168.1.0/24

# Ignorar discovery (assume host ativo)
nmap -Pn 192.168.1.10

# Host específico
nmap -sn 192.168.1.10
```

---

## 🔢 Notação de Rede

```bash
# Subnet /24 (254 hosts)
nmap -sn 192.168.1.0/24

# Intervalo específico
nmap -sn 192.168.1.1-50

# Múltiplos IPs
nmap -sn 192.168.1.1 192.168.1.5 192.168.1.10

# Arquivo com IPs
nmap -sn -iL hosts.txt
```

---

## 📊 -sn vs -Pn

| Flag | Comportamento |
|------|--------------|
| `-sn` | Faz discovery (ICMP/ARP), não escaneia portas |
| `-Pn` | Pula o discovery, assume host ativo, vai direto para port scan |

**Quando usar `-Pn`:**
- Host não responde a ping mas você sabe que está ativo
- Ambiente corporativo com ICMP bloqueado
- CTFs onde você sabe o IP alvo

---

## ⚙️ Como o Nmap Decide o Probe

```
Rede local (mesma subnet)?
    SIM → ARP (sem opção de desabilitar — é automático)
    NÃO → ICMP Echo + TCP SYN 443 + TCP ACK 80 + ICMP Timestamp
```

---

## 🔁 Workflow Padrão

```bash
# 1. Descobrir quem está vivo
nmap -sn 192.168.56.0/24

# 2. Anotar os IPs com "Host is up"
# 3. Escanear portas só nos ativos
sudo nmap -p- -T4 192.168.56.105
```

---

## 📌 Relacionados

- [[Host Discovery]]
- [[Nmap — Port Scanning]]
- [[Nmap — Visão Geral]]
- [[Network Layer — IP & ICMP]]

#ferramenta/nmap #recon/ativo
