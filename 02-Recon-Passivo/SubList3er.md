# 🔍 SubList3er

> Ferramenta de enumeração de subdomínios via OSINT.
> Consulta múltiplos buscadores e fontes públicas simultaneamente.

---

## 🧠 O Que Faz

SubList3er agrega resultados de vários search engines (Google, Bing, Yahoo, Baidu, Ask, Netcraft, VirusTotal, ThreatCrowd, DNSdumpster, ReverseDNS) para encontrar subdomínios sem interagir diretamente com o alvo.

---

## 🔧 Comandos

```bash
# Básico
sublist3r -d dominio.com

# Especificar search engines
sublist3r -d dominio.com -e google,bing,yahoo

# Salvar resultado
sublist3r -d dominio.com -o subdomains.txt

# Modo verbose
sublist3r -d dominio.com -v

# Brute force adicional
sublist3r -d dominio.com -b

# Threads (velocidade)
sublist3r -d dominio.com -t 50
```

---

## ⚠️ Rate Limit do Google

O Google pode bloquear requisições automatizadas.

**Soluções:**
- Usar VPN e trocar quando bloquear
- Usar `-e` para excluir o Google temporariamente
- Usar `-t` menor para reduzir velocidade

```bash
# Pulando Google se estiver bloqueado
sublist3r -d dominio.com -e bing,yahoo,baidu
```

---

## 📋 Output Típico

```
[*] Searching in Google...
[*] Searching in Bing...
[-] Total Unique Subdomains Found: 47

admin.dominio.com
api.dominio.com
dev.dominio.com
mail.dominio.com
staging.dominio.com
vpn.dominio.com
```

---

## 🎯 O Que Fazer com os Subdomínios

1. Verificar quais estão ativos (`host subdominio.com`)
2. Buscar subdomínios de desenvolvimento/staging (menos protegidos)
3. Alimentar ferramentas de fingerprinting (`whatweb`, `nmap`)
4. Verificar certificados SSL (pode revelar mais subdomínios via SAN)

---

## 📌 Relacionados

- [[DNSdumpster]]
- [[theHarvester]]
- [[Google Dorks]]
- [[WhatWeb]]

#recon/passivo #ferramenta/osint #protocolo/dns
