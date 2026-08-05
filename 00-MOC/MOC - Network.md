# 🗺️ MOC — Fundamentos de Rede

> Base teórica necessária para entender o que as ferramentas realmente fazem.
> Sem isso, você executa comandos — com isso, você entende o que acontece.

---

## 🧱 Modelo OSI

- [[Modelo OSI]] — as 7 camadas e o que cada uma faz

| Camada | Nome | Relevância em Pentest |
|--------|------|----------------------|
| 7 | Application | HTTP, DNS, FTP |
| 4 | Transport | TCP/UDP, port scanning |
| 3 | Network | IP, roteamento, ICMP |
| 2 | Data Link | ARP, MAC, LAN |

---

## 🌐 Protocolos Essenciais

- [[Transport Layer — TCP & UDP]] — handshake, portas, estados
- [[Network Layer — IP & ICMP]] — roteamento, TTL, NAT
- [[DNS — Records & Funcionamento]] — como o DNS funciona e seus tipos

---

## 🔁 Fluxo de Dados (simplificado)

```
Aplicação (HTTP)
    ↓
Transporte (TCP — porta 80)
    ↓
Rede (IP — endereçamento)
    ↓
Enlace (MAC — entrega local)
    ↓
Física (bits no cabo)
```

---

## 🔎 Relevância para Pentest

| Camada | Ataque/Técnica |
|--------|---------------|
| L7 | SQL Injection, XSS, diretórios |
| L4 | Port scanning, SYN flood |
| L3 | IP spoofing, ICMP enum |
| L2 | ARP spoofing, MAC flooding |

---

## 📌 Relacionados

- [[MOC - Recon]]
- [[Nmap — Visão Geral]]
- [[Cheatsheet — Portas Importantes]]

#moc #fundamentos #rede
