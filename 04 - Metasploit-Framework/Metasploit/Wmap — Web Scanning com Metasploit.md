# 🕸️ Wmap — Web Scanning com Metasploit

> Plugin integrado ao Metasploit que automatiza a enumeração de aplicações web usando os módulos auxiliares HTTP da framework.
> Une o poder dos módulos HTTP do MSF com o banco de dados centralizado do Metasploit.

---

## 🧠 O Que é Wmap

**Wmap** é um plugin de web scanning integrado ao **Metasploit Framework**. Ele usa os próprios módulos auxiliares HTTP do Metasploit (`auxiliary/scanner/http/*`) de forma coordenada para fazer um assessment de aplicação web, salvando tudo no banco de dados PostgreSQL do Metasploit.

**Diferença em relação a rodar módulos HTTP manualmente:**

| Abordagem | Forma |
|-----------|-------|
| Manual | Carregar um módulo por vez, configurar, rodar |
| Wmap | Coordena múltiplos módulos automaticamente em sequência |

**Diferença em relação a Nikto/dirb/dirsearch:**
O Wmap é nativo do Metasploit — os resultados vão direto para o banco de dados (`hosts`, `services`, `vulns`) e ficam disponíveis para os próximos passos de exploração.

---

## ⚙️ Pré-requisito: Banco de Dados Ativo

O Wmap salva tudo no banco do Metasploit — sem banco ativo, nada funciona.

```bash
# Iniciar banco de dados
msfdb init
msfdb start

# Iniciar msfconsole
msfconsole -q

# Verificar conexão com banco
msf> db_status
# [*] Connected to msf. Connection type: postgresql.
```

---

## 🔧 Configuração e Uso

### Passo 1 — Carregar o Plugin
```bash
msf> load wmap
# [*] WMAP loaded.
# [*] Successfully loaded plugin: wmap
```

### Passo 2 — Verificar Comandos Disponíveis
```bash
msf> wmap_
# Pressionar Tab para autocompletar e ver todos os comandos
```

| Comando | Função |
|---------|--------|
| `wmap_sites` | Gerenciar sites alvo |
| `wmap_targets` | Gerenciar targets específicos |
| `wmap_run` | Executar o scan |
| `wmap_vulns` | Listar vulnerabilidades encontradas |
| `wmap_modules` | Gerenciar módulos usados |
| `wmap_nodes` | Gerenciar nós distribuídos |

### Passo 3 — Adicionar o Site Alvo
```bash
# Adicionar site pelo IP ou hostname
msf> wmap_sites -a http://IP
msf> wmap_sites -a http://demo.ine.local

# Listar sites cadastrados
msf> wmap_sites -l
# [*] Available sites
# ===============
# Id  Host           Vhost           Port  Proto  # Pages  # Mods
# --  ----           -----           ----  -----  -------  ------
#  0  192.168.1.10   demo.ine.local  80    http   0        0
```

### Passo 4 — Definir o Target
```bash
# Usar o site cadastrado como target
msf> wmap_targets -t http://IP/
msf> wmap_targets -t http://demo.ine.local/

# Listar targets
msf> wmap_targets -l
```

### Passo 5 — Ver Módulos que Serão Executados
```bash
msf> wmap_run -t
# Mostra todos os módulos que serão executados
# [*] Testing target:
# [*] Site: demo.ine.local (192.168.1.10)
# [*] Port: 80 SSL: false
# ============================================================
# [*] Testing started. 2024-01-01 12:00:00 +0000
# [*]
# =[ SSL testing ]=
# ...
# =[ Web Server testing ]=
# auxiliary/scanner/http/http_version
# auxiliary/scanner/http/open_proxy
# auxiliary/scanner/http/options
# ...
# =[ Path & File testing ]=
# auxiliary/scanner/http/brute_dirs
# auxiliary/scanner/http/dir_listing
# auxiliary/scanner/http/dir_scanner
# auxiliary/scanner/http/files_dir
# ...
# =[ Unique Query testing ]=
# auxiliary/scanner/http/blind_sql_query
# auxiliary/scanner/http/error_sql_injection
# ...
```

### Passo 6 — Executar o Scan
```bash
# Executar scan completo
msf> wmap_run -e

# Aguardar conclusão — pode demorar vários minutos
```

### Passo 7 — Ver Resultados
```bash
# Vulnerabilidades encontradas pelo Wmap
msf> wmap_vulns -l

# Resultados também disponíveis no banco geral
msf> vulns
msf> hosts
msf> services
```

---

## 📋 Módulos Executados pelo Wmap

O Wmap organiza os módulos em categorias:

### SSL/TLS
```
auxiliary/scanner/http/ssl
auxiliary/scanner/http/cert
```

### Servidor Web
```
auxiliary/scanner/http/http_version     ← versão do servidor
auxiliary/scanner/http/options          ← métodos HTTP aceitos
auxiliary/scanner/http/open_proxy       ← proxy aberto
```

### Diretórios e Arquivos
```
auxiliary/scanner/http/brute_dirs       ← força bruta de diretórios
auxiliary/scanner/http/dir_listing      ← listagem aberta
auxiliary/scanner/http/dir_scanner      ← scanner com wordlist
auxiliary/scanner/http/files_dir        ← arquivos sensíveis
auxiliary/scanner/http/robots_txt       ← robots.txt
```

### Injeção
```
auxiliary/scanner/http/blind_sql_query  ← SQL injection cego
auxiliary/scanner/http/error_sql_injection ← SQL injection por erro
auxiliary/scanner/http/xss_injection    ← XSS
```

### Formulários e Autenticação
```
auxiliary/scanner/http/form_auth_bypass ← bypass de autenticação
auxiliary/scanner/http/http_login       ← brute force Basic Auth
```

---

## 🔁 Workflow Completo

```bash
# 1. Garantir banco ativo
msfdb start
msfconsole -q
db_status

# 2. Carregar plugin
load wmap

# 3. Adicionar alvo
wmap_sites -a http://IP
wmap_targets -t http://IP/

# 4. Revisar módulos (opcional)
wmap_run -t

# 5. Executar scan
wmap_run -e

# 6. Analisar resultados
wmap_vulns -l
vulns
services

# 7. Explorar findings manualmente
# Para cada vuln interessante:
use auxiliary/scanner/http/MODULO_ESPECIFICO
set RHOSTS IP
run
```

---

## 💡 Wmap vs Ferramentas Alternativas

| Ferramenta | Integração MSF | Velocidade | Cobertura Web |
|-----------|--------------|-----------|---------------|
| **Wmap** | ✅ Nativa | Lento | Média |
| **Nikto** | ❌ Externo | Rápido | Alta |
| **dirb** | ❌ Externo | Médio | Diretórios |
| **dirsearch** | ❌ Externo | Rápido | Diretórios + arquivos |
| **Gobuster** | ❌ Externo | Muito rápido | Diretórios |

**Quando usar Wmap:**
- Quando já está dentro do Metasploit e quer centralizar tudo no banco
- Quando quer ter os resultados diretamente em `vulns` para cruzar com exploits

**Quando usar alternativas:**
- Para enumeração de diretórios rápida → Gobuster/dirsearch
- Para assessment web completo → Nikto

---

## ⚠️ Limitações

- **Lento** — coordena muitos módulos em sequência
- **Gera ruído** — muitas requisições para o servidor alvo
- **Menos atualizado** que ferramentas dedicadas como Nikto
- **Requer banco ativo** — sem PostgreSQL não funciona

---

## 📌 Relacionados

- [[Metasploit — Banco de Dados e Workspaces]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[HTTP Apache Enumeration com Metasploit]]
- [[Dirb]]
- [[Gobuster]]
- [[Nessus — Scanner de Vulnerabilidades]]

#ferramenta/metasploit #recon/ativo #protocolo/http #web-scanning
