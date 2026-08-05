# 🕷️ Burp Suite — Proxy de Interceptação Web

> Burp Suite é a ferramenta central de qualquer teste em aplicação web.
> Ele se posiciona entre o navegador e o servidor, permitindo interceptar, inspecionar e modificar cada requisição HTTP antes que ela chegue ao destino.

---

## 🧠 O Que é o Burp Suite

Um **proxy de interceptação** que atua como man-in-the-middle entre o cliente (navegador) e o servidor web, dando controle total sobre as requisições e respostas HTTP/HTTPS.

**Fluxo conceitual:**

Navegador → Burp Proxy (127.0.0.1:8080) → Servidor
↓
Requisição interceptada
(visualizar / editar / repetir / automatizar)


---

## 🔧 Configuração Inicial

### 1. Configurar proxy no navegador

Endereço: 127.0.0.1
Porta: 8080


### 2. Instalar certificado do Burp (para HTTPS)

Acessar http://burp no navegador com o proxy ativo
→ Baixar certificado CA
→ Importar nas autoridades certificadoras confiáveis do navegador


### 3. Confirmar interceptação

Proxy → Intercept → Intercept is on
Navegar até o alvo → requisição deve aparecer travada no Burp


---

## 🧰 Módulos Principais

| Módulo | Função |
|---|---|
| **Proxy** | Intercepta e exibe requisições/respostas em tempo real |
| **Target / Site map** | Mapeia estrutura da aplicação conforme você navega |
| **Repeater** | Reenvia manualmente uma requisição capturada, editando parâmetros à vontade |
| **Intruder** | Automatiza ataques (brute force, fuzzing) usando listas de payload |
| **Decoder** | Codifica/decodifica dados (Base64, URL, Hex) |
| **Comparer** | Compara duas respostas lado a lado |

---

## 🎯 Repeater — Uso Prático
Interceptar requisição no Proxy
Botão direito → Send to Repeater
Editar parâmetro (ex: id=1 → id=1' OR '1'='1)
Send
Analisar resposta (tamanho, status code, conteúdo)

---

## ⚔️ Intruder — Tipos de Ataque

| Tipo | Comportamento |
|---|---|
| **Sniper** | Um payload por vez, em uma posição marcada |
| **Battering ram** | Mesmo payload em todas as posições simultaneamente |
| **Pitchfork** | Payloads diferentes, sincronizados por posição |
| **Cluster bomb** | Todas as combinações entre múltiplas listas de payload |

**Workflow típico de brute force de login:**
Interceptar requisição de login
Send to Intruder
Marcar posição do parâmetro (username ou password) com §
Attack type: Sniper
Payloads → carregar wordlist
Start attack
Ordenar por Status Code / Length para achar a resposta diferente

---

## 📌 Relacionados

- [[SQL Injection]]
- [[Cross-Site Scripting (XSS)]]
- [[Hydra]]
- [[Nmap — Visão Geral]]

#web #burpsuite #proxy #enumeracao