# 🔓 Have I Been Pwned

> Verifica se emails, senhas ou números de telefone foram expostos em vazamentos de dados públicos.

---

## 🧠 O Que É

Have I Been Pwned (HIBP) é uma base de dados de credenciais e informações pessoais coletadas de vazamentos (breaches) públicos.

**Site:** `https://haveibeenpwned.com`

---

## 🔧 Como Usar

**Via browser:**
```
https://haveibeenpwned.com
→ Digite o email
→ Veja os breaches associados
```

**Via API (automação):**
```bash
curl https://haveibeenpwned.com/api/v3/breachedaccount/email@exemplo.com \
  -H "hibp-api-key: SUA_KEY"
```

---

## 📋 O Que Retorna

Para cada email comprometido, mostra:
- Nome do breach (ex: LinkedIn 2012, Adobe 2013)
- Data do vazamento
- Dados expostos (email, senha, nome, telefone…)
- Quantidade de contas afetadas

---

## 🎯 Relevância em Pentest

| Dado Encontrado | Uso |
|----------------|-----|
| Email confirmado | Valida que o email é real |
| Senha em texto claro | Credential stuffing direto |
| Hash de senha | Cracking offline (hashcat, john) |
| Dados pessoais | Engenharia social mais precisa |

---

## 🔁 Workflow

```
theHarvester coleta emails do alvo
        ↓
Emails verificados no HIBP
        ↓
Senha em texto claro → tenta login direto
        ↓
Hash → hashcat/john → tenta login
        ↓
Sem senha → usa para engenharia social
```

---

## 💡 Dica

Muitas pessoas reutilizam senhas. Uma senha vazada de um breach antigo pode funcionar em sistemas atuais da empresa. Isso é **credential stuffing** e é um dos vetores mais eficazes na prática.

---

## 📌 Relacionados

- [[theHarvester]]
- [[Google Dorks]]
- [[MOC - Recon]]

#recon/passivo #ferramenta/osint
