# 📘 eJPT Certification Notes

> Anotações de estudo produzidas durante a preparação para a certificação **eJPT (eLearnSecurity Junior Penetration Tester)** — aprovado ✅.
> Organizadas por metodologia de pentest: da fundamentação teórica até laboratórios completos de exploração e pós-exploração.

## 🚀 Comece por aqui

Se você também tá estudando pra eJPT e caiu aqui perdido sem saber por onde começar, esses arquivos concentram o essencial antes de mergulhar em cada pasta específica:

| Arquivo | Por que começar por ele |
|---|---|
| [`00-MOC/MOC - Recon.md`](00-MOC/MOC%20-%20Recon.md) | Mapa geral do fluxo de reconhecimento — passivo → ativo → enumeração. Mostra onde cada ferramenta do repo se encaixa antes de você entrar em qualquer pasta específica |
| [`00-MOC/MOC - Network.md`](00-MOC/MOC%20-%20Network.md) | Base teórica de rede (OSI, TCP/UDP, IP) conectada direto com técnica de ataque por camada — útil pra entender o "porquê" por trás dos comandos |
| [`04 - Metasploit-Framework/Cheatsheets/Recon Workflow.md`](04%20-%20Metasploit-Framework/Cheatsheets/Recon%20Workflow.md) | Workflow de recon condensado — bom pra ter aberto do lado enquanto pratica ou durante a prova |
| [`04 - Metasploit-Framework/Cheatsheets/Portas Importantes.md`](04%20-%20Metasploit-Framework/Cheatsheets/Portas%20Importantes.md) | Referência rápida de portas e serviços — economiza tempo de lookup na hora do exame |
| [`04 - Metasploit-Framework/Cheatsheets/Nmap Flags.md`](04%20-%20Metasploit-Framework/Cheatsheets/Nmap%20Flags.md) | Flags de Nmap mais usadas, sem precisar abrir a nota completa de cada técnica |
| [`12-Post-Exploitation/Manual_de_Metodologia.md`](12-Post-Exploitation/Manual_de_Metodologia.md) | Checklist consolidado de pós-exploração — o que verificar assim que você ganha acesso a um host |
| [`13-Labs/`](13-Labs) | Depois da teoria, os labs mostram a metodologia aplicada de ponta a ponta (enumeração → exploração → pós-exploração) em cenários reais de prova |

**Ordem sugerida de leitura:** MOCs → Fundamentos → Recon (passivo/ativo) → Enumeração Web → Metasploit → Exploração → Pós-Exploração → Labs.

---

## 🗂️ Estrutura

| Pasta | Conteúdo |
|---|---|
| `00-MOC` | Mapas de conteúdo (Maps of Content) — visão geral de Network e Recon |
| `01-Fundamentos` | OSI, TCP/UDP, DNS, ICMP, auditoria de segurança, top 10 vulnerabilidades em serviços Windows |
| `02-Recon-Passivo` | OSINT: Whois, theHarvester, DNSdumpster, DNSRecon, Google Dorks, Have I Been Pwned, NetCraft, SubList3er |
| `03-Recon-Ativo` | Host discovery, network mapping, DNS interrogation, web fingerprinting, Wafw00f, Nessus |
| `04 - Metasploit-Framework` | Nmap (scanning, evasão, NSE, output formats), Metasploit (fundamentos, workspaces, auxiliary modules, MSFVenom), cheatsheets |
| `05-Enumeracao-Web` | Gobuster, Dirb, Dirsearch, Curl, HTTrack, WhatWeb, Hydra, enum4linux |
| `06-Defesas-Evasao` | Firewall, IDS/IPS |
| `07-Protocolos` | FTP, SMB, IIS |
| `08-Exploracao` | Exploits e técnicas: EternalBlue, BlueKeep, Shellshock, SMB Relay, WebDAV, SNMP, MSSQL, payloads com msfvenom, entre outros |
| `12-Post-Exploitation` | Pós-exploração Windows e Linux — credential attacks, privilege escalation, persistence, lateral movement, pivoting, defense evasion |
| `13-Labs` | Walkthroughs completos de laboratórios (enumeração → exploração → pós-exploração) |

---

## 🎯 Sobre a eJPT

A **eJPT** é uma certificação prática de entrada em pentest, focada em:
- Reconhecimento (passivo e ativo)
- Varredura e enumeração de serviços
- Exploração de vulnerabilidades comuns
- Pós-exploração e movimentação lateral
- Uso de ferramentas como Nmap, Metasploit, Hydra, Burp Suite

Este repositório segue essa mesma progressão, funcionando tanto como **material de estudo** quanto como **referência rápida** (cheatsheets e comandos) para engajamentos de pentest.

---

## 🛠️ Como usar

- As pastas são numeradas seguindo a ordem lógica de uma metodologia de pentest (recon → enumeração → exploração → pós-exploração).
- `13-Labs` contém os laboratórios práticos completos, com comandos e resultados reais de cada etapa — bom ponto de partida para quem quer ver a metodologia aplicada de ponta a ponta.
- `04 - Metasploit-Framework/Cheatsheets` reúne comandos de referência rápida (flags de Nmap, portas importantes, workflow de recon).

---

## ⚠️ Aviso

Todo o conteúdo aqui é para fins **educacionais**, produzido em ambientes de laboratório controlados (INE/eLearnSecurity). Não utilize essas técnicas contra sistemas sem autorização explícita.

---

## 📫 Contato

Feito por [Murilo](https://github.com/murilopysklewitz) — dúvidas, sugestões ou correções são bem-vindas via issues ou pull request.