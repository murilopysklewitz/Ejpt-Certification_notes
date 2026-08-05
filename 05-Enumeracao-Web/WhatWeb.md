# 🕵️ WhatWeb

> Identifica tecnologias de um site: servidor, CMS, framework, linguagem, versões.
> O fingerprinting completo em um comando.

---

## 🧠 O Que Faz

WhatWeb analisa cabeçalhos HTTP, cookies, HTML e JavaScript para identificar automaticamente o stack tecnológico de qualquer site.

---

## 🔧 Comandos

```bash
# Básico
whatweb dominio.com

# Verbose (mais detalhe)
whatweb -v dominio.com

# Muito verbose (máximo detalhe)
whatweb -v -v dominio.com

# Nível de agressividade (1-4)
whatweb -a 3 dominio.com  # mais intrusivo, mais info

# Múltiplos alvos
whatweb dominio1.com dominio2.com

# A partir de arquivo
whatweb -i alvos.txt

# Proxy
whatweb --proxy 127.0.0.1:8080 dominio.com
```

---

## 📋 O Que Detecta

| Categoria | Exemplos |
|-----------|---------|
| Servidor web | Apache, nginx, IIS |
| Linguagem | PHP, Python, Ruby, Java |
| CMS | WordPress, Joomla, Drupal |
| Framework | Laravel, Django, Rails |
| Bibliotecas JS | jQuery, Bootstrap, React |
| Analytics | Google Analytics, Hotjar |
| CDN | Cloudflare, Akamai |

---

## 🔁 Fluxo de Uso

```
whatweb dominio.com
    ↓
WordPress 5.8 detectado
    ↓
Procurar CVEs do WordPress 5.8
    ↓
Plugins detectados → procurar CVEs dos plugins
    ↓
Definir vetor de ataque
```

---

## 📌 Relacionados

- [[Web Fingerprinting]]
- [[Dirb]]
- [[Gobuster]]
- [[NetCraft]]

#ferramenta/web #recon/ativo
