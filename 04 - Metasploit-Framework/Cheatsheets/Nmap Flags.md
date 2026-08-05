# ⚡ Cheatsheet — Nmap Flags

---

## 🔍 Tipos de Scan

| Flag | Scan | Precisa sudo? | Uso |
|------|------|--------------|-----|
| `-sS` | SYN scan (stealth) | ✅ | Pentest padrão |
| `-sT` | TCP Connect | ❌ | Sem privilégios |
| `-sU` | UDP | ✅ | Serviços UDP |
| `-sA` | ACK (detecta firewall) | ✅ | Mapeamento de regras |
| `-sN` | NULL scan | ✅ | Evasão |
| `-sF` | FIN scan | ✅ | Evasão |
| `-sX` | Xmas scan | ✅ | Evasão |

---

## 🔢 Seleção de Portas

| Flag | Função |
|------|--------|
| `-p 22` | Porta específica |
| `-p 22,80,443` | Múltiplas portas |
| `-p 1-1000` | Intervalo |
| `-p-` | Todas (1–65535) |
| `-F` | Top 100 portas |
| `--top-ports 500` | Top N portas |

---

## 🎯 Host Discovery

| Flag | Função |
|------|--------|
| `-sn` | Ping scan (sem port scan) |
| `-Pn` | Sem discovery (assume ativo) |
| `-PS80,443` | TCP SYN ping |
| `-PA80` | ACK ping |
| `-PU53` | UDP ping |

---

## 🔬 Detecção

| Flag | Função |
|------|--------|
| `-sV` | Versão do serviço |
| `-O` | OS detection |
| `--osscan-guess` | Forçar palpite de OS |
| `-A` | Agressivo (sV+O+sC+traceroute) |
| `-sC` | Scripts NSE padrão |
| `--script=NOME` | Script específico |
| `--script=vuln` | Scripts de vulnerabilidade |

---

## ⏱️ Timing

| Flag | Velocidade | Uso |
|------|-----------|-----|
| `-T0` | Paranoid | Máxima evasão |
| `-T1` | Sneaky | Evasão |
| `-T2` | Polite | Evasão leve |
| `-T3` | Normal | Padrão |
| `-T4` | Aggressive | Labs |
| `-T5` | Insane | Muito rápido, pode perder dados |

---

## 🥷 Evasão

| Flag | Técnica |
|------|---------|
| `-f` | Fragmentação (8 bytes) |
| `--mtu 8` | MTU customizado |
| `-D RND:5` | 5 decoys aleatórios |
| `-g 53` | Source port DNS |
| `--scan-delay 5s` | Delay entre probes |
| `--data-length 50` | Payload aleatório |
| `--ttl 64` | TTL customizado |
| `-S FAKE_IP` | Spoof source IP |

---

## 💾 Output

| Flag | Formato |
|------|---------|
| `-oN arquivo.txt` | Normal (texto) |
| `-oX arquivo.xml` | XML |
| `-oG arquivo.gnmap` | Grepable |
| `-oA prefixo` | Todos os formatos |

---

## 📋 Verbosidade

| Flag | Função |
|------|--------|
| `-v` | Verbose |
| `-vv` | Mais verbose |
| `-d` | Debug |

---

## ⚡ Comandos Prontos

```bash
# Host discovery
nmap -sn 192.168.1.0/24

# Scan rápido (lab)
sudo nmap -F -sV IP

# Scan completo
sudo nmap -p- -T4 IP

# Completo com versão + OS
sudo nmap -T4 -sS -sV -O --osscan-guess -p- IP

# Com scripts de vuln
sudo nmap -sV --script=vuln IP

# Furtivo
sudo nmap -sS -Pn -T2 -f IP

# Máxima evasão (lab)
sudo nmap -sS -Pn -f --mtu 8 --data-length 32 --ttl 64 -D RND:5 -g 53 -T2 IP

# Detectar firewall
nmap -sA -p- IP

# Salvar resultado
sudo nmap -sS -sV -p- IP -oN scan.txt
```

---

## 📌 Relacionado

- [[Nmap — Visão Geral]]
- [[Nmap — Port Scanning]]
- [[Nmap — Evasão & Firewall]]
- [[Cheatsheet — Recon Workflow]]

#cheatsheet #ferramenta/nmap
