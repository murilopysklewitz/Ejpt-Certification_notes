# 🌐 Curl

> Ferramenta de linha de comando para fazer requisições HTTP manualmente.
> Útil para inspecionar headers, testar endpoints e verificar respostas brutas.

---

## 🧠 O Que É

Curl (Client URL) faz requisições a qualquer URL via linha de comando. Essencial para explorar APIs, verificar cabeçalhos HTTP, testar autenticação e muito mais.

---

## 🔧 Comandos Essenciais

```bash
# Requisição GET básica
curl http://IP

# Ver headers + corpo (i = include headers)
curl -i http://IP

# Ver só os headers
curl -I http://IP

# Seguir redirecionamentos
curl -iL http://IP

# Arquivo específico
curl -i http://IP/admin/config.php

# POST com dados
curl -X POST http://IP/login \
  -d "username=admin&password=1234"

# POST com JSON
curl -X POST http://IP/api/login \
  -H "Content-Type: application/json" \
  -d '{"user":"admin","pass":"1234"}'

# Com cookie
curl http://IP -b "session=abc123"

# Com token de autenticação
curl http://IP -H "Authorization: Bearer TOKEN"

# Via proxy (Burp Suite)
curl http://IP --proxy http://127.0.0.1:8080

# Salvar resposta em arquivo
curl http://IP -o resposta.html

# Verbose (ver tudo)
curl -v http://IP
```

---

## 🎯 O Que Procurar nos Headers

```
HTTP/1.1 200 OK
Server: Apache/2.4.49             ← versão do servidor
X-Powered-By: PHP/7.4.3          ← linguagem + versão
Set-Cookie: PHPSESSID=...        ← tipo de sessão
X-Frame-Options: DENY            ← headers de segurança
Content-Security-Policy: ...     ← CSP configurado?
```

Ausência de headers de segurança = misconfiguration potencial.

---

## 🔁 Casos de Uso em Pentest

| Uso | Comando |
|-----|---------|
| Verificar resposta de porta | `curl -i http://IP:8080` |
| Testar injeção manual | `curl "http://IP/page?id=1'"` |
| Verificar métodos HTTP | `curl -X OPTIONS http://IP -i` |
| Testar CORS | `curl -H "Origin: evil.com" http://IP -i` |

---

## 📌 Relacionados

- [[Web Fingerprinting]]
- [[WhatWeb]]
- [[Dirsearch]]

#ferramenta/web #recon/ativo
