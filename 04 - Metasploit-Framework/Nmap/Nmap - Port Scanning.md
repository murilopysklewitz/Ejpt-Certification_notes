# 🔌 Nmap — Port Scanning

> Descobrir quais portas estão abertas, fechadas ou filtradas em um host.
> Port scanning não é "tentar entrar" — é medir a resposta do protocolo TCP/IP.

---

## 🧠 Como Funciona (TCP SYN Scan)

```
Nmap envia:   SYN
Resposta:     SYN-ACK  →  porta OPEN   ✅
              RST      →  porta CLOSED ❌
              nada     →  porta FILTERED (firewall) 🚫
```

O SYN scan não completa o handshake — por isso é semi-stealth.

---

## 🔧 Tipos de Scan

### -sS — SYN Scan (o mais usado)
```bash
sudo nmap -sS IP
```
- Não completa o handshake TCP
- Rápido e relativamente discreto
- **Precisa de sudo** (raw sockets)
- Padrão em pentest

---

### -sT — TCP Connect Scan
```bash
nmap -sT IP
```
- Completa o handshake (connect() completo)
- Mais detectável
- **Não precisa de sudo**
- Usado quando não tem privilégios

---

### -sU — UDP Scan
```bash
sudo nmap -sU IP
sudo nmap -sU --top-ports 50 IP
```
- Mais lento e menos confiável
- Importante para DNS (53), SNMP (161), DHCP (67/68)
- Sem confirmação de entrega no UDP

---

## 🔢 Seleção de Portas

```bash
# Portas específicas
nmap -p 22,80,443 IP

# Intervalo
nmap -p 1-1000 IP

# Todas as 65535 portas
sudo nmap -p- IP

# 100 portas mais comuns
nmap -F IP

# Top N portas
nmap --top-ports 500 IP
```

---

## 📊 Estados das Portas

| Estado | Significado | O Que Fazer |
|--------|-------------|-------------|
| `open` | Serviço ativo | Enumerar versão |
| `closed` | Sem serviço, host responde | Ignorar (mas host está vivo) |
| `filtered` | Firewall bloqueando | Tentar evasão |
| `open\|filtered` | Nmap não sabe (UDP) | Investigar manualmente |

---

## ⚡ Combinações Práticas

```bash
# Lab padrão (rápido e completo)
sudo nmap -T4 -sS -p- IP

# Rápido com serviços
sudo nmap -F -sV IP

# Scan silencioso
sudo nmap -sS -T2 IP

# Scan + salvar
sudo nmap -sS -p- IP -oN resultado.txt
```

---

## ⚠️ Erros Comuns

**Só escanear as 1000 portas padrão** → serviços interessantes frequentemente ficam em portas altas (8080, 8443, 9090, etc.). Sempre use `-p-` quando possível.

**Usar `-A` em tudo** → barulhento e lento. Melhor: scan de portas primeiro, depois `-sV` só nas abertas.

---

## 📌 Relacionados

- [[Nmap — Visão Geral]]
- [[Nmap — Host Discovery]]
- [[Nmap — Service & OS Detection]]
- [[Transport Layer — TCP & UDP]]
- [[Firewall — Conceito]]

#ferramenta/nmap #protocolo/tcp #recon/ativo
