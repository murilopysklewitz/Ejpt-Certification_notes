# 🧱 Firewall — Conceito

> Sistema que **bloqueia ou filtra pacotes** baseado em regras.
> É o porteiro da rede — decide o que entra, o que sai e o que é ignorado.

---

## 🧠 O Que É

Um firewall analisa pacotes de rede e decide o que fazer com eles baseado em regras pré-configuradas. Pode filtrar por IP, porta, protocolo e estado da conexão.

---

## 🔧 Decisões do Firewall

```
Pacote chega
    ↓
Verificar regras
    ↓
✅ PASSA     — regra permite
❌ BLOQUEIA  — regra nega (envia RST ou ICMP unreachable)
👻 IGNORA    — descarta silenciosamente (DROP — sem resposta)
```

---

## 📊 Tipos de Firewall

| Tipo | Camada OSI | O Que Analisa |
|------|-----------|--------------|
| Packet Filter | L3/L4 | IP, porta, protocolo |
| Stateful | L4 | Estado da conexão TCP |
| Application (WAF) | L7 | Conteúdo HTTP, SQL, XSS |
| Next-Gen (NGFW) | L3-L7 | Tudo + comportamento + IA |

---

## 🔎 Detectando Firewall com Nmap

### Porta filtrada = indício de firewall
```bash
nmap -p 22 IP
```
Estado `filtered` → algo bloqueou a resposta.

---

### ACK Scan — detecção clássica
```bash
nmap -sA IP
```

| Resultado | Interpretação |
|-----------|--------------|
| `unfiltered` | Pacote passou — sem firewall ativo |
| `filtered` | Firewall está bloqueando |

---

### Mapeamento de Regras
```bash
nmap -sA -p- IP
```
Revela quais portas passam e quais são bloqueadas → mapa das regras.

---

## 🥷 Técnicas para Contornar

| Técnica | Comando | Mecanismo |
|---------|---------|-----------|
| Fragmentação | `nmap -f IP` | Divide pacotes em pedaços |
| MTU customizado | `nmap --mtu 8 IP` | Fragmentos extremamente pequenos |
| Source port falso | `nmap -g 53 IP` | Finge ser DNS |
| ACK scan | `nmap -sA IP` | Usa pacotes que passam stateful |
| Timing lento | `nmap -T1 IP` | Abaixo do threshold do IDS |
| Ignorar ICMP | `nmap -Pn IP` | Assume host ativo sem ping |

---

## ⚠️ Firewalls Modernos

Equipamentos atuais:
- Reconstroem fragmentos antes de analisar
- Detectam port scanning mesmo lento
- Analisam comportamento e correlacionam sessões
- Usam ML para identificar anomalias

Evasão funciona melhor contra configs legadas ou mal configuradas.

---

## 🧠 Analogia

```
Firewall  →  Porteiro
IDS       →  Câmera de segurança
IPS       →  Segurança armado
```

---

## 📌 Relacionados

- [[IDS & IPS]]
- [[Nmap — Evasão & Firewall]]
- [[Wafw00f]]
- [[Network Mapping]]

#defesas #evasao #conceito
