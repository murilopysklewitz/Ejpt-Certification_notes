# 🔍 Whois

> Consulta o registro público de um domínio ou IP.
> Revela informações sobre o proprietário, servidores DNS, datas de registro e contatos.

---

## 🧠 O que é

O Whois é um protocolo de consulta que acessa bases de dados públicas de registros de domínios e IPs.
Cada domínio registrado na internet tem um registro Whois associado.

---

## 🔧 Comandos

```bash
# Consulta de domínio
whois dominio.com

# Consulta de IP
whois 8.8.8.8
```

---

## 📋 O Que Você Encontra

| Campo | O Que Revela |
|-------|-------------|
| Registrant | Nome/empresa do dono |
| Registrant Email | Email de contato |
| Registrar | Empresa de registro (GoDaddy, Namecheap…) |
| Creation Date | Quando o domínio foi registrado |
| Expiry Date | Quando expira |
| Name Servers | Servidores DNS do domínio |
| Admin Contact | Contato administrativo |
| Tech Contact | Contato técnico |

---

## 🎯 O Que Procurar

- **Email do registrante** → útil para OSINT, theHarvester, phishing
- **Name Servers** → revela provedor de DNS (Cloudflare? Route53? próprio?)
- **Datas** → domínio novo pode indicar phishing; domínio antigo = mais legítimo
- **Registrante** → empresa real ou dados de privacidade?

> ⚠️ Muitos domínios hoje usam **WHOIS privacy** (proteção de privacidade), o que oculta dados reais do dono. Mas os Name Servers geralmente ficam visíveis.

---

## 🔁 Fluxo de Uso

```
whois dominio.com
    ↓
Anota Name Servers → alimenta DNSRecon
    ↓
Anota emails → alimenta theHarvester
    ↓
Anota empresa → pesquisa em LinkedIn, Google
```

---

## 📌 Relacionados

- [[DNSRecon]]
- [[theHarvester]]
- [[DNS — Records & Funcionamento]]
- [[MOC - Recon]]

#recon/passivo #ferramenta/osint
