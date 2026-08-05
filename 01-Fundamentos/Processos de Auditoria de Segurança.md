# 🔎 Processos de Auditoria de Segurança

> Auditoria de segurança é o processo formal de avaliar controles, políticas e sistemas para identificar riscos e verificar conformidade.
> É o lado estruturado e documentado do que um pentester faz tecnicamente.

---

## 🧠 O Que é Auditoria de Segurança

Uma **auditoria de segurança** é uma avaliação sistemática e independente de sistemas, políticas, controles e procedimentos com o objetivo de:

- Identificar vulnerabilidades e riscos
- Verificar conformidade com normas e regulamentos
- Avaliar a eficácia dos controles de segurança existentes
- Gerar evidências documentadas para gestão e stakeholders

**Diferença entre Auditoria e Pentest:**

| | Auditoria de Segurança | Pentest |
|-|----------------------|---------|
| Foco | Conformidade + controles + processos | Exploração técnica de vulnerabilidades |
| Abordagem | Sistemática, baseada em normas | Orientada a vulnerabilidades |
| Escopo | Amplo (técnico + administrativo) | Geralmente técnico |
| Output | Relatório de conformidade + riscos | Relatório de vulnerabilidades + exploração |
| Frequência | Anual ou por exigência regulatória | Conforme necessidade ou contrato |
| Quem faz | Auditor interno ou externo | Pentester |

---

## 📋 Tipos de Auditoria de Segurança

### Auditoria Técnica
Foca em sistemas, infraestrutura e configurações.
- Análise de vulnerabilidades (Nessus, OpenVAS)
- Revisão de configurações de firewall, roteadores, switches
- Análise de hardening do sistema operacional
- Revisão de permissões e controle de acesso

### Auditoria de Conformidade
Verifica aderência a normas e regulamentos.
- ISO/IEC 27001
- PCI-DSS (cartões de pagamento)
- HIPAA (saúde nos EUA)
- LGPD (dados pessoais no Brasil)
- SOC 2

### Auditoria de Código
Revisão de segurança no nível de aplicação.
- Análise estática (SAST) — sem executar o código
- Análise dinâmica (DAST) — com execução
- Revisão manual de lógica de negócio

### Auditoria de Processos e Políticas
Avalia controles administrativos e organizacionais.
- Política de senha
- Processo de onboarding/offboarding
- Gestão de patches
- Plano de resposta a incidentes
- Segurança física

---

## 🔁 Fases de uma Auditoria de Segurança

### 1️⃣ Planejamento e Escopo

Definir o que será auditado, como e quando.

```
Documentos produzidos:
- Statement of Work (SoW)
- Termo de autorização assinado
- Definição de escopo (IPs, sistemas, aplicações)
- Metodologia adotada (OWASP, NIST, ISO 27001)
- Cronograma e janela de testes
- Contatos de emergência
```

**Perguntas respondidas nesta fase:**
- Quais sistemas estão no escopo?
- Quais estão fora do escopo?
- Haverá interrupção de serviços?
- Quem autoriza o trabalho?
- Como serão tratadas evidências?

---

### 2️⃣ Reconhecimento e Coleta de Informações

Levantamento técnico do ambiente auditado.

```bash
# Técnicas e ferramentas

# Descoberta de hosts
nmap -sn 192.168.1.0/24

# Scan de portas e serviços
nmap -sV -p- IP

# Versões e vulnerabilidades
nmap --script vuln IP
use auxiliary/scanner/smb/smb_version

# Enumeração de diretórios web
gobuster dir -u http://IP -w wordlist.txt

# DNS e subdomínios
dnsrecon -d dominio.com
sublist3r -d dominio.com

# Scan de vulnerabilidades
nessus / openvas
```

---

### 3️⃣ Análise de Vulnerabilidades

Identificar e classificar vulnerabilidades encontradas.

**Frameworks de classificação:**

| Framework | O Que Avalia |
|-----------|-------------|
| **CVSS** (Common Vulnerability Scoring System) | Score numérico 0-10 de severidade |
| **CVE** (Common Vulnerabilities and Exposures) | Identificador único de vulnerabilidade |
| **CWE** (Common Weakness Enumeration) | Categoria da fraqueza de software |
| **OWASP Top 10** | Top 10 vulnerabilidades web |

**Classificação CVSS:**

| Score | Severidade | Cor | Ação |
|-------|-----------|-----|------|
| 9.0 – 10.0 | Critical | 🔴 | Remediar imediatamente |
| 7.0 – 8.9 | High | 🟠 | Remediar urgente |
| 4.0 – 6.9 | Medium | 🟡 | Planejar remediação |
| 0.1 – 3.9 | Low | 🔵 | Remediar quando possível |
| 0.0 | Info | ⚪ | Informativo |

---

### 4️⃣ Exploração (se autorizada)

Em auditorias com componente de pentest, verificar se vulnerabilidades são exploráveis.

```
Modalidades de teste:

Black Box  → sem informação prévia do ambiente
Gray Box   → informações parciais (ex: credenciais)
White Box  → acesso completo à documentação e código
```

---

### 5️⃣ Relatório

O produto mais importante de uma auditoria — documentação estruturada de todos os achados.

**Estrutura padrão de relatório:**

```
1. Sumário Executivo
   - Visão de alto nível para gestão não-técnica
   - Principais riscos identificados
   - Nível geral de maturidade de segurança

2. Escopo e Metodologia
   - O que foi auditado
   - Como foi auditado
   - Período de execução

3. Achados (Findings)
   Para cada vulnerabilidade:
   - Título
   - Severidade (Critical/High/Medium/Low/Info)
   - Descrição técnica
   - Evidências (screenshots, logs, outputs)
   - Risco de negócio
   - Recomendação de remediação
   - Referências (CVE, CWE, OWASP)

4. Recomendações Gerais
   - Ações de curto, médio e longo prazo

5. Conclusão
```

---

### 6️⃣ Remediação e Reteste

```
Auditoria não termina com o relatório.

Ciclo:
Auditoria → Relatório → Remediação pelo cliente → Reteste → Fechamento
```

O **reteste** verifica se as vulnerabilidades foram corretamente corrigidas. Findings não corrigidos são re-reportados com atualização de status.

---

## 📐 Metodologias e Frameworks

### OWASP Testing Guide
Focado em aplicações web. Define metodologia para testar cada categoria de vulnerabilidade.
```
https://owasp.org/www-project-web-security-testing-guide/
```

### PTES (Penetration Testing Execution Standard)
Framework completo de pentest com fases bem definidas.
```
https://www.pentest-standard.org/
```

### NIST SP 800-115
Guia técnico do governo americano para testes de segurança.
```
https://csrc.nist.gov/publications/detail/sp/800-115/final
```

### ISO/IEC 27001
Norma internacional para gestão de segurança da informação.
Auditoria de conformidade verifica se a organização implementa os controles do **Anexo A**.

### CIS Controls
18 controles prioritários de segurança, organizados por implementação.
```
Nível 1: Básico (higiene de segurança)
Nível 2: Intermediário
Nível 3: Avançado (organizações maduras)
```

---

## 📝 Tipos de Achados (Findings)

### Vulnerabilidade Técnica
Falha explorável em sistema, serviço ou aplicação.
```
Exemplo: SMBv1 ativo → vulnerável ao MS17-010 (EternalBlue)
CVE: CVE-2017-0143
CVSS: 9.3 (Critical)
```

### Misconfiguration
Configuração incorreta que expõe risco.
```
Exemplos:
- Directory listing habilitado
- phpinfo() em produção
- .git exposto publicamente
- Conta padrão sem alteração de senha
```

### Política Ausente ou Inadequada
Controle administrativo faltando.
```
Exemplos:
- Sem política de senha documentada
- Sem processo de gestão de patches
- Sem plano de resposta a incidentes
- Sem revisão periódica de acessos
```

### Conformidade
Desvio de uma norma ou regulamento.
```
Exemplos:
- Dados de cartão armazenados sem criptografia (PCI-DSS)
- Log de auditoria insuficiente (ISO 27001)
- Sem aviso de uso aceitável (LGPD)
```

---

## 📄 Estrutura de um Finding Bem Documentado

```markdown
## Finding #01 — SMBv1 Ativo e Vulnerabilidade MS17-010

**Severidade:** Critical (CVSS 9.3)
**CVE:** CVE-2017-0143
**Host afetado:** 192.168.1.10 (WIN-SERVER-01)

### Descrição
O servidor possui o protocolo SMBv1 ativo, que contém uma vulnerabilidade
crítica de execução remota de código sem autenticação (MS17-010).
Um atacante com acesso à rede pode obter controle total do sistema
(NT AUTHORITY\SYSTEM) sem nenhuma credencial.

### Evidência
[Screenshot do output do Nmap]
nmap --script smb-vuln-ms17-010 -p445 192.168.1.10
| smb-vuln-ms17-010:
|   VULNERABLE:
|   State: VULNERABLE (Exploitable)

### Risco de Negócio
Comprometimento total do servidor, possível propagação para outros
sistemas na rede (wormable), extração de dados sensíveis.

### Recomendação
1. Aplicar patch KB4012212 imediatamente
2. Desabilitar SMBv1: Set-SmbServerConfiguration -EnableSMB1Protocol $false
3. Bloquear porta 445 entre segmentos de rede

### Referências
- https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
- https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
```

---

## 🧰 Ferramentas por Fase

| Fase | Ferramentas |
|------|------------|
| Reconhecimento | Nmap, theHarvester, Shodan, Maltego |
| Enumeração | enum4linux, smbclient, gobuster, dirb |
| Scan de vulnerabilidades | Nessus, OpenVAS, Nikto |
| Exploração | Metasploit, Burp Suite, sqlmap |
| Pós-exploração | Meterpreter, Mimikatz, BloodHound |
| Relatório | Dradis, Faraday, Plextrac, Word/PDF |

---

## 🧠 Modelo Mental — Auditoria vs Pentest

```
AUDITORIA DE SEGURANÇA
Pergunta: "Quão seguro estamos?"
Responde: Visão geral de conformidade, riscos e maturidade
Output: Relatório estruturado por severidade + recomendações

PENTEST
Pergunta: "Conseguimos comprometer o sistema?"
Responde: Demonstração técnica de exploração
Output: Relatório de vulnerabilidades exploradas + evidências

AMBOS:
- Requerem autorização formal por escrito
- Têm escopo definido
- Geram documentação
- Seguem metodologia
- Apresentam recomendações
```

---

## 📌 Relacionados

- [[Nessus — Scanner de Vulnerabilidades]]
- [[Nmap — Visão Geral]]
- [[Top 10 Vulnerabilidades — Servicos Windows]]
- [[Metasploit — Banco de Dados e Workspaces]]
- [[MOC - Recon]]

#conceito #auditoria #vulnerability-assessment #metodologia
