# 🔎 DNSRecon

> Ferramenta para reconhecimento DNS completo.
> Revela todos os records DNS de um domínio de forma automatizada.

---

## 🧠 O Que Faz

DNSRecon consulta múltiplos tipos de records DNS e tenta enumerar subdomínios.
É mais completo que um simples `host` ou `dig`.

---

## 🔧 Comandos

```bash
# Recon padrão (todos os records)
dnsrecon -d dominio.com

# Especificar tipo de record
dnsrecon -d dominio.com -t std

# Brute force de subdomínios com wordlist
dnsrecon -d dominio.com -t brt -D /usr/share/wordlists/dnsmap.txt

# Zone Transfer
dnsrecon -d dominio.com -t axfr
```

---

## 📋 O Que Retorna

| Record | Informação |
|--------|-----------|
| A | IPs dos servidores |
| AAAA | IPs IPv6 |
| MX | Servidores de email |
| NS | Name servers |
| TXT | SPF, DKIM, verificações |
| SOA | Servidor autoritativo |
| PTR | Reverso (IP → domínio) |

---

## 🎯 O Que Procurar

- **Subdomínios** → cada um é uma superfície de ataque potencial
- **MX records** → provedor de email pode ter vulnerabilidades próprias
- **Zone Transfer** → se bem-sucedido, vaza toda a estrutura DNS

---

## ⚠️ Zone Transfer (AXFR)

Se o servidor NS estiver mal configurado, a tentativa de Zone Transfer pode retornar **todos os records** do domínio de uma vez. Isso é uma misconfiguration séria.

```bash
# Tentativa de zone transfer
dnsrecon -d dominio.com -t axfr

# Via dig
dig axfr @ns1.dominio.com dominio.com
```

---

## 📌 Relacionados

- [[DNS — Records & Funcionamento]]
- [[DNSdumpster]]
- [[DNS Interrogation]]
- [[Whois]]

#recon/passivo #ferramenta/dns #protocolo/dns
