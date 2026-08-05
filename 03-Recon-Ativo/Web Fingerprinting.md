# 🕵️ Web Fingerprinting

> Identificar as tecnologias usadas em um site/aplicação web.
> Quanto mais souber sobre o stack, mais preciso o ataque.

---

## 🧠 O Que É

Fingerprinting web é o processo de descobrir o servidor, linguagem, framework, CMS, bibliotecas e versões usadas por um alvo. Cada tecnologia tem potenciais vulnerabilidades associadas.

---

## 🛠️ Ferramentas

### WhatWeb (principal)
```bash
# Básico
whatweb dominio.com

# Verbose (mais detalhe)
whatweb -v dominio.com

# Múltiplos alvos
whatweb -i lista.txt

# Agressividade (1=passivo, 3=ativo)
whatweb -a 3 dominio.com
```

---

### Curl — Inspecionar headers manualmente
```bash
# Headers da resposta HTTP
curl -I dominio.com

# Com corpo e headers
curl -i dominio.com

# Seguir redirecionamentos
curl -iL dominio.com
```

---

### Arquivos Informativos do Site

**robots.txt** — revela diretórios que o admin não quer indexados:
```
https://dominio.com/robots.txt
```

**sitemap.xml** — mapa completo das páginas:
```
https://dominio.com/sitemap.xml
https://dominio.com/sitemap_index.xml
```

**WordPress específico:**
```
https://dominio.com/wp-includes/
https://dominio.com/wp-admin/
https://dominio.com/wp-login.php
```

---

## 📋 O Que Procurar

| Dado | Relevância |
|------|-----------|
| Versão do servidor (Apache 2.4.49) | Buscar CVEs específicos |
| CMS (WordPress, Joomla) | Plugins/temas vulneráveis |
| Linguagem (PHP, Python, Java) | Vetores de injeção específicos |
| Framework (Laravel, Django) | Vulnerabilidades de framework |
| Headers de segurança ausentes | CORS, CSP, X-Frame-Options |

---

## 🧠 Stack Comum e o Que Buscar

| Stack | O Que Verificar |
|-------|---------------|
| Apache + PHP | versão Apache, phpinfo exposto |
| Nginx + Node | versão nginx, endpoints da API |
| WordPress | versão WP, plugins desatualizados |
| IIS (Windows) | versão IIS, .NET framework |

---

## 🔁 HTTrack — Espelhar o Site

Para análise offline completa do código-fonte:
```bash
httrack https://dominio.com -O /pasta/saida
```
Baixa todo o HTML/CSS/JS para analisar localmente.

---

## 📌 Relacionados

- [[WhatWeb]]
- [[HTTrack]]
- [[Curl]]
- [[Wafw00f]]
- [[Google Dorks]]

#recon/ativo #ferramenta/web
