# 🔍 Nessus — Scanner de Vulnerabilidades

> Ferramenta de vulnerability assessment mais usada no mercado.
> Enquanto o Nmap mapeia a rede, o Nessus analisa cada serviço em busca de vulnerabilidades conhecidas, misconfigurations e CVEs.

---

## 🧠 O Que é Nessus

**Nessus** é um scanner de vulnerabilidades desenvolvido pela **Tenable**. Ele automatiza a identificação de:

- Vulnerabilidades com CVE conhecidos
- Misconfigurations de sistema e serviço
- Senhas padrão e fracas
- Software desatualizado
- Problemas de conformidade (PCI-DSS, HIPAA, CIS)

**Diferença fundamental entre Nmap e Nessus:**

| | Nmap | Nessus |
|-|------|--------|
| Foco | Descoberta de hosts e portas | Análise de vulnerabilidades |
| Profundidade | Superfície (portas abertas, versões) | Profunda (exploração controlada, CVEs) |
| Output | Lista de serviços | Relatório de vulnerabilidades com severidade |
| Velocidade | Rápido | Lento (análise profunda) |
| Uso típico | Recon inicial | Vulnerability assessment |

---

## 📦 Versões do Nessus

| Versão | Custo | Limite | Uso |
|--------|-------|--------|-----|
| **Nessus Essentials** | Gratuito | 16 IPs | Home lab, estudos, certificações |
| **Nessus Professional** | Pago | Ilimitado | Pentest profissional |
| **Nessus Expert** | Pago | Ilimitado + cloud | Red team, cloud assessment |
| **Tenable.io** | SaaS | Ilimitado | Enterprise, gestão centralizada |

> 💡 Para estudos e certificações (eJPT, OSCP), o **Nessus Essentials** (gratuito) é suficiente.

---

## 🔧 Instalação no Kali Linux

```bash
# 1. Baixar o pacote (registrar em https://www.tenable.com/products/nessus/nessus-essentials)
# Baixar o .deb para Debian/Kali

# 2. Instalar
dpkg -i Nessus-*.deb

# 3. Iniciar o serviço
systemctl start nessusd

# 4. Verificar status
systemctl status nessusd

# 5. Acessar via browser
# https://localhost:8834
```

**Primeira vez:**
- Acessar `https://localhost:8834`
- Criar usuário admin
- Inserir código de ativação (recebido por email no registro)
- Aguardar atualização de plugins (pode demorar 30-60 minutos na primeira vez)

---

## 🗂️ Tipos de Scan (Policies)

### Basic Network Scan
Scan padrão de vulnerabilidades — o mais usado para avaliação geral.

```
Cobertura: Portas, serviços, CVEs, misconfigurations
Tempo: Médio
Uso: Assessment geral de um host ou rede
```

### Advanced Scan
Controle total sobre cada parâmetro do scan.

### Credentialed Scan
Scan com credenciais do sistema — acesso autenticado permite análise muito mais profunda.

```
Sem credencial: análise externa (o que é visível da rede)
Com credencial: análise interna (software instalado, patches, configs locais)

Diferença de cobertura: 30% → 90% de vulnerabilidades detectadas
```

### Web Application Tests
Focado em aplicações web — SQL Injection, XSS, misconfigurations HTTP.

### Malware Scan
Verificação de arquivos suspeitos e indicadores de comprometimento.

---

## 🔧 Criar e Executar um Scan

### Via Interface Web

```
1. Acessar https://localhost:8834
2. New Scan → Basic Network Scan (ou tipo desejado)
3. Configurar:
   - Name: identificação do scan
   - Targets: IP, range (192.168.1.0/24), hostname
4. Aba Credentials (opcional mas recomendado):
   - Add → Windows → SMB credentials
   - ou SSH para Linux
5. Save → Launch (▶)
6. Aguardar conclusão
7. Clicar no scan → ver vulnerabilidades
```

### Via Linha de Comando (nessuscli)
```bash
# Verificar scans disponíveis
/opt/nessus/sbin/nessuscli

# Exportar relatório
# (geralmente feito pela interface web)
```

---

## 📊 Interpretando o Output — Severidades

O Nessus classifica vulnerabilidades por severidade:

| Cor | Severidade | CVSS | O Que Fazer |
|-----|-----------|------|------------|
| 🔴 | **Critical** | 9.0 – 10.0 | Remediar imediatamente |
| 🟠 | **High** | 7.0 – 8.9 | Remediar urgente |
| 🟡 | **Medium** | 4.0 – 6.9 | Planejar remediação |
| 🔵 | **Low** | 0.1 – 3.9 | Remediar quando possível |
| ⚪ | **Info** | 0.0 | Informativo — sem ação urgente |

### Anatomia de um Finding

Para cada vulnerabilidade encontrada, o Nessus mostra:
```
Nome:        MS17-010: Security Update for Microsoft Windows SMB Server
Severidade:  Critical (9.3)
CVE:         CVE-2017-0143, CVE-2017-0144, CVE-2017-0145
Plugin ID:   97833
Host:        192.168.1.10:445

Synopsis:    The remote Windows host is affected by multiple vulnerabilities.
Description: The remote Windows host supports SMBv1 and is missing a patch...

Solution:    Microsoft has released a set of patches for Windows Vista, 2008,
             7, 2008 R2, 8.1, 2012, RT 8.1, 2012 R2, 10, and 2016.
             Apply KB4012212 immediately.

See Also:    https://docs.microsoft.com/en-us/security-updates/...
             https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143

Plugin Output:
  Installed version  : 6.1.7601.24441
  Fixed version      : 6.1.7601.24514
```

---

## 🔁 Workflow com Nessus em Pentest

```
1. Recon inicial com Nmap (hosts e portas)
        ↓
2. Nessus Essentials com IPs encontrados
   - Basic Network Scan → sem credencial
   - Credentialed Scan  → com credencial (maior profundidade)
        ↓
3. Analisar findings por severidade
   - Critical/High → testar exploração com Metasploit
   - Medium        → documentar para relatório
   - Info          → contexto adicional
        ↓
4. Cruzar CVEs do Nessus com módulos Metasploit
   searchsploit CVE-2017-0143
   use exploit/windows/smb/ms17_010_eternalblue
        ↓
5. Exportar relatório (PDF/HTML) para documentação
```

---

## 🔗 Nessus + Metasploit (Importar Resultados)

O Nessus pode exportar em formato `.nessus` (XML) que o Metasploit importa diretamente:

```bash
# No Nessus: Export → Nessus (formato .nessus)

# No msfconsole:
db_import /caminho/scan.nessus

# Ver o que foi importado
hosts
services
vulns        ← vulnerabilidades do Nessus aparecem aqui!
```

Isso integra o assessment do Nessus com a framework de exploração do Metasploit.

---

## ⚠️ Limitações do Nessus

| Limitação | Descrição |
|-----------|-----------|
| **Falsos positivos** | Pode reportar vulnerabilidades que não são exploráveis naquele contexto |
| **Falsos negativos** | Pode não detectar vulnerabilidades em versões muito novas ou muito antigas |
| **Scan sem credencial** | Detecta ~30% das vulnerabilidades reais — credencial muda drasticamente |
| **IDS/IPS** | Scans agressivos podem ser detectados e bloqueados |
| **Performance** | Scan completo pode demorar horas em redes grandes |

> 💡 **Nunca confiar 100% no Nessus.** Use como ponto de partida para investigação manual, não como conclusão definitiva.

---

## 🧠 Modelo Mental

```
Nessus não confirma que a vulnerabilidade é explorável —
apenas que o sistema parece afetado com base em versão/config.

Critical no Nessus
        ↓
Verificar manualmente com Nmap NSE
        ↓
Confirmar com Metasploit (scanner, não exploit)
        ↓
Explorar de forma controlada
        ↓
Documentar resultado real
```

---

## 📌 Relacionados

- [[Nmap — Visão Geral]]
- [[Nmap — Service & OS Detection]]
- [[Metasploit — Banco de Dados e Workspaces]]
- [[Top 10 Vulnerabilidades — Servicos Windows]]
- [[Wmap — Web Scanning com Metasploit]]

#ferramenta/nessus #vulnerability-assessment #recon/ativo
