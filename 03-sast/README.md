# 03 — SAST — Static Application Security Testing

> **Posição no pipeline:** CI — roda a cada Push e Pull Request.  
> **O que protege:** Vulnerabilidades no código que você escreveu — sem precisar executar a aplicação.

---

## O que é SAST?

SAST é a análise do **código-fonte** em busca de vulnerabilidades. A ferramenta lê o código como texto, entende a lógica, e identifica padrões que podem ser explorados por atacantes.

A analogia mais simples: é como um revisor especializado em segurança que lê cada linha do seu código, conhece milhares de padrões de vulnerabilidade, nunca se cansa e devolve o resultado em segundos.

**O que o SAST encontra:**

| Vulnerabilidade | Exemplo no código |
|---|---|
| SQL Injection | `query = "SELECT * FROM users WHERE id = " + user_input` |
| Cross-Site Scripting (XSS) | `document.innerHTML = req.query.name` |
| Command Injection | `os.system("ls " + user_path)` |
| Path Traversal | `open("/files/" + filename)` |
| Criptografia fraca | `hashlib.md5(password)` |
| Deserialização insegura | `pickle.loads(user_data)` |
| Secrets hardcoded | `password = "admin123"` |

**O que o SAST não encontra:** Vulnerabilidades que só aparecem em tempo de execução — como lógica de negócio incorreta, problemas de autorização entre usuários, ou comportamentos dependentes de configuração de ambiente. Para isso existe o DAST (etapa 07).

---

## Ferramentas

### Semgrep

A ferramenta open-source mais popular para SAST. Rápida, precisa, com regras para dezenas de linguagens e frameworks.

**Linguagens:** Python, JavaScript, TypeScript, Java, Go, Ruby, PHP, C/C++, Kotlin, Scala e outras.

**Como funciona:** O Semgrep usa um sistema de regras em YAML que descrevem padrões de código inseguro. Existem milhares de regras prontas da comunidade, e você pode escrever as suas para padrões específicos da empresa.

**Rodando localmente:**
```bash
# Instala
pip install semgrep

# Roda com o conjunto de regras automático (recomendado para começar)
semgrep --config=auto .

# Roda apenas as regras do OWASP Top 10
semgrep --config=p/owasp-top-ten .

# Roda em um arquivo específico
semgrep --config=auto src/app.py
```

**Exemplo de saída:**
```
  src/auth/login.py
  ❯❯❱ python.lang.security.audit.sql-injection
        SQL Injection via string concatenation
        
        78 │ query = "SELECT * FROM users WHERE email = '" + email + "'"
        
        Details: https://semgrep.dev/r/python.lang.security.audit.sql-injection
```

**Regra customizada — exemplo:**

Se sua empresa tem um padrão interno que não deve ser usado:
```yaml
# regras/empresa-custom.yaml
rules:
  - id: nao-usar-funcao-legada
    patterns:
      - pattern: legacy_auth($USER, $PASS)
    message: "Use a função authenticate() do módulo auth_v2. legacy_auth() foi descontinuada."
    languages: [python]
    severity: ERROR
```

---

### SonarQube / SonarCloud

Plataforma mais completa — combina qualidade de código com segurança. O **SonarCloud** é a versão SaaS (gratuita para repositórios públicos). O **SonarQube** é para instalação própria.

**Além do SAST, o SonarQube também analisa:**
- Cobertura de testes
- Code smells e débito técnico
- Duplicação de código
- Complexidade ciclomática

**Quality Gate:** funcionalidade que bloqueia o merge se o código não atender critérios mínimos de qualidade e segurança. Você define os critérios (ex: nenhum bug crítico novo, cobertura de testes acima de 80%).

**Integração com GitHub:**
1. Acesse [sonarcloud.io](https://sonarcloud.io)
2. Conecte com sua conta GitHub
3. Importe o repositório
4. Adicione o token `SONAR_TOKEN` nos secrets do repositório
5. Adicione o step no workflow (veja `pipeline/github-actions/security.yml`)

---

### Ferramentas por linguagem

Para linguagens específicas, existem ferramentas focadas que complementam o Semgrep:

| Linguagem | Ferramenta | Observação |
|---|---|---|
| Python | **Bandit** | Especializado em Python, muito usado em pipelines |
| Java | **SpotBugs + FindSecBugs** | Plugin do SpotBugs focado em segurança |
| JavaScript / Node | **ESLint + plugins de segurança** | eslint-plugin-security |
| Go | **Gosec** | Padrões de segurança específicos do Go |
| Ruby on Rails | **Brakeman** | Focado em Rails, muito preciso |
| .NET / C# | **Security Code Scan** | Plugin para Visual Studio e MSBuild |

---

## Integrando no pipeline

O SAST deve rodar em **todo Pull Request** e bloquear o merge se encontrar findings CRITICAL.

No GitHub Actions (arquivo em `pipeline/github-actions/security.yml`):
```yaml
sast:
  name: SAST (Semgrep)
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: returntocorp/semgrep-action@v1
      with:
        config: auto          # usa regras automáticas baseadas nas linguagens do repo
      env:
        SEMGREP_APP_TOKEN: ${{ secrets.SEMGREP_APP_TOKEN }}
```

---

## Como lidar com os resultados

**CRITICAL** → Bloqueia. Não merge enquanto não estiver corrigido.

**HIGH** → Bloqueia em ambientes maduros. Em times que estão começando, pode ser alerta com SLA de 7 dias.

**MEDIUM / LOW** → Não bloqueia. Registra para revisão. O desenvolvedor deve avaliar se é um falso positivo ou uma vulnerabilidade real.

**Falsos positivos:** São comuns em SAST. A ferramenta não entende o contexto do negócio e pode alertar sobre código que é seguro no seu caso específico. Você pode criar exceções documentadas:

```python
# Em Semgrep, para suprimir um finding específico:
query = "SELECT * FROM users WHERE id = %s"  # nosec: sql-injection (usando parametrização)
```

---

## Ganhos desta etapa

- **Cobertura automática** — cada linha de código novo é analisada sem esforço manual
- **Feedback no PR** — o desenvolvedor vê o problema antes de o código chegar à main
- **Educação contínua** — o dev aprende padrões seguros ao ver os findings e as explicações
- **Cobertura do OWASP Top 10** — as principais vulnerabilidades de aplicação web são cobertas

---

## Referências

- [Semgrep](https://semgrep.dev)
- [SonarCloud](https://sonarcloud.io)
- [OWASP Testing Guide — Code Review](https://owasp.org/www-project-web-security-testing-guide/)
- [CWE Top 25 Most Dangerous Software Weaknesses](https://cwe.mitre.org/top25/)
- [OWASP Top 10:2021](https://owasp.org/Top10)
