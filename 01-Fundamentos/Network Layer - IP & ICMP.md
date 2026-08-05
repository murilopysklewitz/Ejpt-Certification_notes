# 🌐 Network Layer — IP & ICMP

> **Camada 3 do OSI.**
> Responsável por permitir comunicação **entre redes diferentes**.
> Se a camada 2 resolve entrega dentro da LAN, a camada 3 resolve entrega **entre redes**.

---

## 📌 Funções Principais

**Endereçamento lógico** — cada dispositivo recebe um IP (não físico como MAC).
- IPv4 → `192.168.1.10`
- IPv6 → `2001:db8::1`

**Roteamento** — escolher o melhor caminho até o destino via tabelas de roteamento e métricas.

**Encapsulamento** — recebe segmentos da camada 4 e adiciona IP de origem + destino → cria o **Packet (pacote)**.

**Fragmentação** — se o pacote for maior que o MTU, é dividido em fragmentos menores.

---

## 🔧 Protocolos

### IP (Internet Protocol)
- Principal protocolo da camada 3
- Entrega **best-effort** (sem garantia — confiabilidade fica no TCP)
- Versões: IPv4 e IPv6

### ICMP (Internet Control Message Protocol)
- Mensagens de erro e diagnóstico de rede
- `ping` → Echo Request / Echo Reply
- `traceroute` → usa ICMP ou UDP

### IPsec
- Criptografia no nível IP
- Base para VPNs

---

## 🧭 Como o Roteamento Funciona

```
Host quer enviar dados para outro IP
        ↓
Destino está na mesma rede?
    SIM → envia direto (ARP → camada 2)
    NÃO → envia para o gateway (roteador)
        ↓
Roteador lê IP destino
        ↓
Consulta tabela de rotas
        ↓
Encaminha para próximo salto (next hop)
        ↓
Repete até chegar ao destino
```

---

## 🔑 Conceitos Importantes

### TTL (Time To Live)
- Campo no pacote IP que **diminui a cada roteador**
- Evita loops infinitos na rede
- Quando chega a 0, o pacote é descartado e um ICMP é enviado de volta
- Usado pelo `traceroute` para mapear roteadores

Valores padrão por SO:
| Sistema | TTL Padrão |
|---------|-----------|
| Linux | 64 |
| Windows | 128 |
| Cisco | 255 |

### Gateway Padrão (Default Gateway)
Roteador que conecta sua rede a outras redes (ex: internet). Todo pacote para fora da LAN passa por ele.

### NAT (Network Address Translation)
Tradução entre IP privado e IP público. Muito comum em redes domésticas e corporativas.

### Subnetting
Dividir redes em sub-redes menores.
```
192.168.1.0/24  →  256 hosts possíveis
192.168.1.0/25  →  128 hosts por sub-rede
```

---

## ⚔️ L2 vs L3 — Diferença Chave

| | Camada 2 (Data Link) | Camada 3 (Network) |
|-|---------------------|-------------------|
| Endereço | MAC address | IP address |
| Escopo | Rede local (LAN) | Entre redes |
| Dispositivo | Switch | Roteador |

> **Analogia:**
> MAC = endereço da casa dentro do condomínio
> IP = endereço da cidade no mapa mundial

---

## 🔎 Na Prática — Pentest

```bash
# Ping sweep (ICMP)
nmap -sn 192.168.1.0/24

# Traceroute
traceroute IP
nmap --traceroute IP

# ICMP ping simples
ping 192.168.1.1
```

Ataques comuns na camada 3:
- IP spoofing
- ICMP enumeration
- Routing manipulation

---

## 📌 Relacionados

- [[Modelo OSI]]
- [[Transport Layer — TCP & UDP]]
- [[Host Discovery]]
- [[Nmap — Host Discovery]]

#fundamentos #protocolo/ip #protocolo/icmp #rede
