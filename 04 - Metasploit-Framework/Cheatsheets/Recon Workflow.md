# ⚡ Cheatsheet — Recon Workflow

> Fluxo completo de reconhecimento do zero até mapeamento de serviços.

---

## 🔁 Fluxo Geral

```
PASSIVO (sem tocar no alvo)
    ↓
ATIVO (contato com o alvo)
    ↓
ENUMERAÇÃO WEB
    ↓
ANÁLISE & EXPLORAÇÃO
```

---

## 1️⃣ Reconhecimento Passivo

```bash
# Registro do domínio
whois dominio.com

# IP do domínio
host dominio.com
host -t MX dominio.com   # servidores de email
host -t NS dominio.com   # name servers

# DNS completo
dnsrecon -d dominio.com

# Subdomínios via DNS
dnsdumpster dominio.com   # ou via https://dnsdumpster.com

# Subdomínios via OSINT (buscadores)
sublist3r -d dominio.com
sublist3r -d dominio.com -e google,bing,yahoo

# Emails, IPs, subdomínios via buscadores
theHarvester -d dominio.com -b all

# Google Dorks
site:*.dominio.com
inurl:admin site:dominio.com
filetype:pdf site:dominio.com
intitle:"index of" site:dominio.com

# Tecnologias (browser)
# https://sitereport.netcraft.com/?url=dominio.com

# Emails/senhas vazados
# https://haveibeenpwned.com
```

---

## 2️⃣ Fingerprinting Web

```bash
# Detectar WAF
wafw00f dominio.com

# Tecnologias do site
whatweb dominio.com
whatweb -v dominio.com

# Headers HTTP
curl -I http://dominio.com

# Arquivos informativos
curl http://dominio.com/robots.txt
curl http://dominio.com/sitemap.xml
curl http://dominio.com/sitemap_index.xml
```

---

## 3️⃣ Host Discovery

```bash
# Descobrir hosts ativos (rede local)
nmap -sn 192.168.1.0/24

# Se ICMP bloqueado
nmap -sn -PS80,443 192.168.1.0/24

# ARP scan (LAN)
arp-scan 192.168.1.0/24
netdiscover -i eth0 -r 192.168.1.0/24
```

---

## 4️⃣ Port Scanning

```bash
# Scan completo de portas (sempre que possível)
sudo nmap -p- -T4 IP

# Scan rápido (top 1000)
sudo nmap -sS IP

# Scan UDP (serviços importantes)
sudo nmap -sU --top-ports 50 IP

# Detectar firewall
nmap -sA -p- IP
```

---

## 5️⃣ Service & OS Detection

```bash
# Versão dos serviços nas portas abertas
sudo nmap -sV -p 22,80,443 IP

# OS detection
sudo nmap -O --osscan-guess IP

# Tudo junto (eficiente)
sudo nmap -sV -O --osscan-guess -p PORTAS_ABERTAS IP
```

---

## 6️⃣ NSE Scripts

```bash
# Scripts padrão
sudo nmap -sC -p PORTAS IP

# Vulnerabilidades
sudo nmap --script=vuln -p PORTAS IP

# FTP anônimo
sudo nmap --script=ftp-anon -p21 IP

# SMB
sudo nmap --script=smb-enum-shares,smb-os-discovery -p445 IP

# HTTP
sudo nmap --script=http-title,http-headers -p80 IP
```

---

## 7️⃣ Enumeração Web

```bash
# Diretórios (básico)
dirb http://IP

# Diretórios + arquivos (completo)
dirsearch -u http://IP -e php,html,txt,js

# Com wordlist e extensões
gobuster dir \
  -u http://IP \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,html,txt \
  -t 30

# DNS interrogation / zone transfer
dig axfr @ns1.dominio.com dominio.com
dnsenum dominio.com
fierce -dns dominio.com
```

---

## 8️⃣ Salvar Tudo

```bash
# Nmap com output
sudo nmap -sS -sV -p- IP -oN scan_$(date +%Y%m%d_%H%M).txt

# Dirsearch com output
dirsearch -u http://IP -o web_$(date +%Y%m%d_%H%M).txt
```

---

## 🔥 One-liner Lab Completo

```bash
# Recon completo em um comando
sudo nmap -T4 -sS -sV -O --osscan-guess -p- -oN scan.txt IP
```

---

## 📌 Relacionado

- [[Cheatsheet — Nmap Flags]]
- [[Cheatsheet — Portas Importantes]]
- [[MOC - Recon]]

#cheatsheet #recon
