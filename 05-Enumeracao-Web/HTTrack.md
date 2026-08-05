# 📥 HTTrack

> Baixa uma cópia completa de um site para análise offline.
> Útil para estudar o código-fonte, estrutura e links sem alertar o servidor repetidamente.

---

## 🧠 O Que Faz

HTTrack "espelha" um site — baixa todo o HTML, CSS, JavaScript, imagens e links para uma pasta local. Você pode navegar offline e analisar o código com calma.

---

## 🔧 Comandos

```bash
# Básico (pasta de saída = diretório atual)
httrack https://dominio.com

# Especificar pasta de saída
httrack https://dominio.com -O /home/user/site-espelho

# Limitar profundidade de links
httrack https://dominio.com -r3  # até 3 níveis

# Baixar apenas páginas do mesmo domínio
httrack https://dominio.com -%e0

# Com proxy
httrack https://dominio.com --proxy 127.0.0.1:8080
```

---

## 🎯 Por Que Usar

| Situação | Benefício |
|---------|----------|
| Análise de código-fonte | Ver HTML/JS sem DevTools repetidamente |
| Encontrar endpoints escondidos | Links não listados no sitemap |
| Analisar comentários no código | Desenvolvedores deixam comentários úteis |
| Mapeamento de estrutura | Hierarquia real de páginas |
| Buscar credenciais hardcoded | Chaves de API, senhas no JS |

---

## 💡 Dica

Após baixar o site, use `grep` para buscar informações sensíveis:

```bash
# Buscar credenciais hardcoded
grep -r "password" /pasta/site/
grep -r "api_key" /pasta/site/
grep -r "secret" /pasta/site/

# Buscar endpoints de API
grep -r "api/" /pasta/site/*.js
```

---

## ⚠️ Considerações

- Gera bastante tráfego no servidor — use com moderação
- Em ambientes de produção pode violar ToS
- Sempre use com autorização do alvo

---

## 📌 Relacionados

- [[WhatWeb]]
- [[Web Fingerprinting]]
- [[Gobuster]]
- [[Curl]]

#ferramenta/web #recon/ativo
