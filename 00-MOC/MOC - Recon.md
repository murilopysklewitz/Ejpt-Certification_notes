# 🗺️ MOC — Reconhecimento

> **Objetivo:** Coletar o máximo de informações sobre o alvo antes de qualquer exploração.
> Regra de ouro: quanto mais informação na fase de recon, menor o risco na fase de ataque.

---

## 🔁 Fluxo

```
Passivo (sem contato) → Ativo (com contato) → Enumeração Detalhada
```

---

## 🕵️ Reconhecimento Passivo

Sem interagir diretamente com o alvo. Usa fontes públicas.

- [[Whois]] — registros do domínio
- [[NetCraft]] — tecnologias e histórico do site
- [[DNSRecon]] — registros DNS completos
- [[DNSdumpster]] — subdomínios via DNS
- [[SubList3er]] — subdomínios via OSINT
- [[theHarvester]] — emails, IPs, subdomínios via buscadores
- [[Google Dorks]] — pesquisas avançadas no Google
- [[Have I Been Pwned]] — emails/senhas vazados

---

## 🔎 Reconhecimento Ativo

Interage diretamente com o alvo. Pode gerar logs.

- [[Host Discovery]] — quem está vivo na rede
- [[Network Mapping]] — mapa completo da rede
- [[DNS Interrogation]] — consultas DNS diretas (zone transfer)
- [[Wafw00f]] — detecta WAF/Firewall do alvo
- [[Web Fingerprinting]] — tecnologias do site

---

## 🌐 Enumeração Web

- [[Dirb]] — enumera diretórios
- [[Dirsearch]] — enumera páginas e arquivos
- [[Gobuster]] — enumera com wordlist customizável
- [[WhatWeb]] — fingerprinting completo
- [[HTTrack]] — espelha o site inteiro
- [[Curl]] — requisições HTTP manuais

---

## 🔧 Ferramenta Central

- [[Nmap — Visão Geral]] — hub do Nmap

---

## 📌 Relacionados

- [[MOC - Network]] — fundamentos de rede
- [[Cheatsheet — Recon Workflow]] — comandos rápidos

#moc #recon
