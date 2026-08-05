# 🗺️ DNSdumpster

> Ferramenta online para mapeamento de subdomínios via DNS.
> Gera um mapa visual da infraestrutura DNS do alvo.

---

## 🧠 O Que Faz

DNSdumpster consulta múltiplas fontes públicas para encontrar subdomínios e gerar um mapa de infraestrutura DNS do domínio-alvo.

---

## 🔧 Como Usar

**Online (principal):**
```
https://dnsdumpster.com
```
Digite o domínio e recebe um relatório completo.

**Via terminal (tool alternativa):**
```bash
dnsdumpster dominio.com
```

---

## 📋 O Que Retorna

- Lista de subdomínios encontrados
- IPs associados a cada subdomínio
- Servidores MX e NS
- Mapa visual da infraestrutura (exportável)
- Histórico de registros DNS

---

## 🎯 O Que Procurar

| Achado | Relevância |
|--------|-----------|
| Subdomínios esquecidos | `dev.`, `staging.`, `old.`, `test.` |
| IPs diretos (sem CDN) | Bypass de proteção |
| Servidores de email | Possível phishing/spoofing |
| Infraestrutura interna exposta | `vpn.`, `internal.`, `admin.` |

> Subdomínios como `dev.site.com`, `staging.site.com` ou `admin.site.com` frequentemente têm segurança menor que o site principal.

---

## 🔁 Complementa

```
DNSdumpster (subdomínios)  +  SubList3er (mais fontes)  =  mapa completo
```

---

## 📌 Relacionados

- [[DNSRecon]]
- [[SubList3er]]
- [[DNS — Records & Funcionamento]]
- [[DNS Interrogation]]

#recon/passivo #ferramenta/dns #protocolo/dns
