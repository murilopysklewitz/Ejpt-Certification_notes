# 🥷 Nmap — Evasão & Detecção de Firewall

> Como detectar firewalls e como fazer o Nmap parecer menos com um scanner.
> Evasão ≠ invisibilidade. É reduzir a probabilidade de detecção.

---

## 🔎 Detectando Firewall

### Scan Normal vs Filtrado
```bash
nmap -p 22 IP
```

| Estado | Significado |
|--------|-------------|
| `open` | Respondeu — porta acessível |
| `closed` | Respondeu negativamente — host vivo, sem serviço |
| `filtered` | Sem resposta — **cheiro forte de firewall** |

---

### ACK Scan — Detecção Clássica
```bash
nmap -sA IP
```

Este scan **não descobre portas abertas**. Serve especificamente para detectar firewall.

| Resultado | Interpretação |
|-----------|--------------|
| `unfiltered` | Pacote chegou ao host — sem firewall ativo nessa porta |
| `filtered` | Firewall bloqueando |

---

### Mapeamento de Regras do Firewall
```bash
nmap -sA -p- IP
```

Escanear todas as portas com ACK revela o **mapa de regras do firewall**: quais portas passam e quais são bloqueadas.

---

## 🥷 Técnicas de Evasão de IDS

IDS procura padrões como: muitos pacotes rápidos, SYN scans óbvios, sequências previsíveis.
O objetivo é **parecer menos um scanner**.

---

### Timing Control (-T)
```bash
nmap -T2 IP   # lento
nmap -T1 IP   # extremamente furtivo (paranoid)
```

IDS detecta facilmente 10.000 pacotes em 2 segundos. Humanos são lentos — imite isso.

| T0 | Paranoid (5min entre probes) |
| T1 | Sneaky |
| T2 | Polite |
| T3 | Normal (padrão) |
| T4 | Aggressive (labs) |
| T5 | Insane |

---

### Fragmentação de Pacotes (-f)
```bash
nmap -f IP
nmap --mtu 16 IP
nmap --mtu 8 IP
```

Divide o pacote TCP em fragmentos menores.

| MTU | Efeito |
|-----|--------|
| 1500 | Normal (sem fragmentação) |
| 32 | Fragmentado |
| 8 | Extremamente fragmentado |

**Como funciona:** firewalls antigos analisavam pacotes individualmente, não o reconstruído. O host final remonta corretamente. O firewall via pedaços "inocentes".

**Realidade moderna:** firewalls atuais reconstroem fragmentos antes de analisar. Eficácia limitada hoje, mas útil em ambientes legados.

---

### Scan Delay (--scan-delay)
```bash
nmap --scan-delay 5s IP
```

IDS baseado em taxa não detecta o que não vê como anômalo. 5 segundos entre probes → invisível para detecção por taxa.

---

### Decoys — Clones Falsos (-D)
```bash
nmap -D RND:5 IP
nmap -D IP1,IP2,ME,IP3 IP
```

O alvo vê scans vindos de múltiplos IPs. Logs ficam assim:
```
10.0.0.5
23.44.1.9
SEU_IP_REAL
91.22.3.1
```

IDS precisa adivinhar qual é o IP real.

> ⚠️ Respostas só voltam para você. Os decoys apenas confundem os logs, não recebem respostas.

---

### Source Port Falso (-g)
```bash
nmap -g 53 IP   # finge ser DNS
nmap -g 443 IP  # finge ser HTTPS
nmap -g 20 IP   # finge ser FTP data
```

Firewall mal configurado pensa:
> "tráfego porta 53 = DNS = permitido ✅"

Mas é scan disfarçado.

---

### Dados Aleatórios (--data-length)
```bash
nmap --data-length 50 IP
```

Adiciona payload aleatório ao pacote. Quebra a assinatura padrão do Nmap que IDS conhece.

---

### TTL Customizado (--ttl)
```bash
nmap --ttl 64 IP    # parecer Linux
nmap --ttl 128 IP   # parecer Windows
nmap --ttl 255 IP   # parecer Cisco
```

Disfarça a origem do scan imitando diferentes sistemas.

---

### Ignorar ICMP Discovery (-Pn)
```bash
nmap -Pn IP
```

Firewalls bloqueiam ICMP. Com `-Pn` você assume que o host está vivo e pula o ping, indo direto para o port scan.

---

## ⚡ Combinações Reais

**Pentest padrão moderado:**
```bash
sudo nmap -sS -Pn -T2 -f IP
```

**Ambiente hostil:**
```bash
sudo nmap -sS -Pn -D RND:10 --scan-delay 2s IP
```

**Combinação completa de evasão:**
```bash
sudo nmap -sS -Pn -f --mtu 8 \
  --data-length 32 \
  --ttl 64 \
  -D RND:5 \
  -g 53 IP
```

O resultado: pacotes fragmentados, tamanho variável, origem falsa, múltiplos IPs aparentes, porta confiável. Você deixou de parecer Nmap.

---

## ⚠️ Realidade Moderna

Equipamentos atuais:
- Reconstroem fragmentos automaticamente
- Detectam decoys por análise de comportamento
- Correlacionam sessões (um IP muda comportamento mas o padrão se repete)
- Usam ML/análise estatística

| Situação | Eficácia da Evasão |
|---------|-------------------|
| Labs e CTFs | Alta |
| Configs ruins / legado | Média-Alta |
| Redes enterprise modernas | Baixa |

---

## 🧠 Os 3 Níveis de Recon

| Nível | Abordagem |
|-------|-----------|
| 1 — Iniciante | Scan direto sem pensar |
| 2 — Intermediário | Scan furtivo com evasão |
| 3 — Avançado | Analisa o sistema defensivo antes de agir |

---

## 📌 Relacionados

- [[Firewall — Conceito]]
- [[IDS & IPS]]
- [[Nmap — Port Scanning]]
- [[Nmap — Host Discovery]]

#ferramenta/nmap #evasao #defesas
