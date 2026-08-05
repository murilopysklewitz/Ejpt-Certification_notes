# 🌐 NetCraft

> Plataforma web para reconhecimento passivo.
> Faz praticamente tudo sobre um domínio sem você tocar no alvo diretamente.

---

## 🧠 O Que Faz

NetCraft é uma das fontes mais completas para recon passivo. Agrega dados históricos e atuais sobre infraestrutura web.

---

## 📋 O Que Revela

| Informação | Detalhes |
|-----------|---------|
| Tecnologias | Servidor web (Apache, nginx), linguagem, CMS |
| Histórico de hosting | IPs anteriores do domínio |
| Certificados SSL | Emissor, validade, SANs |
| Uptime e disponibilidade | Histórico de tempo online |
| Sistema operacional | Inferido pelo servidor |
| Registrar | Empresa de registro do domínio |
| Subdomínios | Subdomínios detectados |

---

## 🔧 Como Usar

Acesso via browser (sem linha de comando):
```
https://sitereport.netcraft.com/?url=dominio.com
```

Ou pelo aplicativo/extensão do browser.

---

## 🎯 O Que Procurar

- **IP real do servidor** → se estiver atrás de Cloudflare, o histórico pode revelar o IP original
- **Stack tecnológico** → PHP? WordPress? Versões antigas?
- **Mudanças de hosting** → rastro de infraestrutura anterior
- **Subdomínios** → mais superfície de ataque

---

## ⚡ Insight Importante

Domínios protegidos por CDN (Cloudflare, Akamai) mascaram o IP real.
O histórico de hosting do NetCraft pode mostrar o IP **antes** da CDN ser ativada — que pode ainda ser o IP real do servidor.

---

## 📌 Relacionados

- [[Whois]]
- [[WhatWeb]]
- [[Web Fingerprinting]]
- [[DNSdumpster]]
- [[MOC - Recon]]

#recon/passivo #ferramenta/osint
