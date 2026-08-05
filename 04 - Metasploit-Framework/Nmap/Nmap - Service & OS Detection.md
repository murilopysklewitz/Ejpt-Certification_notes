# 🔬 Nmap — Service & OS Detection

> Identificar **o que está rodando** em cada porta e **qual sistema operacional** está no host.
> Versão = vetor de ataque. Saber a versão exata importa.

---

## 🔧 -sV — Service Version Detection

```bash
# Básico
sudo nmap -sV IP

# Eficiente: detecta versão só nas portas abertas
sudo nmap -sV -p 22,80,443 IP

# Intensidade da detecção (0-9, padrão=7)
sudo nmap -sV --version-intensity 9 IP
```

### O Que Retorna

```
80/tcp  open  http    Apache httpd 2.4.49
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3
3306/tcp open  mysql   MySQL 5.7.36
```

Agora você tem tecnologia + versão + potencial CVE.

---

## 🔧 -O — OS Detection

```bash
# Básico
sudo nmap -O IP

# Forçar palpite mesmo com baixa confiança
sudo nmap -O --osscan-guess IP

# Limite de tentativas (mais rápido)
sudo nmap -O --osscan-limit IP
```

### Como Funciona

O Nmap analisa comportamento da pilha TCP/IP:
- TTL das respostas
- Tamanho da janela TCP
- Padrões de resposta ICMP
- Timestamps

Cada SO responde de forma ligeiramente diferente. Nmap compara com sua base de fingerprints.

### O Que Retorna

```
OS details: Linux 5.4 - 5.15
```
ou
```
OS details: Microsoft Windows 10 Pro
```

### ⚠️ Requisitos para Funcionar Bem

- Pelo menos 1 porta **open** E 1 porta **closed**
- Executar como root (`sudo`)
- Firewall pode atrapalhar muito

---

## 🔧 --osscan-guess

```bash
sudo nmap -O --osscan-guess IP
```

Sem essa flag, quando há muitos matches possíveis o Nmap diz:
> "Too many fingerprints match this host"

Com `--osscan-guess`, ele arrisca uma hipótese ordenada por probabilidade.
⚠️ Pode errar. Tratar como palpite.

---

## ⚡ Comando Completo (Recon Profundo)

```bash
sudo nmap -T4 -sS -sV -O --osscan-guess -p- IP
```

| Flag | Função |
|------|--------|
| `-T4` | Velocidade (lab) |
| `-sS` | SYN scan |
| `-sV` | Versão dos serviços |
| `-O` | OS detection |
| `--osscan-guess` | Forçar hipótese de OS |
| `-p-` | Todas as 65535 portas |

---

## 🔥 -A — Modo Agressivo

```bash
sudo nmap -A IP
```

Equivale a:
- `-sV` (versão)
- `-O` (OS)
- `-sC` (scripts padrão NSE)
- `--traceroute`

**Use em:** labs, CTFs, ambiente controlado.
**Evite em:** produção, redes com IDS sensível — é muito barulhento.

---

## 📊 Diferença Clara

| Técnica | Descobre |
|---------|---------|
| Port Scan | Porta aberta/fechada/filtrada |
| `-sV` | Serviço + versão + às vezes distro |
| `-O` | Sistema operacional estimado |

---

## 🔁 Workflow Eficiente

```bash
# Passo 1: descobrir portas
sudo nmap -p- -T4 IP

# Passo 2: apenas nas abertas, versão + OS
sudo nmap -sV -O --osscan-guess -p 22,80,443 IP
```

Mais rápido que `-A` de cara, e menos ruído.

---

## ⚠️ Problemas Comuns

- **Firewall** → portas aparecem como `filtered`
- **NAT** → OS detection pode dar errado
- **Sem porta closed** → OS detection pode falhar
- **Windows Defender** → pode bloquear probes

---

## 📌 Relacionados

- [[Nmap — Port Scanning]]
- [[Nmap — NSE]]
- [[Nmap — Visão Geral]]
- [[Transport Layer — TCP & UDP]]

#ferramenta/nmap #recon/ativo
