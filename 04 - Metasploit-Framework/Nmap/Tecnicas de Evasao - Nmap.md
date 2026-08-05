# 🥷 Técnicas de Evasão — Nmap

> Referência completa de todas as flags de evasão do Nmap.
> Objetivo: parecer menos como um scanner, reduzir probabilidade de detecção.

---

## 🧠 Princípio

> Evasão ≠ invisibilidade.
> É **reduzir a probabilidade de detecção**.

IDS procura padrões: muitos pacotes rápidos, SYN scans óbvios, sequências previsíveis, tamanho de pacote típico do Nmap.
Você quebra esses padrões.

---

## 📋 Referência Rápida

| Flag | Técnica | Contra |
|------|---------|--------|
| `-T0` a `-T2` | Timing lento | IDS por taxa |
| `-f` | Fragmentação | Firewalls antigos |
| `--mtu N` | MTU customizado | Firewalls antigos |
| `-D RND:N` | Decoys | Análise de logs |
| `--scan-delay Ns` | Delay entre probes | IDS por taxa |
| `-g N` | Source port falso | Firewall mal configurado |
| `--data-length N` | Payload aleatório | Assinatura Nmap |
| `--ttl N` | TTL customizado | Fingerprinting |
| `-Pn` | Sem ping discovery | Firewall que bloqueia ICMP |
| `-sA` | ACK scan | Detecção de firewall |

---

## ⏱️ Timing — `-T`

```bash
nmap -T0 IP   # Paranoid  — 5min entre probes
nmap -T1 IP   # Sneaky    — segundos entre probes
nmap -T2 IP   # Polite    — frações de segundo
nmap -T3 IP   # Normal    — padrão
nmap -T4 IP   # Aggressive — labs
nmap -T5 IP   # Insane    — pode perder dados
```

Para evasão real: `-T1` ou `-T2`

---

## ✂️ Fragmentação — `-f` e `--mtu`

```bash
nmap -f IP           # fragmentos de 8 bytes
nmap -f -f IP        # fragmentos de 16 bytes
nmap --mtu 16 IP     # MTU customizado (múltiplo de 8)
nmap --mtu 8 IP      # extremamente fragmentado
```

**Como funciona:** divide o cabeçalho TCP em fragmentos menores.
Firewall antigo via pedaços "inocentes" — host remonta corretamente.

**Limitação moderna:** NGFWs reconstroem fragmentos automaticamente antes de analisar.

---

## 👻 Decoys — `-D`

```bash
nmap -D RND:5 IP                    # 5 IPs aleatórios
nmap -D RND:10 IP                   # 10 decoys
nmap -D IP1,IP2,ME,IP3 IP          # decoys específicos + você no meio
```

**O alvo vê:** scans vindos de 6 ou 11 IPs diferentes.
**IDS precisa:** adivinhar qual é o real.
**Respostas:** voltam só para você — decoys não participam.

> ⚠️ Decoys com IPs de terceiros podem causar problemas legais. Em labs, usar `-D RND`.

---

## 💤 Scan Delay — `--scan-delay`

```bash
nmap --scan-delay 5s IP
nmap --scan-delay 500ms IP
nmap --max-scan-delay 5s IP
```

IDS baseado em taxa precisa que eventos ocorram em janela de tempo próxima. Com delays longos, o scan se torna estatisticamente invisível.

---

## 🎭 Source Port Falso — `-g`

```bash
nmap -g 53 IP    # finge ser DNS
nmap -g 20 IP    # finge ser FTP data
nmap -g 443 IP   # finge ser HTTPS
```

Firewalls mal configurados permitem tráfego de portas "confiáveis" sem inspeção profunda.

---

## 📦 Payload Aleatório — `--data-length`

```bash
nmap --data-length 25 IP
nmap --data-length 50 IP
```

Nmap padrão gera pacotes de tamanho específico e reconhecível.
Adicionar bytes aleatórios quebra a assinatura.

---

## 🕐 TTL Customizado — `--ttl`

```bash
nmap --ttl 64 IP    # Linux
nmap --ttl 128 IP   # Windows
nmap --ttl 255 IP   # Cisco/roteador
```

TTL padrão varia por SO. Disfarça a origem do scan imitando outros sistemas.

---

## 🔇 Sem Ping — `-Pn`

```bash
nmap -Pn IP
```

Firewalls frequentemente bloqueiam ICMP. Com `-Pn` o Nmap assume que o host está vivo e vai direto para o port scan sem tentar ping.

---

## 🔍 ACK Scan — `-sA`

```bash
nmap -sA IP
nmap -sA -p- IP
```

Não descobre portas abertas — detecta firewall.
`filtered` = firewall ativo. `unfiltered` = pacote passou.

---

## ⚡ Combinações Reais

### Lab padrão (furtivo mas funcional)
```bash
sudo nmap -sS -Pn -T2 -f IP
```

### Ambiente hostil
```bash
sudo nmap -sS -Pn -D RND:10 --scan-delay 2s IP
```

### Evasão máxima (labs/CTF)
```bash
sudo nmap -sS -Pn \
  -f --mtu 8 \
  --data-length 32 \
  --ttl 64 \
  -D RND:5 \
  -g 53 \
  -T2 IP
```

---

## ⚠️ Limitações Modernas

| Técnica | Eficaz Contra | Não Funciona Contra |
|---------|--------------|---------------------|
| Fragmentação | Firewalls antigos | NGFW (remontam antes de analisar) |
| Decoys | Análise simples de logs | Correlação comportamental |
| Timing lento | IDS por taxa | IDS comportamental |
| Source port | Firewall sem DPI | NGFW com inspeção profunda |

---

## 📌 Relacionados

- [[Nmap — Evasão & Firewall]]
- [[Firewall — Conceito]]
- [[IDS & IPS]]
- [[Nmap — Port Scanning]]

#evasao #defesas #ferramenta/nmap
