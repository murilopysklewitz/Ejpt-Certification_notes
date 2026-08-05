# 💾 Nmap — Output Formats

> Salvar o resultado dos scans é tão importante quanto o scan em si.
> Você vai querer rever, grepar, importar em outras ferramentas e documentar.

---

## 📋 Tipos de Output

| Flag | Formato | Extensão Comum | Melhor Para |
|------|---------|---------------|------------|
| `-oN` | Normal | `.txt` | Leitura humana |
| `-oX` | XML | `.xml` | Importar em ferramentas (Metasploit, Faraday) |
| `-oG` | Grepable | `.gnmap` | `grep`, `awk`, scripts bash |
| `-oA` | Todos os três | (prefixo) | Quando não sabe o que vai precisar depois |
| `-oS` | Script Kiddie | `.txt` | Nunca use isso |

---

## 🔧 Uso de Cada Flag

### -oN — Normal (texto legível)
```bash
sudo nmap -sS -sV IP -oN resultado.txt
```

Output é idêntico ao que aparece no terminal. Fácil de ler, difícil de parsear automaticamente.

```
# Nmap 7.94 scan initiated Mon Mar 02 10:00:00 2026
Nmap scan report for 192.168.1.10
Host is up (0.00045s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1
80/tcp   open  http    Apache httpd 2.4.49
443/tcp  open  https   Apache httpd 2.4.49

# Nmap done: 1 IP address (1 host up) scanned in 8.42 seconds
```

---

### -oX — XML
```bash
sudo nmap -sS -sV IP -oX resultado.xml
```

Formato estruturado. Pode ser importado diretamente no **Metasploit**, **Faraday**, **Dradis** e outras plataformas de pentest.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<nmaprun scanner="nmap" ...>
  <host>
    <address addr="192.168.1.10" addrtype="ipv4"/>
    <ports>
      <port protocol="tcp" portid="22">
        <state state="open"/>
        <service name="ssh" product="OpenSSH" version="8.2p1"/>
      </port>
    </ports>
  </host>
</nmaprun>
```

**Importar no Metasploit:**
```bash
msf> db_import resultado.xml
msf> hosts
msf> services
```

---

### -oG — Grepable
```bash
sudo nmap -sS -sV IP -oG resultado.gnmap
```

Cada host fica em uma linha. Fácil de processar com `grep` e `awk`.

```
Host: 192.168.1.10 ()   Status: Up
Host: 192.168.1.10 ()   Ports: 22/open/tcp//ssh//OpenSSH 8.2p1/, 80/open/tcp//http//Apache 2.4.49/
```

**Exemplos de grep no output grepable:**
```bash
# Hosts com porta 80 aberta
grep "80/open" resultado.gnmap

# Só os IPs que estão up
grep "Status: Up" resultado.gnmap | awk '{print $2}'

# Extrair IPs com SSH aberto
grep "22/open" resultado.gnmap | cut -d" " -f2

# Portas abertas de um host específico
grep "192.168.1.10" resultado.gnmap
```

---

### -oA — Todos os Formatos de Uma Vez
```bash
sudo nmap -sS -sV IP -oA scan_resultado
```

Gera três arquivos com o mesmo prefixo:
```
scan_resultado.nmap    <- formato normal
scan_resultado.xml     <- XML
scan_resultado.gnmap   <- grepable
```

Use quando não tem certeza do que vai precisar depois. Custo zero, cobertura total.

---

## 🗂️ Boas Práticas de Organização

### Nomear com data e alvo
```bash
sudo nmap -sS -sV IP -oA scans/$(date +%Y%m%d_%H%M)_192.168.1.10
```

Resultado:
```
scans/20260302_1430_192.168.1.10.nmap
scans/20260302_1430_192.168.1.10.xml
scans/20260302_1430_192.168.1.10.gnmap
```

---

### Estrutura de pastas por engajamento
```
pentest/
└── alvo/
    ├── recon/
    │   ├── nmap/
    │   │   ├── host_discovery.nmap
    │   │   ├── port_scan.gnmap
    │   │   └── full_scan.xml
    │   ├── web/
    │   └── dns/
    └── exploitation/
```

---

## ⚡ Combinações Úteis

### Scan completo salvando tudo
```bash
sudo nmap -T4 -sS -sV -O -p- IP -oA scans/full_$(date +%Y%m%d)
```

---

### Extrair só IPs ativos do grepable
```bash
grep "Status: Up" scan.gnmap | awk '{print $2}' > hosts_vivos.txt
```

---

### Extrair portas abertas para uso posterior
```bash
# Formato: "22,80,443" para usar em -p
grep "Ports:" scan.gnmap | \
  grep -oP '\d+/open' | \
  cut -d/ -f1 | \
  tr '\n' ',' | \
  sed 's/,$//'
```

---

### Usar lista de IPs como input
```bash
# Salvar hosts vivos
nmap -sn 192.168.1.0/24 -oG - | grep "Up" | awk '{print $2}' > vivos.txt

# Usar como input no proximo scan
sudo nmap -sV -iL vivos.txt -oA scans/servicos
```

---

### Verbose no terminal + salvar arquivo simultaneamente
```bash
sudo nmap -sS -sV -v IP -oN resultado.txt
# -v mostra progresso em tempo real enquanto salva
```

---

## 🔄 Comparar Dois Scans com ndiff

Util para ver mudancas entre scans feitos em momentos diferentes:
```bash
ndiff scan_antes.xml scan_depois.xml
```

Mostra portas que abriram, fecharam ou mudaram de servico.

---

## 🛠️ Ferramentas que Consomem XML do Nmap

| Ferramenta | Como Importar |
|-----------|--------------|
| Metasploit | `db_import scan.xml` |
| Faraday | Import direto pela UI |
| Dradis | Upload de arquivo |
| nmap-parse-output | `nmap-parse-output scan.xml` |
| nmapviz | Gera grafico da rede |

---

## ⚠️ Erros Comuns

**Esquecer de salvar e precisar repetir o scan** — sempre `-oA` como habito.

**Sobrescrever scans anteriores** — incluir data/hora no nome do arquivo.

**Usar so `-oN`** — limita automacao; XML e grepable tem usos especificos essenciais.

---

## 📌 Relacionados

- [[Nmap - Visao Geral]]
- [[Nmap - Port Scanning]]
- [[Nmap Flags]]
- [[Recon Workflow]]

#ferramenta/nmap #cheatsheet
