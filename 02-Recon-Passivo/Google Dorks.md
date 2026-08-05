# 🔎 Google Dorks

> Uso de operadores avançados do Google para encontrar informações específicas sobre um alvo.
> 100% passivo — você nunca toca no alvo diretamente.

---

## 🧠 O Que É

Google Dorking (ou Google Hacking) é a técnica de usar operadores de busca avançados para encontrar informações que não estão facilmente visíveis, mas estão publicamente indexadas.

---

## 🔧 Operadores Principais

### `site:` — limitar ao domínio
```
site:empresa.com
```
Mostra tudo indexado do domínio.

```
site:*.empresa.com
```
Mostra subdomínios indexados.

---

### `inurl:` — buscar na URL
```
inurl:admin site:empresa.com
inurl:login site:empresa.com
inurl:upload site:empresa.com
inurl:config site:empresa.com
```

---

### `intitle:` — buscar no título da página
```
intitle:admin site:empresa.com
intitle:"index of" site:empresa.com
intitle:"dashboard" site:empresa.com
```

---

### `filetype:` — buscar por tipo de arquivo
```
filetype:pdf site:empresa.com
filetype:xls site:empresa.com
filetype:doc site:empresa.com
filetype:sql site:empresa.com
filetype:env site:empresa.com
```

---

### `intitle:index of` — diretórios expostos
```
intitle:"index of" site:empresa.com
```
Revela servidores com listagem de diretório habilitada (misconfiguration).

---

## 📋 Dorks Úteis em Pentest

| Dork | O Que Encontra |
|------|---------------|
| `site:*.empresa.com` | Subdomínios públicos |
| `inurl:admin` | Painéis administrativos |
| `intitle:admin` | Páginas com "admin" no título |
| `filetype:pdf` | Documentos PDF (podem ter metadados) |
| `filetype:sql` | Backups de banco expostos |
| `filetype:env` | Arquivos .env expostos |
| `intitle:"index of"` | Diretórios abertos |
| `inurl:wp-admin` | Painéis WordPress |
| `inurl:phpMyAdmin` | Painéis de banco de dados |
| `"powered by" site:empresa.com` | Tecnologias usadas |

---

## 🗃️ Google Hacking Database (GHDB)

Repositório de dorks prontos, organizados por categoria:
```
https://www.exploit-db.com/google-hacking-database
```

Categorias:
- Footholds
- Files containing usernames
- Sensitive directories
- Web server detection
- Vulnerable files
- Error messages

---

## ⚠️ Limites e Ética

- Google pode bloquear buscas automatizadas (CAPTCHA)
- Use com moderação — é passivo mas deixa rastro nos logs do Google
- Nunca use para acessar sistemas sem autorização — o dork apenas encontra, acessar é outra coisa

---

## 📌 Relacionados

- [[theHarvester]]
- [[SubList3er]]
- [[NetCraft]]
- [[Web Fingerprinting]]

#recon/passivo #ferramenta/osint
