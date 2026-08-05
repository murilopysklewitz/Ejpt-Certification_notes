# 🌍 DNS — Records & Funcionamento

> **DNS (Domain Name System)** é a agenda telefônica da internet.
> Traduz nomes de sites (`google.com`) em endereços IP (`142.250.78.14`).
> Sem DNS, você decoraria números em vez de nomes.

---

## 🔁 Como Funciona (Fluxo)

```
1. Você digita google.com no navegador
2. O sistema pergunta ao servidor DNS: "qual o IP de google.com?"
3. DNS responde: "142.250.78.14"
4. Navegador usa o IP para acessar o servidor correto
```

---

## 📋 Tipos de Records

| Record | Nome | Função | Exemplo |
|--------|------|--------|---------|
| **A** | Address | Domínio → IPv4 | `site.com → 192.168.1.1` |
| **AAAA** | Quad-A | Domínio → IPv6 | `site.com → 2001:db8::1` |
| **CNAME** | Canonical Name | Alias de domínio | `www.site.com → site.com` |
| **MX** | Mail Exchange | Servidor de email | `site.com → mail.site.com` |
| **NS** | Name Server | Quem gerencia o DNS | `site.com → ns1.provider.com` |
| **TXT** | Text | Informações extras | SPF, DKIM, verificação |
| **PTR** | Pointer | DNS reverso (IP → domínio) | `192.168.1.1 → site.com` |

---

## 🔎 Em Detalhes

### A Record
Associa um domínio a um IPv4. O mais básico e comum.
```
site.com.    IN    A    93.184.216.34
```

### CNAME
Cria um apelido. `www.site.com` pode apontar para `site.com`.
> ⚠️ CNAME não pode coexistir com outros records no mesmo nome.

### MX
Define qual servidor recebe emails. Tem prioridade (menor = maior prioridade).
```
site.com.    IN    MX    10    mail.site.com.
```

### TXT
Usado para verificação de domínio (Google, Microsoft), SPF (anti-spam), DKIM.
```
site.com.    IN    TXT    "v=spf1 include:sendgrid.net ~all"
```

### PTR
DNS reverso. Mapeia um IP de volta para um domínio. Usado em verificações de email e logs.

---

## 🔎 Relevância em Pentest

O DNS revela muito sobre a infraestrutura de um alvo:
- **A records** → IPs dos servidores
- **MX records** → provedor de email (pode revelar filtros anti-spam)
- **NS records** → quem gerencia o DNS (possível vetor de ataque)
- **TXT records** → tecnologias usadas
- **Subdomínios** → superfície de ataque expandida

---

## 🛠️ Ferramentas

```bash
# Consulta simples
host site.com
host -t MX site.com

# Recon DNS completo
dnsrecon -d site.com

# Subdomínios
dnsdumpster.com

# Zone Transfer (se mal configurado)
dig axfr @ns1.site.com site.com
fierce -dns site.com
dnsenum site.com
```

---

## ⚠️ Zone Transfer

Se o servidor DNS estiver mal configurado, pode revelar **todos os records de uma vez** (zone transfer / AXFR).
Isso é uma misconfiguration clássica e vaza toda a estrutura DNS do domínio.

```bash
dig axfr @IP -p porta
dnsenum dominio.com
fierce -dns dominio.com
```

---

## 📌 Relacionados

- [[DNSRecon]]
- [[DNSdumpster]]
- [[DNS Interrogation]]
- [[Network Layer — IP & ICMP]]
- [[MOC - Recon]]

#fundamentos #protocolo/dns #recon/passivo
