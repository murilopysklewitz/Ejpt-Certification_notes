# 📁 Cadaver — Cliente WebDAV de Linha de Comando

> Cliente interativo para servidores WebDAV via linha de comando.
> Funciona como um "FTP client" mas para WebDAV — navega, faz upload, download e gerencia arquivos remotamente via HTTP.

---

## 🧠 O Que é Cadaver

**Cadaver** é um cliente WebDAV para Linux/Unix que oferece uma interface interativa similar ao cliente FTP. Quando um servidor tem WebDAV habilitado (IIS, Apache com mod_dav, etc.), o Cadaver permite interagir diretamente com o sistema de arquivos remoto via HTTP.

**Por que é relevante em pentest:**
- Interface simples para explorar WebDAV manualmente
- Confirma se PUT está funcional (além de só checar o header)
- Permite upload de arquivos de forma direta e interativa
- Útil quando ferramentas automatizadas falham

---

## 🔧 Instalação

```bash
# Debian/Ubuntu/Kali
apt install cadaver

# Verificar instalação
cadaver --version
```

---

## 🔌 Conectar ao Servidor

```bash
# Conexão básica (sem credencial)
cadaver http://IP/

# Com porta não-padrão
cadaver http://IP:8080/

# Com path específico
cadaver http://IP/webdav/

# HTTPS
cadaver https://IP/
```

Ao conectar, entra no **prompt interativo**:
```
dav:/> 
```

---

## 📋 Comandos Internos

```bash
# Navegação
ls                    # listar arquivos e diretórios
ls -l                 # listagem detalhada
pwd                   # diretório atual
cd pasta              # entrar em diretório
cd ..                 # voltar um nível

# Transferência de arquivos
get arquivo.txt       # baixar arquivo para diretório local
get arquivo.txt /tmp/ # baixar para caminho específico
put arquivo.txt       # fazer upload de arquivo local
put /caminho/local/arquivo.txt  # upload com path completo
mget *.txt            # baixar múltiplos arquivos
mput *.txt            # upload de múltiplos arquivos

# Gerenciamento
mkdir novodir         # criar diretório no servidor
rmdir pasta           # remover diretório
delete arquivo.txt    # deletar arquivo remoto
move origem destino   # mover/renomear arquivo
copy origem destino   # copiar arquivo

# Informações
propget arquivo.txt   # ver propriedades do arquivo
propset arquivo.txt chave valor  # definir propriedade

# Sessão
open http://IP/       # conectar a novo servidor
close                 # fechar conexão
quit / exit / bye     # sair do cadaver
help                  # listar todos os comandos
```

---

## 🔐 Autenticação

```bash
# O cadaver solicita credenciais automaticamente quando necessário
dav:/> ls
Authentication required for realm 'WebDAV' on server 'IP':
Username: administrator
Password:
```

```bash
# Ou informar na URL (menos seguro — aparece no histórico)
cadaver http://usuario:senha@IP/
```

---

## 🔁 Workflow Prático

### Verificar acesso e fazer upload

```bash
# 1. Conectar
cadaver http://IP/

# 2. Verificar o que existe
dav:/> ls

# 3. Navegar para diretório gravável
dav:/> cd /uploads/

# 4. Testar se PUT funciona
dav:/> put /tmp/teste.txt

# 5. Se sucesso → confirmar o upload
dav:/> ls
```

---

## ⚠️ Limitações do Cadaver

- **Não testa automaticamente** se extensões executáveis são permitidas — só faz o upload
- **Sem automação** — cada ação é manual
- **Sem verificação de extensão** — você precisa testar cada extensão separadamente
- Para teste automatizado de extensões → usar **DAVTest**

---

## 🧠 Cadaver vs DAVTest

| | Cadaver | DAVTest |
|-|---------|---------|
| Tipo | Interativo manual | Automatizado |
| Testa extensões executáveis | ❌ Manual | ✅ Automático |
| Upload interativo | ✅ | ❌ |
| Melhor para | Exploração manual | Reconhecimento rápido |

**Uso ideal:** DAVTest primeiro (automatizado) → Cadaver para ação manual específica.

---

## 📌 Relacionados

- [[IIS — Internet Information Services]]
- [[DAVTest — Testando WebDAV]]
- [[ASP Webshell — Upload e Execução]]
- [[FTP — File Transfer Protocol]]

#ferramenta/webdav #exploração #windows #iis
