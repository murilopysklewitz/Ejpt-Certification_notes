# 🚀 Gobuster

> Ferramenta de força bruta para diretórios, arquivos e subdomínios.
> Mais rápida que Dirb e Dirsearch por usar Go (concorrência nativa).

---

## 🧠 O Que Faz

Gobuster usa wordlists para descobrir diretórios, arquivos e subdomínios via força bruta. Tem modos diferentes: `dir` (diretórios), `dns` (subdomínios), `vhost` (virtual hosts).

---

## 🔧 Modos e Comandos

### dir — Enumeração de Diretórios/Arquivos
```bash
# Básico
gobuster dir -u http://IP -w /usr/share/wordlists/dirb/common.txt

# Com extensões
gobuster dir -u http://IP \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,html,txt,js

# Threads (default=10)
gobuster dir -u http://IP -w wordlist.txt -t 50

# Ignorar códigos
gobuster dir -u http://IP -w wordlist.txt -b 404,403

# Com autenticação
gobuster dir -u http://IP -w wordlist.txt -U user -P senha

# Salvar output
gobuster dir -u http://IP -w wordlist.txt -o resultado.txt
```

---

### dns — Enumeração de Subdomínios
```bash
gobuster dns -d dominio.com -w /usr/share/wordlists/subdomains.txt
```

---

### vhost — Virtual Hosts
```bash
gobuster vhost -u http://IP -w wordlist.txt
```

---

## 📋 Wordlists Recomendadas

```bash
# Rápida (recon inicial)
/usr/share/wordlists/dirb/common.txt

# Completa (mais cobertura)
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# Instalação SecLists (gold standard)
apt install seclists
/usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
```

---

## ⚡ Comando Padrão de Lab

```bash
gobuster dir \
  -u http://IP \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,html,txt \
  -t 30
```

---

## 📌 Relacionados

- [[Dirb]]
- [[Dirsearch]]
- [[Web Fingerprinting]]
- [[SubList3er]]

#ferramenta/web #recon/ativo
