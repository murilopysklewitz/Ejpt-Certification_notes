# 📂 Dirb

> Enumera diretórios e arquivos em servidores web usando wordlists.
> Força bruta de paths para descobrir conteúdo não linkado publicamente.

---

## 🧠 O Que Faz

Dirb faz requisições HTTP para cada palavra de uma wordlist e verifica qual retorna resposta válida (200, 301, 302, etc.). Descobre páginas, painéis e diretórios ocultos.

---

## 🔧 Comandos

```bash
# Básico (wordlist padrão)
dirb http://IP

# URL específica
dirb http://IP/pasta/

# Wordlist customizada
dirb http://IP /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# Extensões específicas
dirb http://IP -X .php,.html,.txt

# Ignorar códigos de resposta
dirb http://IP -N 404

# Salvar output
dirb http://IP -o resultado.txt
```

---

## 📋 Wordlists Comuns

```
/usr/share/wordlists/dirb/common.txt        (rápida)
/usr/share/wordlists/dirb/big.txt           (completa)
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  (referência)
```

---

## 🎯 O Que Procurar no Output

| Path Encontrado | Relevância |
|----------------|-----------|
| `/admin/` | Painel administrativo |
| `/backup/` | Backups expostos |
| `/upload/` | Upload de arquivos |
| `/config/` | Arquivos de configuração |
| `/api/` | Endpoints de API |
| `/.git/` | Repositório exposto |

---

## 📌 Relacionados

- [[Dirsearch]]
- [[Gobuster]]
- [[Web Fingerprinting]]

#ferramenta/web #recon/ativo
