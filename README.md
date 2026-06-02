# 🔐 DevSecOps — Security Pipeline Hub

> Pipeline de segurança completo: do commit ao runtime.  
> Exemplos de CI/CD, checklists prontos para uso, playbooks de resposta a incidentes e diagramas de arquitetura.

[![OWASP](https://img.shields.io/badge/OWASP-Top_10-000000?style=flat-square&logo=owasp)](https://owasp.org/Top10)
[![NIST](https://img.shields.io/badge/NIST-SP_800--218-1A3A5C?style=flat-square)](https://csrc.nist.gov/publications/detail/sp/800-218/final)
[![CIS](https://img.shields.io/badge/CIS-Controls_v8-2D6A4F?style=flat-square)](https://www.cisecurity.org/controls/v8)
[![MITRE](https://img.shields.io/badge/MITRE-ATT%26CK-FF0000?style=flat-square)](https://attack.mitre.org)

---

## Como funciona a esteira

```
IDE → Pre-commit → [ Secret Scan → SAST → SCA+SBOM → Container → IaC ] → Build → DAST → Produção
       (local)              CI — Continuous Integration                           CD      Runtime
```

Cada etapa tem um **gate de segurança**: findings CRITICAL bloqueiam automaticamente, HIGH alertam com SLA definido. Ninguém precisa rodar nada manualmente — o pipeline dispara a cada push ou Pull Request.

📊 Visualização completa: [`diagrams/devsecops-pipeline-topology.html`](./diagrams/devsecops-pipeline-topology.html)

---

## Estrutura

```
devsecops/
│
├── 📁 pipeline/
│   ├── .github/
│   │   └── dependabot.yml          # Configuração Dependabot (atualização automática de deps)
│   ├── gitlab-ci/
│   │   └── .gitlab-ci.yml          # Template de pipeline para GitLab CI/CD
│   └── github-actions/
│       └── security.yml            # Template de pipeline para GitHub Actions
│
├── 📁 checklists/
│   ├── security-checklist.md       # Mapa completo: IDE → Deploy (9 etapas)
│   ├── pr-security-checklist.md    # Referência rápida para code review / PR
│   └── deploy-security-checklist.md # Gate final antes de promover para produção
│
├── 📁 incident-response/
│   └── playbooks/
│       ├── ransomware.md           # Playbook PICERL — Ransomware
│       └── credential-compromise.md # Playbook PICERL — Credencial comprometida
│
└── 📁 diagrams/
    └── devsecops-pipeline-topology.html  # Topologia interativa da esteira
```

---

## Conceitos Fundamentais

### SSDLC — Secure Software Development Lifecycle

O SSDLC é a integração de práticas de segurança em **cada fase** do ciclo de desenvolvimento — não apenas no final, como um pentest antes do go-live.

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  REQUISITOS │→ │   DESIGN    │→ │   CÓDIGO    │→ │   TESTE     │→ │   DEPLOY    │→ │  OPERAÇÃO   │
│             │  │             │  │             │  │             │  │             │  │             │
│ Req. de     │  │ Threat      │  │ SAST        │  │ DAST        │  │ Sign &      │  │ Runtime     │
│ segurança   │  │ Modeling    │  │ Secret Scan │  │ Pentest     │  │ Verify      │  │ Security    │
│ Abuse cases │  │ Arch Review │  │ SCA / SBOM  │  │ Fuzzing     │  │ IaC Scan    │  │ Falco       │
│ LGPD / PCI  │  │ STRIDE      │  │ Pre-commit  │  │ API Scan    │  │ Admission   │  │ SIEM / SOC  │
└─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘
```

**Referências:** NIST SP 800-218 (SSDF) · OWASP SAMM v2 · Microsoft SDL

---

### Shift-Left Security

Mover os controles de segurança para **mais cedo** no ciclo. Quanto mais tarde uma vulnerabilidade é encontrada, mais caro é corrigir.

```
CUSTO DE CORREÇÃO
     │
 $$$$$│                                                    ██ Produção (100x)
  $$$$│                                          ██ Staging / QA (10x)
   $$$│                               ██ CI Pipeline (5x)
    $$│                    ██ Pre-commit / IDE (2x)
     $│    ██ Requisitos / Design (1x)  ← mais barato
     └──────────────────────────────────────────────────────→ FASE
       Design    Código    Commit    CI/CD    Staging    Prod

                ◄─────────────── SHIFT LEFT ───────────────
```

Na prática: plugin de SAST na IDE, pre-commit com Gitleaks, security gates no pipeline antes do merge, segurança na Definition of Done da squad e Threat Modeling antes de codar.

---

### Threat Modeling

Processo estruturado para identificar ameaças ao sistema **antes de construí-lo**. Responde: *o que pode dar errado? onde? com que impacto?*

#### STRIDE vs PASTA — Comparativo

| Critério | STRIDE | PASTA |
|---|---|---|
| **Origem** | Microsoft | Tony UcedaVelez (2012) |
| **Foco** | Técnico — tipo de ameaça | Negócio — risco real para a empresa |
| **Quando usar** | Design de componentes, APIs, fluxos | Avaliações formais de risco, PCI-DSS |
| **Curva de aprendizado** | Baixa | Alta |
| **Output** | Lista de ameaças por componente | Risk score por cenário de ataque |

#### STRIDE — As 6 categorias

| Letra | Ameaça | Controle |
|---|---|---|
| **S** | Spoofing — fingir ser outro | Autenticação forte, MFA |
| **T** | Tampering — modificar dados | HMAC, assinatura digital |
| **R** | Repudiation — negar uma ação | Logs imutáveis, auditoria |
| **I** | Info Disclosure — expor dados | Criptografia, least privilege |
| **D** | Denial of Service — indisponibilidade | Rate limiting, DDoS protection |
| **E** | Elevation of Privilege — escalar permissões | RBAC, menor privilégio |

#### PASTA — Os 7 estágios

```
1 → Objetivos de negócio e requisitos de segurança
2 → Escopo técnico (componentes, dependências, casos de uso)
3 → Decomposição da aplicação (DFDs, limites de confiança)
4 → Análise de ameaças (TTPs do MITRE ATT&CK)
5 → Análise de vulnerabilidades (SAST, SCA, pentest findings)
6 → Modelagem de ataques (árvores de ataque, cenários realistas)
7 → Análise de risco e impacto ao negócio → plano de mitigação
```

**Recomendação:** use STRIDE no dia a dia para novos features. Use PASTA para avaliações formais e sistemas de alta criticidade.

---

### Repositório de Binários (Artifact Registry)

Armazena e distribui **artefatos de build** (JARs, pacotes npm, binários, charts Helm) de forma centralizada, segura e rastreável.

**Por que é crítico para segurança:**
- Garante que o que foi testado no CI é o que vai para produção (imutabilidade)
- Permite policy de promoção: artefato só avança (dev → staging → prod) se passou nos gates
- Viabiliza SBOM por versão — rastreabilidade de quais libs estão em cada release
- Previne dependency confusion attacks

```
Build CI → Assinar (Cosign) → Push para dev-repo
                ↓  gates passaram
         Promover para staging-repo
                ↓  aprovação + DAST
         Promover para prod-repo → Deploy
```

| Ferramenta | Tipo | Formatos |
|---|---|---|
| **JFrog Artifactory** | Comercial / OSS | Maven, npm, PyPI, Docker, Helm, Go, Terraform |
| **Sonatype Nexus** | Comercial / OSS | Maven, npm, PyPI, Docker, Helm, NuGet |
| **GitHub Packages** | SaaS | npm, Maven, Docker, NuGet |
| **GitLab Package Registry** | SaaS / Self-hosted | npm, Maven, PyPI, Docker, Helm, Go |
| **AWS CodeArtifact** | SaaS (AWS) | npm, Maven, PyPI, NuGet |
| **Google Artifact Registry** | SaaS (GCP) | Docker, Maven, npm, Python, Go |

---

### Repositório de Imagens de Container (Container Registry)

Armazena e distribui **imagens Docker/OCI**. Todo container que vai para produção deve vir de um registry controlado — nunca direto do Docker Hub público sem validação.

**Controles essenciais:**
- **Image signing:** assinar com Cosign (Sigstore) e verificar no admission controller antes do deploy
- **Scanning nativo:** maioria dos registries modernos escaneia automaticamente no push
- **Imutabilidade de tags:** proibir sobrescrever `v1.2.3` — versões semânticas fixas sempre
- **Acesso via OIDC:** push restrito ao CI/CD, pull liberado apenas para workloads específicos

| Ferramenta | Destaques de segurança |
|---|---|
| **Harbor** (CNCF, open-source) | Scanning integrado, RBAC, assinatura, proxy cache |
| **AWS ECR** | Scan nativo (Inspector), lifecycle policies, immutable tags |
| **Google Artifact Registry** | Binary Authorization, CMEK, VPC Service Controls |
| **Azure Container Registry** | Defender for Containers, geo-replication, trust policy |
| **GHCR** (GitHub) | Integrado ao Actions, free para repos públicos |
| **GitLab Container Registry** | Integrado ao CI/CD, scanning via Trivy, cleanup policies |

---

### Dependabot — Atualização Automática de Dependências

Feature nativa do GitHub que monitora dependências e abre PRs automaticamente quando há nova versão ou CVE publicado.

```
GitHub detecta dependência desatualizada ou com CVE
               ↓
     Abre PR automático com a atualização
               ↓
    CI roda os testes na nova versão
               ↓
  Você aprova (ou auto-merge se testes passam)
```

| Modo | Quando usa |
|---|---|
| **Version updates** | Atualiza versões periodicamente conforme schedule |
| **Security updates** | PR imediato ao detectar CVE em uma dependência — automático |

Configuração: [`pipeline/.github/dependabot.yml`](./pipeline/.github/dependabot.yml) — inclui exemplos para GitHub Actions, npm, pip e Docker com comentários explicativos.

---

## Priorização de Vulnerabilidades

Nem todo CVE precisa de correção imediata. Os fatores que definem a prioridade real:

- **CVSS Score** — severidade técnica base (0–10)
- **EPSS Score** — probabilidade de exploração nos próximos 30 dias
- **Exposição** — o ativo está na internet? rede interna? isolado?
- **Criticidade do ativo** — impacto no negócio se comprometido
- **Exploitability** — existe exploit público ou está no CISA KEV?
- **Contexto** — a lib vulnerável está realmente em uso? (reachability)

### Matriz de Risco

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

### Fluxo de Decisão

```
Finding identificado
        │
        ▼
A lib/código vulnerável está realmente em uso? (reachability)
        │ NÃO → Aceitar risco · Registrar · Revisar em 90 dias
        │ SIM
        ▼
Existe exploit público ou consta no CISA KEV? (EPSS > 10%)
        │ NÃO → Aplicar matriz acima para definir SLA
        │ SIM
        ▼
O ativo é exposto à internet ou faz parte de ambiente regulado?
        │ NÃO → ALTO — 7 dias
        │ SIM
        ▼
   ┌──────────┐
   │ CRÍTICO  │ → Correção imediata + notificar segurança
   │ IMEDIATO │ → Mitigação temporária se patch não disponível
   └──────────┘
```

**Fontes para enriquecer a priorização:** [NVD](https://nvd.nist.gov) · [EPSS](https://www.first.org/epss) · [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) · [Exploit-DB](https://exploit-db.com)

---

## 📁 pipeline/ — Como usar os templates de CI/CD

Os arquivos de pipeline são **templates comentados para adaptar** ao repositório da aplicação que você quer proteger.

### `.gitlab-ci.yml` — GitLab CI/CD

1. Copie para a **raiz** do repositório da aplicação
2. Configure as variáveis (`Settings → CI/CD → Variables`): `SONAR_HOST_URL`, `SONAR_TOKEN`, `STAGING_URL`
3. Abra um Merge Request — o pipeline dispara automaticamente

| Stage | Ferramenta | Dispara em |
|---|---|---|
| Secret Scanning | Gitleaks | Todo MR |
| SAST | Semgrep + SonarQube | Todo MR |
| SCA + SBOM | Trivy (gera sbom.json) | Todo MR |
| Container Scan | Trivy | Push na main |
| IaC Scan | Checkov | Todo MR |
| DAST | OWASP ZAP | Push na main |

### `security.yml` — GitHub Actions

1. Crie `.github/workflows/` no repositório da aplicação e copie o arquivo para lá
2. Configure os secrets (`Settings → Secrets → Actions`): `SEMGREP_APP_TOKEN`, `STAGING_URL`
3. Abra um Pull Request — o workflow dispara automaticamente

| Job | Ferramenta | Dispara em |
|---|---|---|
| secret-scan | Gitleaks | Todo PR |
| sast | Semgrep | Todo PR |
| sca-sbom | Trivy (SBOM arquivado) | Todo PR |
| iac-scan | Checkov | Todo PR |
| container-scan | Trivy | Push na main |
| dast | OWASP ZAP | Push na main |

---

## 📁 checklists/ — Como usar

| Arquivo | Quando usar |
|---|---|
| [`security-checklist.md`](./checklists/security-checklist.md) | Mapa completo (9 etapas). Use para onboarding, auditoria do processo ou referência de estudo |
| [`pr-security-checklist.md`](./checklists/pr-security-checklist.md) | Cole no corpo do PR. Gate de segurança antes do merge |
| [`deploy-security-checklist.md`](./checklists/deploy-security-checklist.md) | Gate final antes de promover para produção |

---

## Quality Gates — Referência

| Severidade | Gate | SLA |
|---|---|---|
| 🔴 CRITICAL | Bloqueia automaticamente | Imediato |
| 🟠 HIGH | Bloqueia (maduro) / Alerta com SLA (início) | 7 dias |
| 🟡 MEDIUM | Alerta — Risk Register | 30 dias |
| 🟢 LOW | Não bloqueia — revisão trimestral | 90 dias |

---

## Referências

| Documento | Link |
|---|---|
| OWASP Top 10:2021 | https://owasp.org/Top10 |
| OWASP API Security Top 10:2023 | https://owasp.org/API-Security |
| NIST SP 800-218 (SSDF) | https://csrc.nist.gov/publications/detail/sp/800-218/final |
| NIST SP 800-61 Rev 2 | https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final |
| MITRE ATT&CK | https://attack.mitre.org |
| CIS Controls v8 | https://www.cisecurity.org/controls/v8 |
| SLSA Framework | https://slsa.dev |
| EPSS | https://www.first.org/epss |
| CISA KEV | https://www.cisa.gov/known-exploited-vulnerabilities-catalog |

---

<div align="center">

*Mantido por [@italoantunes](https://github.com/italoantunes)*

</div>
