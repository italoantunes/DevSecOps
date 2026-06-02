# Conceito — Shift-Left Security

> "Corrigir cedo é barato. Corrigir tarde é caro. Não corrigir é catastrófico."

---

## O que significa Shift-Left?

"Shift-Left" é a ideia de mover os controles de segurança para **mais cedo** no ciclo de desenvolvimento.

Em um fluxo de desenvolvimento representado como linha do tempo (da esquerda para a direita), "shift-left" significa empurrar as verificações de segurança para a esquerda — para mais perto do início.

```
ANTES (Shift-Right):
─────────────────────────────────────────────────────────────────────────────▶
Código → Código → Código → Testes → Deploy → Testes de segurança → Incidente?

DEPOIS (Shift-Left):
─────────────────────────────────────────────────────────────────────────────▶
Sec → Código → Sec → Sec → Testes → Sec → Deploy → Sec → Monitoramento
↑          ↑       ↑              ↑           ↑
IDE    Pre-commit  CI           Staging    Produção
```

---

## Por que o custo aumenta com o tempo?

Imagine que uma vulnerabilidade de SQL Injection foi introduzida no código:

**Se encontrada na IDE (pelo SonarLint):**
- O desenvolvedor vê o sublinhado vermelho enquanto digita
- Corrige em 2 minutos
- Ninguém mais precisa saber
- **Custo: ~R$ 5 (2 minutos do desenvolvedor)**

**Se encontrada no code review (PR):**
- Reviewer comenta, developer corrige, novo push, nova revisão
- **Custo: ~R$ 50 (30 minutos de duas pessoas)**

**Se encontrada no CI (SAST no pipeline):**
- Build quebra, developer volta ao contexto do código, entende o erro, corrige, novo PR
- **Custo: ~R$ 200 (1-2 horas + overhead de contexto)**

**Se encontrada em staging (DAST):**
- Issue criada, priorizada, developer retoma o trabalho, testa, novo deploy no staging
- **Custo: ~R$ 500-1.000 (meio dia de trabalho, múltiplos envolvidos)**

**Se encontrada em produção (pentest ou incidente):**
- Hotfix urgente, possível rollback, comunicação de incidente, investigação, patches
- **Custo: R$ 50.000 a R$ 500.000+ (dependendo do impacto e regulação)**

**Se virar um breach com dados expostos:**
- Multas LGPD (até 2% do faturamento), dano reputacional, clientes perdidos, processos
- **Custo: incalculável**

---

## Shift-Left na prática — o que muda

### 1. Segurança nos requisitos

Antes de escrever uma linha de código, a equipe já pensa:
- Quais dados sensíveis esse feature vai processar?
- Como a autenticação e autorização vão funcionar?
- Existe regulação aplicável (LGPD, PCI)?

Resultado: o design já nasce seguro, sem precisar ser "corrigido" depois.

### 2. Developer tem as ferramentas

O desenvolvedor precisa de feedback rápido:
- SonarLint na IDE → vê vulnerabilidade enquanto digita
- Snyk IDE → vê CVE na dependência enquanto importa
- Pre-commit com Gitleaks → bloqueado antes de commitar um secret

Sem as ferramentas, mesmo o developer mais cuidadoso não tem como saber.

### 3. Security gates no pipeline, não no final

O pipeline de CI valida segurança em cada PR — não uma vez por trimestre em um pentest:

```
Todo PR →  Secret Scan  →  SAST  →  SCA  →  Container  →  IaC
                ↓             ↓       ↓          ↓           ↓
           Bloqueia se    Bloqueia  Bloqueia   Bloqueia   Bloqueia
           encontrar      CRITICAL  CRITICAL   CRITICAL   CRITICAL
           secrets
```

### 4. Segurança na Definition of Done

O time só considera uma história como "pronta" quando:
- Todos os gates do pipeline passaram
- Nenhum finding CRITICAL em aberto
- Checklist de segurança do PR preenchido

Assim a segurança deixa de ser "responsabilidade do time de segurança" e vira responsabilidade de toda a squad.

---

## O que Shift-Left NÃO é

- **Não é eliminar o pentest** — o pentest ainda é necessário, especialmente para vulnerabilidades de lógica de negócio
- **Não é só ferramentas** — sem cultura e treinamento, as ferramentas ficam desligadas ou ignoradas
- **Não é segurança como bloqueio** — o objetivo é feedback rápido, não criar obstáculos
- **Não é perfeição** — o objetivo é encontrar problemas mais cedo, não garantir que nenhum chegará à produção

---

## Métricas de maturidade Shift-Left

Como saber se o time está evoluindo:

| Métrica | Início | Meta |
|---|---|---|
| % de vulnerabilidades encontradas antes do merge | < 20% | > 80% |
| Tempo médio para corrigir uma HIGH | > 30 dias | < 7 dias |
| % de PRs com security checklist preenchido | < 50% | 100% |
| Número de secrets detectados em produção | > 0 | 0 |
| Cobertura de SAST no CI | < 50% dos repositórios | 100% |

---

## Referências

- [OWASP DevSecOps Guideline](https://owasp.org/www-project-devsecops-guideline/)
- [NIST SP 800-218 — SSDF](https://csrc.nist.gov/publications/detail/sp/800-218/final)
- [IBM — Cost of a Data Breach Report](https://www.ibm.com/reports/data-breach)
- [GitLab DevSecOps Survey](https://about.gitlab.com/developer-survey/)
