# 🛡️ Wafw00f

> Detecta a presença e o tipo de **Web Application Firewall (WAF)** em um domínio.
> Antes de tentar qualquer coisa, saber se há WAF muda toda a estratégia.

---

## 🧠 O Que Faz

Wafw00f envia requisições HTTP específicas e analisa as respostas para identificar se existe um WAF e qual é (Cloudflare, Akamai, AWS WAF, ModSecurity, etc.).

---

## 🔧 Comandos

```bash
# Detecção básica
wafw00f dominio.com

# Com verbose
wafw00f -v dominio.com

# Testar todos os WAFs conhecidos
wafw00f -a dominio.com

# Via HTTPS
wafw00f https://dominio.com
```

---

## 📋 Output Típico

```
[*] Checking https://dominio.com
[+] The site https://dominio.com is behind Cloudflare (Cloudflare Inc.) WAF.
[~] Number of requests: 2
```

ou:

```
[*] Checking https://dominio.com
[~] Generic Detection results:
[-] No WAF detected by the generic detection
```

---

## 🎯 Por Que Isso Importa

| WAF Detectado | Implicação |
|--------------|-----------|
| Cloudflare | IP real mascarado, tráfego filtrado |
| ModSecurity | Regras OWASP ativas |
| AWS WAF | Regras customizáveis por cliente |
| Nenhum | Acesso mais direto ao servidor |

Saber o WAF ajuda a:
- Escolher técnicas de evasão adequadas
- Entender quais payloads podem ser bloqueados
- Ajustar timing e encoding de ataques

---

## 🔁 Posição no Workflow

```
Whois → DNSRecon → SubList3er
        ↓
    Wafw00f  ← aqui você decide a estratégia
        ↓
Fingerprinting + Enumeração (ajustada ao WAF)
```

---

## 📌 Relacionados

- [[Firewall — Conceito]]
- [[IDS & IPS]]
- [[Técnicas de Evasão — Nmap]]
- [[Web Fingerprinting]]

#recon/ativo #ferramenta/web #evasao
