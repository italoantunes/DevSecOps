# 01 — IDE + Pre-commit

> **Posição no pipeline:** Antes de tudo — local, na máquina do desenvolvedor.  
> **O que protege:** Captura problemas antes que cheguem ao repositório.

---

## Por que começar aqui?

Imagine descobrir que um colega commitou uma senha de banco de dados no código. Agora essa senha está no histórico do Git — e mesmo que alguém delete o arquivo depois, o histórico permanece. Se o repositório é público, qualquer pessoa no mundo pode ver.

Esse tipo de problema tem custo zero para corrigir se for pego **antes do commit**. Depois que chega ao Git, o trabalho é muito maior.

Esta é a essência do Shift-Left: trazer os controles de segurança para o mais cedo possível no ciclo. O ponto mais cedo é a IDE do desenvolvedor.

```
  CUSTO DE CORREÇÃO
       ▲
  Alto │                                         ████ Produção
       │                               ████ Staging
       │                     ████ Pipeline CI
       │           ████ Commit / PR
  Baixo│  ████ IDE / Pre-commit   ← estamos aqui
       └─────────────────────────────────────────────▶ momento da descoberta
```

---

## Parte 1 — Plugins de Segurança na IDE

Plugins que rodam **em tempo real** enquanto você digita o código. Funcionam como um corretor ortográfico, mas para vulnerabilidades.

### SonarLint

**O que é:** Plugin gratuito que analisa o código em tempo real e aponta vulnerabilidades, bugs e code smells antes mesmo de você salvar o arquivo.

**Funciona em:** VS Code, IntelliJ IDEA, Eclipse, Visual Studio, PyCharm.

**Linguagens suportadas:** Java, Python, JavaScript, TypeScript, C#, PHP, Go, Kotlin e outras.

**Como instalar no VS Code:**
1. Abra o VS Code
2. Vá em Extensions (Ctrl+Shift+X)
3. Pesquise por `SonarLint`
4. Clique em Install

**O que detecta:** SQL Injection, XSS, senhas hardcoded, uso de funções criptográficas fracas (MD5, SHA1), variáveis não tratadas, e centenas de outros padrões.

**Conexão com SonarQube/SonarCloud (opcional):** Se sua empresa usa SonarQube, o SonarLint pode se conectar a ele e usar as mesmas regras do servidor. Assim o desenvolvedor vê os mesmos problemas que o pipeline detectaria.

---

### Snyk IDE

**O que é:** Plugin que foca em **dependências e bibliotecas**. Enquanto você escreve `import requests` ou `"express": "^4.17.1"`, o Snyk já verifica se essa versão tem vulnerabilidades conhecidas.

**Funciona em:** VS Code, IntelliJ, PyCharm, Eclipse, Visual Studio.

**Como instalar no VS Code:**
1. Extensions → pesquise `Snyk Security`
2. Instale e faça login com sua conta GitHub (plano free disponível)

**O que mostra:** CVE da vulnerabilidade, severidade, versão afetada, versão corrigida, e às vezes o fix automático com um clique.

---

### Semgrep IDE

**O que é:** Extension do Semgrep para a IDE. Permite rodar as mesmas regras que rodarão no CI, diretamente enquanto você desenvolve.

**Como instalar:** Extensions → `Semgrep` (publisher: Semgrep)

---

## Parte 2 — Pre-commit Hooks

Um **hook** é um script que o Git executa automaticamente em momentos específicos. O `pre-commit` roda **antes de cada `git commit`** ser finalizado. Se o script falhar, o commit é bloqueado.

```
$ git commit -m "adiciona feature X"
        │
        ▼
   [pre-commit hook executa]
        │
   ┌────┴────┐
   │ PASSOU  │ → commit é criado normalmente
   │ FALHOU  │ → commit é cancelado + mostra o problema
   └─────────┘
```

Você resolve o problema, e só depois consegue commitar. Simples e eficaz.

---

### Instalando o framework pre-commit

O `pre-commit` é uma ferramenta Python que gerencia hooks de forma padronizada:

```bash
# Instala o framework
pip install pre-commit

# Verifica a instalação
pre-commit --version
```

---

### Configurando os hooks — arquivo `.pre-commit-config.yaml`

Crie este arquivo na **raiz do repositório**:

```yaml
# .pre-commit-config.yaml
repos:
  # Gitleaks — detecta secrets, tokens e senhas no código
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.4
    hooks:
      - id: gitleaks

  # detect-secrets — camada extra de detecção de secrets
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets

  # Verifica arquivos com problemas comuns (trailing whitespace, fim de linha, etc.)
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-merge-conflict
      - id: detect-private-key       # bloqueia chaves privadas (.pem, .key)
```

Depois de criar o arquivo, ative os hooks no repositório:

```bash
# Ativa os hooks (executa uma vez por repositório clonado)
pre-commit install

# Testa rodando em todos os arquivos
pre-commit run --all-files
```

A partir daí, **cada `git commit` dispara os hooks automaticamente**.

---

### Gitleaks — o mais importante

O Gitleaks é especializado em encontrar secrets: tokens de API, senhas, chaves privadas, connection strings de banco de dados, chaves AWS, tokens GitHub, e mais de 150 padrões.

**Exemplo de saída quando encontra um problema:**

```
    │
    │ Secret detected!
    │
    │ Rule:     aws-access-token
    │ File:     src/config.py
    │ Line:     12
    │ Secret:   AKIA...XXXX
    │
    ├── Commit blocked. Fix the issue and try again.
```

**O que fazer quando o Gitleaks encontra algo:**
1. Nunca commite mesmo assim — o dano já pode ter ocorrido se esteve em outro branch
2. **Invalide o secret imediatamente** — vá na plataforma (AWS, GitHub, etc.) e revogue a chave
3. Gere uma nova chave e armazene em variável de ambiente ou secret manager
4. Só então commite o código corrigido

---

## Ferramentas — Resumo

| Ferramenta | Onde roda | O que detecta | Custo |
|---|---|---|---|
| **SonarLint** | IDE (tempo real) | Vulnerabilidades no código | Gratuito |
| **Snyk IDE** | IDE (tempo real) | CVEs em dependências | Gratuito (com limite) |
| **Semgrep IDE** | IDE (tempo real) | Padrões de segurança customizáveis | Gratuito |
| **Gitleaks** | Pre-commit | Secrets, tokens, senhas | Gratuito / Open-source |
| **detect-secrets** | Pre-commit | Secrets com menor falso-positivo | Gratuito / Open-source |

---

## Ganhos desta etapa

- **Custo zero de correção** — problema encontrado antes de existir no Git
- **Feedback imediato** — o desenvolvedor corrige na hora, sem abrir ticket
- **Nenhum secret chega ao repositório** — elimina uma das causas mais comuns de incidentes
- **Educação contínua** — o dev aprende o padrão seguro enquanto escreve

---

## Referências

- [pre-commit framework](https://pre-commit.com)
- [Gitleaks](https://github.com/gitleaks/gitleaks)
- [SonarLint](https://www.sonarsource.com/products/sonarlint/)
- [Snyk IDE](https://docs.snyk.io/ide-tools)
- [OWASP — Secure Coding Practices](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/)
