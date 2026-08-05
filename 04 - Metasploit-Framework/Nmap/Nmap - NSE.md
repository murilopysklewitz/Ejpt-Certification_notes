# 📜 Nmap — NSE (Nmap Scripting Engine)

> O NSE transforma o Nmap de scanner de portas em **plataforma de enumeração ativa**.
> Sem NSE: você descobre portas. Com NSE: você começa a interagir com serviços.

---

## 🧠 O Que É

Sistema de **scripts em Lua** integrado ao Nmap. Os scripts podem:
- Coletar informações detalhadas
- Detectar vulnerabilidades conhecidas
- Testar autenticação (brute force leve)
- Enumerar usuários, shares, configurações
- Identificar misconfigurations

---

## 📁 Onde Ficam os Scripts

```bash
# Diretório dos scripts
ls /usr/share/nmap/scripts/

# Buscar por palavra-chave
ls /usr/share/nmap/scripts/ | grep -e "ftp"
ls /usr/share/nmap/scripts/ | grep -e "smb"
ls /usr/share/nmap/scripts/ | grep -e "http"

# Ver detalhes de um script
nmap --script-help=http-title
nmap --script-help=ftp-anon
```

Cada script termina com `.nse`

---

## 🔧 Comandos

```bash
# Scripts padrão (safe + discovery)
sudo nmap -sC IP

# Script específico
sudo nmap --script=http-title IP

# Múltiplos scripts
sudo nmap --script=http-title,http-headers IP

# Categoria inteira
sudo nmap --script=vuln IP
sudo nmap --script=auth IP

# Curinga
sudo nmap --script=http-* IP

# Combinação poderosa
sudo nmap -sV --script=vuln IP
sudo nmap -sV --script=default,vuln IP
```

---

## 📂 Categorias de Scripts

| Categoria | Descrição | Risco |
|-----------|-----------|-------|
| `default` | Scripts padrão seguros | Baixo |
| `safe` | Não causa impacto | Mínimo |
| `discovery` | Coleta informações | Baixo |
| `version` | Ajuda na detecção de versão | Baixo |
| `auth` | Testa autenticação | Médio |
| `vuln` | Detecta vulnerabilidades conhecidas | Médio |
| `brute` | Tentativas de login | Alto |
| `exploit` | Exploração básica | Alto |
| `malware` | Detecta backdoors | Médio |

---

## 🔥 Scripts Mais Usados

### Detecção de Vulnerabilidades
```bash
sudo nmap -sV --script=vuln IP
```
Retorna CVEs e vulnerabilidades conhecidas dos serviços.

---

### FTP — Login Anônimo
```bash
sudo nmap --script=ftp-anon -p21 IP
```
Verifica se FTP aceita login `anonymous`.

---

### SMB — Enumeração
```bash
sudo nmap --script=smb-enum-shares -p445 IP
sudo nmap --script=smb-os-discovery -p445 IP
sudo nmap --script=smb-vuln-ms17-010 -p445 IP  # EternalBlue
```

---

### HTTP — Informações Web
```bash
sudo nmap --script=http-title -p80 IP
sudo nmap --script=http-headers -p80 IP
sudo nmap --script=http-methods -p80 IP
```

---

### SSH — Enumeração
```bash
sudo nmap --script=ssh-auth-methods -p22 IP
sudo nmap --script=ssh-brute -p22 IP
```

---

## 🔁 Como o NSE Funciona (internamente)

```
1. Nmap encontra porta aberta
2. NSE identifica o serviço
3. Script apropriado envia probes específicos
4. Analisa as respostas
5. Retorna informação estruturada
```

É enumeração automatizada e inteligente.

---

## ⚠️ Importante

Nem todo script é silencioso. Alguns podem:
- Gerar logs nos sistemas-alvo
- Travar serviços frágeis
- Acionar IDS/IPS

Sempre tratar NSE como **ativo**, não passivo.

---

## 🧠 Port Scanning vs NSE

> Port scanning diz: **"existe algo aqui"**
> NSE pergunta: **"o que exatamente isso permite fazer?"**

---

## 📌 Relacionados

- [[Nmap — Visão Geral]]
- [[Nmap — Service & OS Detection]]
- [[IDS & IPS]]
- [[Firewall — Conceito]]

#ferramenta/nmap #recon/ativo
