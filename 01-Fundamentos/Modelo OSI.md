# 🧱 Modelo OSI

> O modelo OSI é uma **abstração em 7 camadas** que descreve como dados trafegam em redes.
> Ele não é um protocolo real — é um modelo conceitual para organizar e entender comunicação entre sistemas.

---

## 📊 As 7 Camadas

| # | Nome | PDU | Função Principal | Protocolos/Equipamentos |
|---|------|-----|-----------------|------------------------|
| 7 | Application | Dados | Interface com aplicações | HTTP, FTP, DNS, SMTP |
| 6 | Presentation | Dados | Formatação, criptografia | SSL/TLS |
| 5 | Session | Dados | Gerencia sessões/diálogo | — |
| 4 | Transport | Segmento | Comunicação fim-a-fim | TCP, UDP |
| 3 | Network | Pacote | Roteamento entre redes | IP, ICMP → Roteadores |
| 2 | Data Link | Frame | Entrega dentro da LAN | Ethernet, ARP → Switches |
| 1 | Physical | Bits | Transmissão física | Cabos, sinais |

---

## 🔁 Fluxo dos Dados

```
EMISSOR                          RECEPTOR
   ↓                                ↑
7. Application   →   dados   →   7. Application
6. Presentation  →   format  →   6. Presentation
5. Session       →   sessão  →   5. Session
4. Transport     →  segmento →   4. Transport
3. Network       →   pacote  →   3. Network
2. Data Link     →   frame   →   2. Data Link
1. Physical      →   bits    →   1. Physical
```

Os dados **descem** ao sair e **sobem** ao chegar.

---

## 🎯 Por Camada — O Que Acontece

### 7 — Application
Interface direta com o usuário ou software. Onde você digita a URL ou manda um email.

### 6 — Presentation
Garante que os dados estejam no formato certo. Aqui entra a criptografia TLS.

### 5 — Session
Controla quando uma comunicação começa, é mantida e termina.

### 4 — Transport
Divide os dados em segmentos e garante (TCP) ou não (UDP) que cheguem corretamente.
Aqui vivem as **portas**.

### 3 — Network
Decide o caminho entre redes diferentes. Aqui vivem os **endereços IP**.

### 2 — Data Link
Entrega dentro da mesma rede local. Aqui vivem os **endereços MAC**.

### 1 — Physical
Converte tudo em sinais elétricos, luz ou rádio.

---

## 🔎 Relevância em Pentest

| Camada | Técnica/Ataque |
|--------|---------------|
| 7 — Application | SQL Injection, XSS, fuzzing de diretórios |
| 4 — Transport | Port scanning TCP/UDP |
| 3 — Network | Ping sweep, IP spoofing, traceroute |
| 2 — Data Link | ARP spoofing, ARP scan |

---

## 🧠 Mnemônico

> **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing
> Application → Presentation → Session → Transport → Network → Data Link → Physical

---

## 📌 Relacionados

- [[Transport Layer — TCP & UDP]]
- [[Network Layer — IP & ICMP]]
- [[MOC - Network]]

#fundamentos #rede #osi #conceito
