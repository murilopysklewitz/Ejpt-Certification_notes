# 🔍 DNS Interrogation

> Processo de consultar servidores DNS **diretamente** para obter informações sobre um domínio.
> Ao contrário do DNSRecon passivo, aqui você interage com os servidores do alvo.

---

## 🧠 O Que É

DNS Interrogation é fazer perguntas diretas aos servidores DNS de um domínio para descobrir sua estrutura: IPs, subdomínios, servidores de email, e potencialmente realizar Zone Transfer.

---

## 🔧 Ferramentas e Comandos

### host (simples)
```bash
# IP do domínio
host dominio.com

# Record MX
host -t MX dominio.com

# Record NS
host -t NS dominio.com
```

---

### dig (completo)
```bash
# Record A
dig dominio.com

# Record específico
dig MX dominio.com
dig NS dominio.com
dig TXT dominio.com

# Zone Transfer (tentativa)
dig axfr @ns1.dominio.com dominio.com
```

---

### dnsenum (automatizado)
```bash
dnsenum dominio.com
```
Faz brute force de subdomínios + tenta zone transfer automaticamente.

---

### fierce (zone transfer + brute force)
```bash
fierce -dns dominio.com
```

---

## ⚡ Zone Transfer (AXFR) — O Jackpot

Se um servidor NS estiver mal configurado para aceitar zone transfers de qualquer IP, você recebe **todos os registros DNS de uma vez**:

```bash
# Pegar os name servers primeiro
host -t NS dominio.com

# Tentar zone transfer em cada NS
dig axfr @ns1.dominio.com dominio.com
dig axfr @ns2.dominio.com dominio.com
```

**Se funcionar, você vê tudo:**
- Todos os subdomínios
- Todos os IPs
- Infraestrutura interna exposta

> Na prática moderna, zone transfers públicos são raros mas ainda existem em ambientes mal configurados, especialmente em labs e CTFs.

---

## 🎯 O Que Procurar

| Consulta | Por Quê |
|---------|---------|
| `host dominio.com` | IP real do servidor |
| `host -t MX` | Provedor de email |
| `host -t NS` | Quem gerencia o DNS |
| Zone Transfer | Mapa completo da infra |

---

## 📌 Relacionados

- [[DNS — Records & Funcionamento]]
- [[DNSRecon]]
- [[DNSdumpster]]
- [[Wafw00f]]

#recon/ativo #ferramenta/dns #protocolo/dns
