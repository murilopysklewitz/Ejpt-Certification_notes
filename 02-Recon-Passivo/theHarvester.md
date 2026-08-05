# 🌾 theHarvester

> Ferramenta OSINT para coletar emails, subdomínios, IPs, nomes e mais.
> Consulta múltiplas fontes públicas e buscadores de uma vez.

---

## 🧠 O Que Faz

theHarvester é uma das ferramentas OSINT mais completas. Agrega dados de várias fontes para construir um perfil do alvo sem interagir diretamente com ele.

---

## 🔧 Comandos

```bash
# Básico
theHarvester -d dominio.com -b all

# Especificar fonte
theHarvester -d dominio.com -b google

# Limitar resultados
theHarvester -d dominio.com -b bing -l 100

# Salvar output
theHarvester -d dominio.com -b all -f resultado

# Ver todas as opções
theHarvester --help
```

---

## 📡 Fontes Disponíveis

```
google, bing, yahoo, baidu, duckduckgo,
linkedin, twitter, github, shodan,
virustotal, crtsh, dnsdumpster,
hackertarget, rapiddns, ...
```

Use `-b all` para consultar todas ao mesmo tempo.

---

## 📋 O Que Encontra

| Dado | Uso em Pentest |
|------|---------------|
| **Emails** | Phishing, credential stuffing, LinkedIn recon |
| **Subdomínios** | Superfície de ataque expandida |
| **IPs** | Mapeamento de infraestrutura |
| **Nomes de funcionários** | Engenharia social |
| **Hosts virtuais** | Múltiplos sites no mesmo servidor |

---

## 🎯 Workflow com theHarvester

```
theHarvester -d dominio.com -b all
    ↓
Emails encontrados → Have I Been Pwned
    ↓
Subdomínios → SubList3er, DNSdumpster
    ↓
Nomes → LinkedIn, redes sociais
    ↓
IPs → Nmap, Shodan
```

---

## 💡 Dica Prática

theHarvester é especialmente útil para encontrar **padrões de email** da empresa.
Se você encontra `joao.silva@empresa.com`, pode inferir que outros emails seguem o mesmo padrão: `nome.sobrenome@empresa.com`.

---

## 📌 Relacionados

- [[SubList3er]]
- [[Google Dorks]]
- [[Have I Been Pwned]]
- [[Whois]]

#recon/passivo #ferramenta/osint
