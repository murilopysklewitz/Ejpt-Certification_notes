# 👁️ IDS & IPS

> Sistemas de detecção e prevenção de intrusões.
> Enquanto o firewall filtra por regras fixas, IDS/IPS analisa **comportamento**.

---

## 🧠 Diferença Fundamental

| Sistema | Função | Age? |
|---------|--------|------|
| **Firewall** | Filtra pacotes por regras | Bloqueia conforme regra |
| **IDS** | Detecta comportamento suspeito | ❌ Só alerta |
| **IPS** | Detecta + age automaticamente | ✅ Bloqueia/altera |

---

## 🔎 IDS — Intrusion Detection System

**Câmera de segurança.**

O IDS monitora o tráfego e gera alertas quando detecta padrões suspeitos. Não bloqueia nada — apenas avisa o time de segurança.

### O Que o IDS Detecta

- Muitos pacotes em pouco tempo (port sweep)
- SYN scans óbvios
- Padrões de assinatura conhecidos (assinatura do Nmap)
- Tentativas de login repetidas
- Payloads maliciosos conhecidos

### Tipos de Detecção

| Tipo | Método | Vantagem | Limitação |
|------|--------|----------|-----------|
| Signature-based | Compara com assinaturas conhecidas | Preciso para ameaças conhecidas | Zero-days passam |
| Anomaly-based | Detecta desvios do comportamento normal | Pega zero-days | Muitos falsos positivos |
| Behavioral | Analisa padrões de comportamento | Contextual | Complexo |

---

## 🛡️ IPS — Intrusion Prevention System

**Segurança armado.**

O IPS detecta E age. Quando identifica tráfego malicioso, pode:
- Bloquear o pacote
- Resetar a conexão
- Colocar IP em blacklist automática
- Modificar o pacote

---

## 🥷 Evitando Detecção

### Timing (reduzir velocidade)
```bash
nmap -T2 IP    # lento
nmap -T1 IP    # muito lento
nmap --scan-delay 5s IP   # delay entre probes
```
IDS baseado em taxa não consegue correlacionar eventos lentos.

---

### Fragmentação (quebrar assinatura)
```bash
nmap -f IP
nmap --mtu 8 IP
```
Divide pacotes — assinatura do Nmap fica irreconhecível.

---

### Tamanho de Pacote (quebrar assinatura)
```bash
nmap --data-length 50 IP
```
Nmap padrão tem pacotes de tamanho específico. Adicionar payload quebra a assinatura.

---

### Decoys (confundir logs)
```bash
nmap -D RND:10 IP
```
IDS vê múltiplas fontes — não sabe qual é real.

---

### Source Port (parecer tráfego legítimo)
```bash
nmap -g 53 IP   # DNS
nmap -g 443 IP  # HTTPS
```
IDS pode não inspecionar tráfego de portas "confiáveis".

---

## ⚠️ Realidade

IDS/IPS modernos:
- Usam ML e análise comportamental
- Correlacionam múltiplas sessões
- Detectam evasão de evasão (meta-evasão)
- Integram com SIEM para análise contextual

Evasão reduz probabilidade de detecção — não garante invisibilidade.

---

## 📌 Relacionados

- [[Firewall — Conceito]]
- [[Nmap — Evasão & Firewall]]
- [[Network Mapping]]

#defesas #evasao #conceito
