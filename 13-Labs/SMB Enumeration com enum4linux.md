# 🧪 Lab Report — SMB Enumeration com enum4linux + smbclient

> **Plataforma:** INE
> **Tema central:** Enumeração SMB com enum4linux → null session → acesso a share via smbclient → extração de arquivo
> **Alvo:** `target.ine.local`

---

## 🎯 Objetivo

Usar o enum4linux para enumerar completamente um servidor Samba via null session, identificar shares acessíveis anonimamente e extrair arquivos via smbclient.

---

## 📋 Sumário de Etapas

| # | Ação | Ferramenta | Resultado |
|---|------|-----------|-----------|
| 1 | Enumeração completa | `enum4linux -a` | Workgroup, null session, share `pubfiles` |
| 2 | Conectar ao share | `smbclient` | Acesso anônimo confirmado |
| 3 | Listar e baixar arquivo | `ls` + `get` | `FLAG1{68b7a86afb254645bd9af958d01bc96f}` |

---

## 🔬 Execução Passo a Passo

### Step 1 — Enumeração Completa com enum4linux

```bash
enum4linux -a target.ine.local
```

**O que é enum4linux:** Wrapper que automatiza múltiplas consultas SMB/RPC usando `smbclient`, `rpcclient`, `net` e `nmblookup` de uma só vez. É o "faz-tudo" de enumeração Samba.

**A flag `-a` significa "all"** — executa todas as verificações disponíveis:

| Verificação | Flag isolada | O Que Faz |
|------------|-------------|-----------|
| Workgroup/Domain | `-w` | Nome do grupo de trabalho |
| Session check | `-s` | Testa null session |
| Domain SID | `-i` | Identifica se é domínio ou workgroup |
| Users via RID cycling | `-r` | Enumera usuários por RID |
| Share brute force | `-s` | Shares acessíveis por null session |
| Group membership | `-g` | Grupos e membros |
| Password policy | `-p` | Lockout, complexidade, expiração |
| OS information | `-o` | Sistema operacional |

---

**Output relevante analisado:**

#### Null Session Confirmada
```
[+] Server target.ine.local allows sessions using username '', password ''
```
O servidor aceita conexão sem credencial. Isso habilita toda a enumeração seguinte.

#### Workgroup Identificado
```
[+] Got domain/workgroup name: WORKGROUP
Domain Sid: (NULL SID)
```
`NULL SID` confirma que é um **workgroup standalone** — não é um Domain Controller do Active Directory. Máquina isolada com usuários locais.

#### Share Encontrado via Brute Force
```
pubfiles EXISTS, Allows access using username: '', password: ''
```
O share `pubfiles` existe **e** aceita conexão anônima. Dois achados em um: o share existe e não exige senha.

---

### Step 2 — Conectar ao Share com smbclient

#### ⚠️ Erros de Sintaxe (documentados para aprendizado)

O smbclient tem uma sintaxe específica que causa confusão. Os erros abaixo foram cometidos no lab:

```bash
# ❌ ERRADO — flag minúscula -u não existe
smbclient target.ine.local/pubfiles -u ""

# ❌ ERRADO — barra simples não funciona, falta o escape
smbclient target.ine.local/pubfiles -U ""

# ❌ ERRADO — só uma barra invertida — precisa de duas (escape no bash)
smbclient \\target.ine.local/pubfiles

# ✅ CORRETO — duas barras duplas (cada \\ = um \ real no bash)
smbclient //target.ine.local/pubfiles
```

**Por que a sintaxe com `\\\\` ou `//`:**

O caminho UNC (Universal Naming Convention) do Windows usa `\\servidor\share`. No bash, `\` precisa ser escapado — então cada `\` vira `\\`. Por isso `\\\\servidor\\share`.

A alternativa é usar a notação Unix com barras normais: `//servidor/share`. Mais simples e funciona igualmente.

**Referência rápida de sintaxe:**

```bash
# Listar shares (com credencial)
smbclient -L \\\\IP\\ -U usuario
smbclient -L //IP -U usuario

# Conectar em share (sem credencial — null session)
smbclient //IP/share -N
smbclient //IP/share              # pressiona Enter na senha

# Conectar em share (com credencial)
smbclient //IP/share -U usuario
smbclient //IP/share -U usuario%senha   # senha inline
```

---

#### Conexão Bem-Sucedida

```bash
smbclient //target.ine.local/pubfiles
# Password: [Enter — sem senha]
```

O smbclient solicitou senha e ao pressionar Enter (sem digitar nada), a conexão foi aceita — confirmando null session.

---

### Step 3 — Listar e Baixar o Arquivo

```bash
smb: \> ls
smb: \> get flag1.txt
smb: \> exit
```

```bash
cat flag1.txt
```

**Resultado:**
```
FLAG1{68b7a86afb254645bd9af958d01bc96f}
```

---

## 📊 Resultado Final

| Informação | Valor |
|-----------|-------|
| Workgroup | WORKGROUP |
| Tipo de host | Standalone (não é DC) |
| Null session | ✅ Permitida |
| Share acessível | `pubfiles` (anônimo) |
| Arquivo extraído | `flag1.txt` |
| Flag | `FLAG1{68b7a86afb254645bd9af958d01bc96f}` |

---

## 🧠 Conceitos Consolidados

### enum4linux vs Ferramentas Individuais

| Tarefa | Manualmente | Com enum4linux |
|--------|------------|----------------|
| Verificar null session | `rpcclient -U "" -N IP` | `-a` inclui tudo |
| Listar shares | `smbclient -L //IP -N` | `-a` inclui tudo |
| Enumerar usuários | `rpcclient → enumdomusers` | `-a` inclui tudo |
| Política de senha | `rpcclient → getdompwinfo` | `-a` inclui tudo |
| RID cycling | Loop manual com rpcclient | `-r` automático |

`enum4linux -a` é o atalho para tudo isso de uma vez. Mas entender o que cada verificação faz individualmente é importante para interpretar o output e reproduzir manualmente quando necessário.

### Sintaxe smbclient — Regra de Ouro

```
Use sempre // (barras normais Unix):

smbclient //IP/share          # sem credencial
smbclient //IP/share -U user  # com usuário
smbclient //IP/share -N       # forçar sem senha
```

Se precisar usar backslash (ambiente Windows ou scripts):
```
smbclient \\\\IP\\share       # cada \\ = um \ no bash
```

### NULL SID — O Que Significa

`Domain SID: (NULL SID)` indica que a máquina não faz parte de um domínio Active Directory. É um workgroup standalone. Isso significa:
- Sem Kerberos
- Sem políticas de domínio centralizadas
- Usuários e grupos são todos locais
- Mais simples de enumerar e atacar

### RID Cycling — Como enum4linux Descobre Usuários

O enum4linux usa um range de RIDs (500-550, 1000-1050) para descobrir usuários mesmo sem permissão explícita de enumerar:

```
RID 500  → Administrator (sempre)
RID 501  → Guest (sempre)
RID 1000 → primeiro usuário criado
RID 1001 → segundo usuário criado
...
```

Consultando cada RID, ele descobre os nomes das contas sem precisar de `enumdomusers`.

---

## ⚠️ Pontos de Atenção — Ambiente Real

| Achado | Risco | Recomendação |
|--------|-------|-------------|
| Null session habilitada | 🔴 Alto | Desabilitar `restrict anonymous = 2` no smb.conf |
| Share `pubfiles` sem auth | 🔴 Alto | Exigir autenticação ou remover share |
| Arquivo sensível em share público | 🔴 Crítico | Revisar conteúdo de shares públicos |

---

## 🔁 Próximos Passos Lógicos

```
Null session confirmada
        ↓
enum4linux -a → usuários encontrados
        ↓
Lista de usuários → brute force SMB
use auxiliary/scanner/smb/smb_login
        ↓
Credencial válida → shares privados acessíveis
smbclient //IP/usuario -U usuario
        ↓
rpcclient → enumdomusers → getdompwinfo
        ↓
Política de lockout? Se não → brute force SSH/FTP
```

---

## 📌 Relacionados

- [[SMB — Server Message Block]]
- [[Samba Recon Completo]]
- [[SMB Brute Force e Acesso a Shares]]
- [[SMB — Enumeração e Comprometimento]]
- [[Nmap — NSE]]

#lab #recon/ativo #protocolo/smb #ferramenta/enum4linux #null-session
