# 🔐 DevSecOps — Security Pipeline Hub

> Segurança integrada em cada etapa do desenvolvimento: do primeiro caractere digitado na IDE até a aplicação rodando em produção.

[![OWASP](https://img.shields.io/badge/OWASP-Top_10-000000?style=flat-square&logo=owasp)](https://owasp.org/Top10)
[![NIST](https://img.shields.io/badge/NIST-SP_800--218-1A3A5C?style=flat-square)](https://csrc.nist.gov/publications/detail/sp/800-218/final)
[![CIS](https://img.shields.io/badge/CIS-Controls_v8-2D6A4F?style=flat-square)](https://www.cisecurity.org/controls/v8)
[![MITRE](https://img.shields.io/badge/MITRE-ATT%26CK-FF0000?style=flat-square)](https://attack.mitre.org)

---

## O Fluxo Completo

```
  DEV LOCAL            CI — roda a cada Push / Pull Request             CD / PRODUÇÃO
  ─────────────────    ─────────────────────────────────────────────    ──────────────

  [01]          [02]      [03]    [04]       [05]       [06]      [07]     [08]
  IDE      →  Secret  →  SAST  →  SCA   →  Container →  IaC  →  DAST  →  Runtime
  Pre-commit   Scan              SBOM       Scan        Scan     Scan      Monit.
```

Cada etapa tem um **gate de segurança**:
- 🔴 **CRITICAL** — bloqueia o pipeline automaticamente
- 🟠 **HIGH** — alerta com prazo de 7 dias para correção
- 🟡 **MEDIUM** — registrado para correção em até 30 dias
- 🟢 **LOW** — não bloqueia, revisão trimestral

📊 Visualização interativa completa: [`diagrams/devsecops-pipeline-topology.html`](./diagrams/devsecops-pipeline-topology.html)

---

## Etapas do Pipeline

| # | Etapa | O que protege | Pasta |
|---|---|---|---|
| 01 | IDE + Pre-commit | Captura problemas antes do commit | [→ 01-ide-precommit](./01-ide-precommit/) |
| 02 | Secret Scanning | Senhas e tokens expostos no código | [→ 02-secret-scan](./02-secret-scan/) |
| 03 | SAST | Vulnerabilidades no código que você escreveu | [→ 03-sast](./03-sast/) |
| 04 | SCA + SBOM | Vulnerabilidades nas bibliotecas que você usa | [→ 04-sca-sbom](./04-sca-sbom/) |
| 05 | Container Scan | Segurança da imagem Docker | [→ 05-container-scan](./05-container-scan/) |
| 06 | IaC Security | Configurações erradas na infraestrutura | [→ 06-iac-security](./06-iac-security/) |
| 07 | DAST | Ataques reais contra a aplicação rodando | [→ 07-dast](./07-dast/) |
| 08 | Runtime | Monitoramento e detecção em produção | [→ 08-runtime](./08-runtime/) |

---

## Conceitos Fundamentais

Antes de mergulhar nas ferramentas, estes conceitos explicam o **porquê** de cada decisão:

| Conceito | Descrição | Pasta |
|---|---|---|
| SSDLC | Segurança em cada fase do desenvolvimento | [→ concepts/ssdlc](./concepts/ssdlc/) |
| Shift-Left | Por que corrigir cedo é mais barato | [→ concepts/shift-left](./concepts/shift-left/) |
| Threat Modeling | Identificar ameaças antes de construir | [→ concepts/threat-modeling](./concepts/threat-modeling/) |
| Artifact Registry | Repositório seguro de binários e pacotes | [→ concepts/artifact-registry](./concepts/artifact-registry/) |
| Container Registry | Repositório seguro de imagens Docker | [→ concepts/container-registry](./concepts/container-registry/) |
| Priorização de Vulnerabilidades | Como decidir o que corrigir primeiro | [→ concepts/vuln-prioritization](./concepts/vuln-prioritization/) |

---

## Templates e Checklists

### Pipeline CI/CD

Templates prontos para adaptar ao seu repositório:

| Arquivo | Para quem | Pasta |
|---|---|---|
| `security.yml` | Repositórios no **GitHub** | [→ pipeline/github-actions](./pipeline/github-actions/) |
| `.gitlab-ci.yml` | Repositórios no **GitLab** | [→ pipeline/gitlab-ci](./pipeline/gitlab-ci/) |
| `dependabot.yml` | Atualização automática de dependências no GitHub | [→ pipeline/.github](./pipeline/.github/) |

### Checklists

| Arquivo | Quando usar |
|---|---|
| [`security-checklist.md`](./checklists/security-checklist.md) | Referência completa — onboarding, auditoria, estudo |
| [`pr-security-checklist.md`](./checklists/pr-security-checklist.md) | Antes de abrir ou aprovar um Pull Request |
| [`deploy-security-checklist.md`](./checklists/deploy-security-checklist.md) | Antes de qualquer deploy em produção |

---

## Referências

| Documento | Link |
|---|---|
| OWASP Top 10:2021 | https://owasp.org/Top10 |
| OWASP API Security Top 10:2023 | https://owasp.org/API-Security |
| NIST SP 800-218 — SSDF | https://csrc.nist.gov/publications/detail/sp/800-218/final |
| MITRE ATT&CK | https://attack.mitre.org |
| CIS Controls v8 | https://www.cisecurity.org/controls/v8 |
| SLSA Framework | https://slsa.dev |
| EPSS — Probabilidade de exploração | https://www.first.org/epss |
| CISA KEV — Vulnerabilidades exploradas ativamente | https://www.cisa.gov/known-exploited-vulnerabilities-catalog |

---

<div align="center">

*Mantido por [@italoantunes](https://github.com/italoantunes)*

</div>
