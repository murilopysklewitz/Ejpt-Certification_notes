# 🧪 Lab Report — Samba Recon com Nmap, Metasploit e Ferramentas NetBIOS

> **Plataforma:** INE
> **Tema central:** Enumeração completa de um servidor Samba — portas, versão, workgroup, NetBIOS e null session
> **Alvo:** `demo.ine.local`

---

## 🎯 Objetivo

Mapear completamente um servidor Samba sem credenciais, usando múltiplas ferramentas para cruzar e validar informações. Confirmar se null session (acesso anônimo) está habilitada.

---

## 🧠 Contexto Teórico — O Que é Samba / SMB / NetBIOS

Antes de executar qualquer comando, entender o que cada protocolo faz:

| Protocolo | Daemon | Função | Portas |
|-----------|--------|--------|--------|
| **SMB** (Server Message Block) | `smbd` | Compartilhamento de arquivos e impressoras | TCP 139, 445 |
| **NetBIOS** | `nmbd` | Resolução de nomes em rede local, registro de nome de máquina | UDP 137, 138 |
| **Samba** | — | Implementação open-source do SMB para Linux/Unix | Ambos acima |

> **SMB** é o protocolo de compartilhamento.
> **NetBIOS** é o sistema de nomes — funciona como um DNS local para redes Windows/Samba.
> **Samba** é o software que implementa tudo isso em Linux.

---

## 📋 Sumário de Etapas

| # | Pergunta / Objetivo | Ferramenta | Resultado |
|---|-------------------|-----------|-----------|
| 1 | Portas TCP padrão do smbd | `nmap` | 139, 445 |
| 2 | Portas UDP padrão do nmbd | `nmap -sU` | 137, 138 |
| 3 | Nome do Workgroup | `nmap -sV` | RECONLABS |
| 4 | Versão exata do Samba (NSE) | `smb-os-discovery.nse` | Samba 4.3.11-Ubuntu |
| 5 | Versão exata do Samba (MSF) | `smb_version` | Samba 4.3.11-Ubuntu |
| 6 | NetBIOS computer name (NSE) | `smb-os-discovery.nse` | SAMBA-RECON |
| 7 | NetBIOS computer name (nmblookup) | `nmblookup` | SAMBA-RECON |
| 8 | Null session via smbclient | `smbclient -L -N` | ✅ Permitida |
| 9 | Null session via rpcclient | `rpcclient -U "" -N` | ✅ Permitida |

---

## 🔬 Execução Passo a Passo

### Step 1 — Portas TCP do smbd

```bash
nmap demo.ine.local
```

**Por quê:** O scan padrão do Nmap cobre as top 1000 portas TCP. O SMB usa duas portas históricas:

| Porta | Motivo |
|-------|--------|
| **139/TCP** | SMB sobre NetBIOS Session Service — legado, ainda presente para compatibilidade |
| **445/TCP** | SMB direto sobre TCP/IP — versão moderna, sem depender do NetBIOS |

**Resultado:**
```
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
```

> 💡 Em ambientes modernos, porta 445 é a principal. Porta 139 existe por compatibilidade com sistemas antigos.

---

### Step 2 — Portas UDP do nmbd

```bash
nmap -sU --top-ports 25 demo.ine.local
```

**Por quê `-sU`:** UDP não tem handshake — o Nmap precisa de um scan específico para detectar serviços UDP. Por padrão o Nmap só faz TCP.

**Por quê `--top-ports 25`:** UDP scan é lento (pode demorar horas em todas as portas). Limitar às 25 mais comuns é o suficiente para encontrar NetBIOS rapidamente.

| Porta | Serviço | Função |
|-------|---------|--------|
| **137/UDP** | NetBIOS Name Service (NBNS) | Registro e resolução de nomes NetBIOS |
| **138/UDP** | NetBIOS Datagram Service | Broadcasts e comunicação sem conexão |

**Resultado:**
```
137/udp  open  netbios-ns
138/udp  open  netbios-dgm
```

> 💡 O `nmbd` é responsável por essas portas UDP. Ele é quem responde perguntas como "qual é o IP do computador chamado SAMBA-RECON na rede?".

---

### Step 3 — Nome do Workgroup

```bash
nmap -sV -p 445 demo.ine.local
```

**Por quê `-sV` revela o workgroup:** O Nmap faz um banner grab e negocia o protocolo SMB para extrair informações do serviço. O workgroup faz parte da apresentação inicial do servidor.

**O que é workgroup:** Agrupamento lógico de máquinas em uma rede Windows/Samba. Em ambientes corporativos, pode ser substituído por um domínio Active Directory. O nome do workgroup pode revelar o contexto da organização.

**Resultado:**
```
Workgroup: RECONLABS
```

---

### Step 4 — Versão Exata via Script NSE

```bash
nmap --script smb-os-discovery.nse -p 445 demo.ine.local
```

**Por quê um script NSE em vez de só `-sV`:** O `-sV` dá uma estimativa de versão. O script `smb-os-discovery` faz uma **negociação SMB completa** e extrai:
- Versão do sistema operacional
- Versão exata do Samba/Windows
- Nome do computador (NetBIOS)
- Nome do domínio/workgroup
- Horário do servidor

**Resultado:**
```
OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
Computer name: SAMBA-RECON
NetBIOS computer name: SAMBA-RECON
Workgroup: RECONLABS
```

> 💡 Versão exata → busca direta de CVEs. `Samba 4.3.11` tem vulnerabilidades conhecidas — inclusive o **SambaCry (CVE-2017-7494)**, que permite RCE.

---

### Step 5 — Versão Exata via Metasploit

```bash
msfconsole -q
use auxiliary/scanner/smb/smb_version
set RHOSTS demo.ine.local
exploit
```

**Por quê `-q`:** Flag "quiet" — inicia o msfconsole sem o banner ASCII. Mais rápido.

**Por quê confirmar com Metasploit se o Nmap já mostrou:** Duas ferramentas independentes confirmando a mesma versão = certeza. Ferramentas diferentes fazem a negociação SMB de formas ligeiramente diferentes — se ambas concordam, o dado é confiável.

**Resultado:**
```
[+] demo.ine.local - Host is running Samba 4.3.11-Ubuntu
```

---

### Step 6 — NetBIOS Computer Name via NSE

```bash
nmap --script smb-os-discovery.nse -p 445 demo.ine.local
```

O mesmo script do Step 4 já retorna o NetBIOS computer name junto.

**O que é NetBIOS computer name:** O "apelido" da máquina na rede local. Em redes Windows, as máquinas se registram com esse nome para que outras possam encontrá-las sem precisar de DNS. É diferente do hostname do sistema operacional (mas geralmente igual).

**Resultado:**
```
NetBIOS computer name: SAMBA-RECON
```

---

### Step 7 — NetBIOS Computer Name via nmblookup

```bash
nmblookup -A demo.ine.local
```

**O que é nmblookup:** Ferramenta que consulta diretamente o serviço NetBIOS Name Service (porta 137/UDP) do alvo. Faz a pergunta "qual é o nome registrado nesse IP?".

**Por quê usar além do Nmap:** `nmblookup` é uma consulta direta ao daemon `nmbd` — sem a camada de abstração do Nmap. Útil para confirmar o nome NetBIOS de forma independente e para redes onde o Nmap pode ser bloqueado mas UDP 137 está aberto.

**Resultado:**
```
192.x.x.x SAMBA-RECON<00>    - Workstation Service
192.x.x.x SAMBA-RECON<03>    - Messenger Service
192.x.x.x SAMBA-RECON<20>    - File Server Service
192.x.x.x RECONLABS<00>      - Workgroup
```

**Decodificando os sufixos NetBIOS:**

| Sufixo | Significado |
|--------|------------|
| `<00>` | Workstation / nome do computador |
| `<03>` | Messenger Service |
| `<20>` | File Server ativo |
| `<1e>` | Grupo de trabalho (workgroup) |

> 💡 Ver `<20>` confirmado é importante — significa que o serviço de **file sharing está ativo** naquele nome.

---

### Step 8 — Null Session via smbclient

```bash
smbclient -L demo.ine.local -N
```

**O que é null session (sessão nula):** Conexão ao SMB **sem fornecer usuário ou senha**. Em configurações antigas ou mal configuradas, o servidor aceita essa conexão e permite listar shares, enumerar usuários e grupos.

**Decodificando as flags:**

| Flag | Função |
|------|--------|
| `-L` | List — listar shares disponíveis |
| `-N` | No password — não solicitar senha (tentar null session) |

**Como interpretar o resultado:**
- Se **retornar lista de shares** → null session permitida ✅
- Se **retornar erro de autenticação** → null session bloqueada ❌

**Resultado:**
```
Sharename    Type    Comment
---------    ----    -------
print$       Disk    Printer Drivers
IPC$         IPC     IPC Service
```

Shares listadas sem senha → **null session permitida**.

> ⚠️ `IPC$` acessível via null session permite enumeração de usuários, grupos e políticas mesmo sem credenciais.

---

### Step 9 — Null Session via rpcclient

```bash
rpcclient -U "" -N demo.ine.local
```

**O que é rpcclient:** Ferramenta que se conecta via **RPC sobre SMB** (Remote Procedure Call). Enquanto o smbclient lista shares, o rpcclient permite executar comandos RPC no servidor — enumerar usuários, grupos, políticas de senha e mais.

**Decodificando as flags:**

| Flag | Função |
|------|--------|
| `-U ""` | Username vazio — tentativa de null session |
| `-N` | No password |

**Como interpretar o resultado:**
- Se abrir **prompt `rpcclient $>`** sem erro → null session permitida ✅
- Se retornar `NT_STATUS_LOGON_FAILURE` → bloqueada ❌

**Resultado:** Prompt `rpcclient $>` aberto sem credenciais.

**Comandos úteis dentro do rpcclient:**
```bash
srvinfo          # informações do servidor
enumdomusers     # listar usuários do domínio
enumdomgroups    # listar grupos
querydominfo     # política de senha e info do domínio
netshareenumall  # listar todos os shares
getdompwinfo     # política de complexidade de senha
```

---

## 📊 Resultado Final — Perfil Completo do Alvo

| Informação | Valor |
|-----------|-------|
| Portas TCP (smbd) | 139, 445 |
| Portas UDP (nmbd) | 137, 138 |
| Workgroup | RECONLABS |
| Versão do Samba | 4.3.11-Ubuntu |
| NetBIOS Name | SAMBA-RECON |
| Null session (smbclient) | ✅ Permitida |
| Null session (rpcclient) | ✅ Permitida |

---

## 🧠 Conceitos Consolidados

### Por Que Usar Múltiplas Ferramentas para a Mesma Info
Cada ferramenta faz a negociação SMB/NetBIOS de forma diferente. Cruzar resultados garante que você não perdeu nada por limitação de uma ferramenta específica. Também é uma prática de documentação — duas fontes independentes = dado confiável no relatório.

### Null Session — Por Que É Crítica
Null session é a chave que abre a enumeração profunda sem credenciais:
```
Null session habilitada
        ↓
smbclient → lista shares
rpcclient → enumera usuários, grupos, políticas
enum4linux → coleta tudo automaticamente
        ↓
Superfície de ataque completa sem nenhuma senha
```

### Samba 4.3.11 — Vulnerabilidades Relevantes
| CVE | Nome | Impacto |
|----|------|---------|
| CVE-2017-7494 | SambaCry | RCE sem autenticação via pipe de escrita |
| CVE-2015-0240 | — | Elevação de privilégio |

```bash
# Verificar SambaCry
nmap --script smb-vuln-cve-2017-7494 -p 445 demo.ine.local
```

---

## 🔁 Próximos Passos Lógicos

```
Null session confirmada
        ↓
enum4linux -a demo.ine.local     ← coleta tudo de uma vez
rpcclient → enumdomusers         ← listar usuários para brute force
        ↓
Usuários encontrados
        ↓
ftp_login / ssh_login / smb_login com wordlist
        ↓
Credencial válida → acesso autenticado
```

---

## 📌 Relacionados

- [[SMB — Enumeração e Comprometimento]]
- [[Nmap — NSE]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Cheatsheet — Portas Importantes]]
- [[FTP Enumeration com Metasploit]]

#lab #recon/ativo #protocolo/smb #ferramenta/nmap #ferramenta/metasploit #windows
