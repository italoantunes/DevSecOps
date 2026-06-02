# 02 — Secret Scanning

> **Posição no pipeline:** CI — roda a cada Push e Pull Request.  
> **O que protege:** Detecta senhas, tokens e chaves de API expostos no código ou no histórico do Git.

---

## O problema

Um "secret" é qualquer informação que dá acesso a um sistema: senha de banco de dados, token de API, chave AWS, chave privada SSH, connection string, credencial de serviço.

O erro mais comum é simples: o desenvolvedor coloca o valor diretamente no código para testar algo rapidamente e acaba commitando sem perceber.

```python
# ❌ Exemplo do que NÃO fazer
DATABASE_URL = "postgresql://admin:SenhaSecreta123@prod-db.empresa.com:5432/users"
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"
AWS_SECRET_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

**O que torna isso grave:**
- O Git guarda todo o histórico. Mesmo que o arquivo seja deletado depois, o secret ainda aparece em commits anteriores
- Se o repositório for público (ou vazar), qualquer pessoa pode usar essas credenciais
- Em média, um secret exposto no GitHub é encontrado por bots maliciosos em **menos de 1 minuto**
- O impacto vai de acesso ao banco de dados até comprometimento total da infraestrutura cloud

---

## Como o Secret Scanning funciona

As ferramentas de secret scanning usam **expressões regulares e padrões** para identificar formatos conhecidos de secrets. Elas sabem, por exemplo, que tokens AWS sempre começam com `AKIA`, que tokens GitHub começam com `ghp_`, que chaves privadas RSA começam com `-----BEGIN RSA PRIVATE KEY-----`.

```
Código commitado
      │
      ▼
Ferramenta varre cada linha de cada arquivo
      │
      ▼
Compara contra biblioteca de padrões (+150 tipos)
      │
      ├── Encontrou algo? → Bloqueia + alerta
      └── Nada encontrado → Pipeline continua
```

---

## Ferramentas

### Gitleaks

A ferramenta mais usada para secret scanning. Open-source, rápida e com boa cobertura de padrões.

**Dois modos de uso:**

**1. No pre-commit hook** (local, antes do commit — veja `01-ide-precommit`):
```bash
# Escaneia o que está prestes a ser commitado
gitleaks protect --staged
```

**2. No CI/CD** (varre todo o histórico a cada push):
```bash
# Escaneia TODO o histórico do repositório
gitleaks detect --source . --exit-code 1
```

O `--exit-code 1` faz o pipeline falhar se encontrar algo — essencial para bloquear o merge.

**Configuração customizada** — arquivo `.gitleaks.toml` na raiz do repositório:
```toml
[extend]
useDefault = true   # usa os padrões built-in + os seus customizados abaixo

[[rules]]
id = "chave-interna-empresa"
description = "Token interno da empresa XYZ"
regex = '''empresa_token_[a-zA-Z0-9]{32}'''
tags = ["internal", "api"]

[allowlist]
# Permite falsos positivos conhecidos (ex: valores de exemplo em testes)
regexes = [
  '''EXAMPLE_KEY''',
  '''TEST_SECRET'''
]
```

---

### TruffleHog

Especializado em **varrer histórico profundo do Git**. Enquanto o Gitleaks é mais rápido, o TruffleHog é mais minucioso e consegue verificar se o secret encontrado é **válido** (testando contra a API correspondente).

```bash
# Escaneia todo o histórico de um repositório
trufflehog git https://github.com/sua-org/seu-repo.git

# Escaneia repositório local
trufflehog git file://. --only-verified
```

O `--only-verified` testa os secrets encontrados contra as APIs reais e retorna apenas os que ainda estão ativos — reduz muito o ruído de falsos positivos.

---

### GitHub Secret Scanning (nativo)

O próprio GitHub tem um scanner nativo que roda automaticamente em **todos os repositórios públicos** e em repositórios privados com GitHub Advanced Security.

**Ativação:** `Settings → Security → Secret scanning → Enable`

**Como funciona:**
- Escaneia cada push automaticamente
- Notifica o responsável pelo repositório por e-mail
- Para tipos de secrets conhecidos (tokens GitHub, AWS, Google, Azure), o GitHub avisa **a plataforma parceira** automaticamente para revogar o token

Não precisa de configuração adicional — basta ativar.

---

## O que fazer quando um secret é encontrado

> ⚠️ **Regra de ouro:** trate todo secret exposto como comprometido, mesmo que "só ficou por alguns minutos".

**Passo a passo:**

```
1. REVOGAR IMEDIATAMENTE
   → Vá na plataforma (AWS Console, GitHub Settings, etc.)
   → Invalide / delete a credencial exposta
   → Isso impede que alguém a use, mesmo que já tenha sido copiada

2. VERIFICAR SE FOI USADO
   → Cheque os logs de acesso da plataforma
   → AWS: CloudTrail → filtre por access key
   → GitHub: Settings → Security log
   → Procure por acessos em horários ou IPs suspeitos

3. GERAR NOVA CREDENCIAL
   → Crie uma nova chave/token
   → Armazene corretamente (ver seção abaixo)

4. LIMPAR O HISTÓRICO (se necessário)
   → Se o repositório é público, use `git filter-repo` para remover o secret do histórico
   → Se privado, avalie o risco — a limpeza do histórico é irreversível e pode causar conflitos

5. INVESTIGAR A CAUSA
   → Por que o secret foi parar no código?
   → O pre-commit hook estava configurado?
   → Criar ação corretiva para evitar recorrência
```

---

## Como armazenar secrets corretamente

Nunca no código. Nunca em variáveis de ambiente plain text em produção. O lugar certo depende do contexto:

| Contexto | Onde armazenar |
|---|---|
| Desenvolvimento local | Arquivo `.env` local (no `.gitignore`) |
| CI/CD GitHub Actions | `Settings → Secrets and variables → Actions` |
| CI/CD GitLab | `Settings → CI/CD → Variables` (tipo "masked") |
| Aplicação em cloud | AWS Secrets Manager, GCP Secret Manager, Azure Key Vault |
| Kubernetes | Kubernetes Secrets + External Secrets Operator |
| Multiplataforma | HashiCorp Vault |

**Arquivo `.env` local — regra obrigatória:**

Sempre adicione `.env` ao `.gitignore` antes de criar o arquivo:
```
# .gitignore
.env
.env.local
.env.*.local
*.pem
*.key
```

---

## Ganhos desta etapa

- **Elimina o vetor de ataque mais comum** — credenciais expostas estão entre as principais causas de breaches
- **Detecção automática** — não depende de revisão manual no code review
- **Cobertura do histórico** — encontra secrets que foram commitados no passado e nunca removidos
- **Resposta rápida** — ferramentas como TruffleHog verificam se o secret ainda está ativo

---

## Referências

- [Gitleaks](https://github.com/gitleaks/gitleaks)
- [TruffleHog](https://github.com/trufflesecurity/trufflehog)
- [GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning)
- [OWASP — Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [CIS Control 3 — Data Protection](https://www.cisecurity.org/controls/v8)
