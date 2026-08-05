# 📂 Dirsearch

> Enumera páginas e arquivos em aplicações web.
> Mais moderno que o Dirb, com melhor output e mais opções.

---

## 🧠 O Que Faz

Dirsearch é um scanner de diretórios e arquivos web que usa wordlists para descobrir conteúdo oculto. Detecta automaticamente extensões comuns e tem filtros avançados.

---

## 🔧 Comandos

```bash
# Básico
dirsearch -u http://IP

# Com extensões específicas
dirsearch -u http://IP -e php,html,js,txt

# Wordlist customizada
dirsearch -u http://IP -w /caminho/wordlist.txt

# Threads (velocidade)
dirsearch -u http://IP -t 50

# Filtrar por código HTTP
dirsearch -u http://IP --exclude-status 403,404

# Salvar output
dirsearch -u http://IP -o resultado.txt

# Com proxy (Burp)
dirsearch -u http://IP --proxy http://127.0.0.1:8080
```

---

## 📋 Extensões Úteis

```bash
# Aplicação PHP
dirsearch -u http://IP -e php,txt,html,bak,old

# Aplicação Python/Django
dirsearch -u http://IP -e py,html,txt,json

# Aplicação Java
dirsearch -u http://IP -e java,jsp,xml,properties
```

---

## 📊 Diferença: Dirb vs Dirsearch vs Gobuster

| Ferramenta | Velocidade | Flexibilidade | Output |
|-----------|-----------|---------------|--------|
| Dirb | Lenta | Básica | Simples |
| Dirsearch | Média | Alta | Organizado |
| Gobuster | Rápida | Alta | Limpo |

---

## 🎯 O Que Procurar

- Painéis de admin (`/admin`, `/administrator`, `/panel`)
- Páginas de login esquecidas
- Backups (`/backup`, `/.bak`, `/old`)
- Arquivos de configuração (`/config.php`, `/.env`)
- Repositórios expostos (`/.git`, `/.svn`)

---

## 📌 Relacionados

- [[Dirb]]
- [[Gobuster]]
- [[Web Fingerprinting]]

#ferramenta/web #recon/ativo
