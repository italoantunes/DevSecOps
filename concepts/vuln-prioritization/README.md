# Conceito — Priorização de Vulnerabilidades

> Nem todo CVE é urgente. Saber o que corrigir primeiro é tão importante quanto encontrar as vulnerabilidades.

---

## O problema do volume

Ferramentas de SAST, SCA e Container Scan geram dezenas ou centenas de findings por repositório. Se o time tentar corrigir tudo ao mesmo tempo, paralisa. Se ignorar tudo, o risco cresce sem controle.

A solução é **priorizar com critérios objetivos** — corrigir o que representa risco real para o negócio, no menor prazo necessário.

---

## Os fatores de priorização

Não basta olhar o CVSS. Uma vulnerabilidade com CVSS 9.0 em uma biblioteca que não está em uso no seu código é menos urgente do que uma com CVSS 7.5 em um endpoint exposto na internet com exploit público disponível.

### 1. CVSS Score — a base

O **Common Vulnerability Scoring System (CVSS)** é uma escala de 0 a 10 que indica a severidade técnica de uma vulnerabilidade:

| Faixa | Classificação |
|---|---|
| 9.0 – 10.0 | Crítico |
| 7.0 – 8.9 | Alto |
| 4.0 – 6.9 | Médio |
| 0.1 – 3.9 | Baixo |

**Limitação do CVSS:** mede a severidade técnica, não a probabilidade real de exploração no seu contexto específico.

### 2. EPSS Score — probabilidade real de exploração

O **EPSS (Exploit Prediction Scoring System)** estima a probabilidade de uma vulnerabilidade ser explorada nos próximos 30 dias, com base em dados reais de exploração.

- Vai de 0% a 100%
- Um CVE com CVSS 9.5 e EPSS 0.1% pode ser menos urgente do que um com CVSS 7.0 e EPSS 45%
- Consulte em: https://www.first.org/epss

### 3. CISA KEV — sendo explorado agora

O **CISA Known Exploited Vulnerabilities Catalog** lista vulnerabilidades que estão sendo **ativamente exploradas em ataques reais** neste momento.

Se um CVE está no KEV → tratamento imediato, independente do CVSS ou EPSS.

Consulte em: https://www.cisa.gov/known-exploited-vulnerabilities-catalog

### 4. Exposição do ativo

| Exposição | Risco |
|---|---|
| Exposto à internet (porta aberta, API pública) | Alto |
| Rede interna apenas | Médio |
| Isolado (sem rede, uso offline) | Baixo |

### 5. Criticidade do ativo para o negócio

| Criticidade | Exemplos |
|---|---|
| **Alto** | Sistema de pagamento, CDE (PCI), dados de clientes, API principal do produto |
| **Médio** | Sistemas de suporte, ferramentas internas, ambientes de homologação |
| **Baixo** | Ferramentas de desenvolvimento, ambientes de teste isolados |

### 6. Reachability — a lib está realmente em uso?

Uma vulnerabilidade em uma função que nunca é chamada no seu código tem impacto muito menor do que a mesma vulnerabilidade em uma função chamada em todo request.

Ferramentas como Snyk e algumas versões do Trivy fazem análise de reachability — verificam se o código vulnerável é realmente acessível na sua aplicação.

---

## A Matriz de Risco

Combine a **exploitabilidade** (CVSS + EPSS + exploit público) com a **criticidade do ativo**:

```
                      CRITICIDADE DO ATIVO / IMPACTO NO NEGÓCIO
                      ┌──────────────┬──────────────┬──────────────┐
                      │     BAIXO    │    MÉDIO     │     ALTO     │
                      │  (interno,   │  (sistemas   │  (produção,  │
                      │  não-crítico)│  de suporte) │  CDE, APIs)  │
  E  ┌────────────────┼──────────────┼──────────────┼──────────────┤
  X  │ CRÍTICO        │    ALTO      │   CRÍTICO    │   CRÍTICO    │
  P  │ CVSS 9–10      │   30 dias    │   7 dias     │  IMEDIATO    │
  L  │ + exploit pub. ├──────────────┼──────────────┼──────────────┤
  O  │ ALTO           │    MÉDIO     │    ALTO      │   CRÍTICO    │
  I  │ CVSS 7–8.9     │   60 dias    │   30 dias    │   7 dias     │
  T  ├────────────────┼──────────────┼──────────────┼──────────────┤
  A  │ MÉDIO          │    BAIXO     │    MÉDIO     │    ALTO      │
  B  │ CVSS 4–6.9     │   90 dias    │   60 dias    │   30 dias    │
  I  ├────────────────┼──────────────┼──────────────┼──────────────┤
  L  │ BAIXO / INFO   │   BACKLOG    │   BACKLOG    │    MÉDIO     │
  I  │ CVSS 0–3.9     │  trimestral  │  trimestral  │   90 dias    │
  D  └────────────────┴──────────────┴──────────────┴──────────────┘
  A
  D
  E
```

---

## Fluxo de decisão — passo a passo

```
Vulnerabilidade encontrada pelo pipeline
                  │
                  ▼
        ┌─────────────────────────────────────┐
        │ O código/lib vulnerável está sendo  │
        │ realmente usado? (reachability)      │
        └─────────────────────────────────────┘
                  │
          NÃO ───►│◄─── SIM
           │               │
           ▼               ▼
     Aceitar risco    Está no CISA KEV ou
     Registrar        EPSS > 10%?
     Revisar em       │
     90 dias      NÃO─┤─SIM
                   │   │
                   │   ▼
                   │  IMEDIATO — corrigir agora
                   │  Considerar mitigação temporária
                   │  (WAF rule, disable feature, network block)
                   │
                   ▼
              Aplicar a Matriz de Risco
              (Exploitabilidade × Criticidade do ativo)
                   │
                   ▼
              Definir SLA e responsável
              Registrar no Risk Register
              Acompanhar até resolução
```

---

## SLAs de referência por classificação final

| Resultado da Matriz | SLA | Ação |
|---|---|---|
| **CRÍTICO — IMEDIATO** | 0-24 horas | Corrigir agora. Se não houver patch, mitigar. Notificar CISO. |
| **CRÍTICO — 7 dias** | 7 dias corridos | Criar issue P1. Escalar se não for resolvido em 48h. |
| **ALTO — 30 dias** | 30 dias | Issue no backlog com prioridade alta. Revisar semanalmente. |
| **MÉDIO — 60 dias** | 60 dias | Issue no backlog. Revisar nas sprints. |
| **BAIXO — 90 dias** | 90 dias | Registrar. Revisar trimestralmente. |
| **BACKLOG** | Trimestral | Registrar. Reavaliar a cada 90 dias. |

---

## O que fazer quando não existe patch?

Nem sempre há uma versão corrigida disponível. Opções:

**1. Mitigação temporária**
- Bloquear o endpoint afetado via WAF rule
- Desabilitar a feature que usa a lib vulnerável
- Restringir acesso via network policy (só IPs internos)

**2. Substituição da biblioteca**
- Existe uma alternativa que faz o mesmo e não tem a vulnerabilidade?

**3. Aceite de risco formal**
- Documentar o risco, as razões para não corrigir agora, e a data de revisão
- Assinar com o responsável técnico e registrar no Risk Register
- Nunca "fechar" silenciosamente — o risco aceito precisa de revisão periódica

---

## Risk Register — onde registrar

Toda vulnerabilidade que não será corrigida imediatamente deve ir para o **Risk Register** (Registro de Riscos). Campos mínimos:

| Campo | Descrição |
|---|---|
| ID | Identificador único (ex: RISK-2024-042) |
| CVE / Finding | Identificador da vulnerabilidade |
| Sistema afetado | Qual aplicação / componente |
| Criticidade | Resultado da matriz (Crítico / Alto / Médio / Baixo) |
| CVSS | Score oficial |
| EPSS | Probabilidade de exploração |
| Status | Aberto / Em tratamento / Aceito / Fechado |
| SLA | Data limite para resolução |
| Responsável | Quem vai corrigir |
| Mitigação temporária | O que foi feito enquanto o patch não chega |
| Data de revisão | Quando será reavaliado |

---

## Referências

- [NVD — National Vulnerability Database](https://nvd.nist.gov)
- [EPSS — Exploit Prediction Scoring System](https://www.first.org/epss)
- [CISA KEV — Known Exploited Vulnerabilities](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [CVSS v3.1 Specification](https://www.first.org/cvss/specification-document)
- [OWASP Risk Rating Methodology](https://owasp.org/www-community/OWASP_Risk_Rating_Methodology)
- [ISO/IEC 27005:2022 — Gestão de Risco de SI](https://www.iso.org/standard/80585.html)
