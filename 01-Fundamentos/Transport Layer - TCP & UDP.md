# 🚦 Transport Layer — TCP & UDP

> **Camada 4 do OSI.**
> Responsável pela comunicação **fim-a-fim entre processos** (aplicações) em dispositivos diferentes.
> A Network Layer entrega entre *hosts* (IP→IP). A Transport Layer entrega entre *aplicações* (porta→porta).

---

## 📌 Funções Principais

**Segmentação** — divide dados grandes em partes menores e reagrupa no destino.

**Multiplexação via portas** — permite várias aplicações usarem a rede simultaneamente.
```
IP + PORTA = socket
192.168.1.10:443  →  HTTPS
192.168.1.10:22   →  SSH
```

**Controle de fluxo** — evita sobrecarregar o receptor.

**Controle de erros** — detecta perda/corrupção e solicita retransmissão (só TCP).

---

## 🟢 TCP — Transmission Control Protocol

**Orientado à conexão. Confiável.**

| Característica | Valor |
|---------------|-------|
| Conexão | Sim (handshake) |
| Confiabilidade | Sim |
| Ordem garantida | Sim |
| Velocidade | Mais lento |
| Uso típico | HTTP, SSH, FTP, SMTP |

### 🤝 Three-Way Handshake

```
Cliente  →  SYN       →  Servidor     "quero conectar"
Cliente  ←  SYN-ACK   ←  Servidor     "ok, pode vir"
Cliente  →  ACK       →  Servidor     "confirmado"
              [conexão estabelecida]
```

> **Por que isso importa em pentest?**
> O SYN scan (`-sS`) manda SYN e analisa a resposta **sem completar** o handshake.
> SYN-ACK = porta aberta. RST = fechada. Sem resposta = filtrada.

### Estados TCP Importantes

| Estado | Significado |
|--------|-------------|
| LISTEN | Esperando conexão |
| SYN_SENT | SYN enviado |
| ESTABLISHED | Conexão ativa |
| FIN_WAIT | Encerrando |
| CLOSE_WAIT | Outro lado encerrou |

---

## 🟡 UDP — User Datagram Protocol

**Sem conexão. Sem garantia. Rápido.**

| Característica | Valor |
|---------------|-------|
| Conexão | Não |
| Confiabilidade | Não |
| Ordem garantida | Não |
| Velocidade | Mais rápido |
| Uso típico | DNS, streaming, VoIP, jogos |

> UDP não confirma entrega. Se o pacote se perder, foi-se.
> É a troca: velocidade por confiabilidade.

---

## ⚔️ TCP vs UDP

| | TCP | UDP |
|-|-----|-----|
| Conexão | ✅ Sim | ❌ Não |
| Confirmação | ✅ Sim | ❌ Não |
| Ordem | ✅ Sim | ❌ Não |
| Velocidade | 🐢 Mais lento | ⚡ Mais rápido |
| Overhead | Alto | Baixo |

---

## 🔢 Portas

**Well-known (0–1023):** reservadas para serviços conhecidos

| Porta | Serviço |
|-------|---------|
| 21 | FTP |
| 22 | SSH |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 445 | SMB |
| 3306 | MySQL |
| 3389 | RDP |

**Registered (1024–49151):** aplicações específicas
**Dynamic/Ephemeral (49152–65535):** alocadas dinamicamente

---

## 🔎 Na Prática — Pentest

```bash
# TCP SYN scan (o mais usado)
sudo nmap -sS IP

# UDP scan (mais lento, mas importante)
sudo nmap -sU IP
sudo nmap -sU --top-ports 50 IP
```

Ataques comuns na camada 4:
- Port scanning
- TCP SYN flood (DoS)
- Session hijacking

---

## 🧠 Insight Central

> A Network Layer responde: **"qual computador?"**
> A Transport Layer responde: **"qual aplicação dentro do computador?"**

---

## 📌 Relacionados

- [[Modelo OSI]]
- [[Network Layer — IP & ICMP]]
- [[Nmap — Port Scanning]]
- [[Cheatsheet — Portas Importantes]]

#fundamentos #protocolo/tcp #protocolo/udp #rede
