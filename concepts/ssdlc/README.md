# Conceito — SSDLC (Secure Software Development Lifecycle)

> Segurança não é uma fase do desenvolvimento. É uma prática contínua em todas as fases.

---

## O que é SSDLC?

O **SSDLC** é a integração de práticas e controles de segurança em **cada fase** do ciclo de desenvolvimento de software — desde os requisitos iniciais até a operação em produção.

O modelo tradicional de desenvolvimento (SDLC) tem fases bem definidas: requisitos, design, código, teste, deploy, operação. O SSDLC simplesmente garante que a segurança participa de todas elas, em vez de ser tratada como uma etapa final ou uma auditoria eventual.

**A diferença na prática:**

| Abordagem tradicional | SSDLC |
|---|---|
| Pentest uma vez por ano | Testes de segurança contínuos no CI/CD |
| Segurança revisa após o código pronto | Threat Modeling antes de começar a codificar |
| Vulnerabilidade descoberta em produção | Vulnerabilidade capturada no PR |
| "Isso é problema do time de segurança" | Segurança é responsabilidade de todos |
| Correção custa 100x mais | Correção custa próximo de zero |

---

## As fases e os controles de segurança

```
FASE 1 — REQUISITOS
─────────────────────────────────────────────────────
Atividade: definir o que o sistema deve fazer

Segurança aqui significa:
  → Identificar requisitos de segurança junto com os funcionais
     "O sistema precisa autenticar com MFA"
     "Dados de cartão não podem ser armazenados em plain text"
  → Identificar regulações aplicáveis (PCI-DSS, LGPD, HIPAA)
  → Escrever abuse cases: "o que um atacante tentaria fazer?"

FASE 2 — DESIGN / ARQUITETURA
─────────────────────────────────────────────────────
Atividade: decidir como o sistema vai funcionar

Segurança aqui significa:
  → Threat Modeling (STRIDE/PASTA) — ver /concepts/threat-modeling
  → Revisão de arquitetura com foco em segurança
  → Definir limites de confiança, fluxos de dados sensíveis
  → Decidir mecanismos de autenticação, autorização, criptografia
  → Aplicar Privacy by Design (LGPD/GDPR)

FASE 3 — DESENVOLVIMENTO (CÓDIGO)
─────────────────────────────────────────────────────
Atividade: escrever o código

Segurança aqui significa:
  → Plugins de SAST na IDE (SonarLint, Snyk)
  → Pre-commit hooks (Gitleaks para secrets)
  → Seguir Secure Coding Guidelines
  → Code review com foco em segurança

FASE 4 — TESTE / CI
─────────────────────────────────────────────────────
Atividade: verificar que o código funciona

Segurança aqui significa:
  → SAST no pipeline (Semgrep, SonarQube)
  → Secret Scanning (Gitleaks, TruffleHog)
  → SCA + SBOM (Trivy, Syft)
  → Container Scan (Trivy)
  → IaC Security (Checkov, tfsec)
  → Testes de segurança automatizados

FASE 5 — DEPLOY / CD
─────────────────────────────────────────────────────
Atividade: colocar o código em produção

Segurança aqui significa:
  → DAST contra staging (OWASP ZAP)
  → Assinatura de artefatos (Cosign + SLSA)
  → Admission control no Kubernetes
  → Checklist de deploy (ver /checklists)
  → Aprovação humana para ambientes regulados

FASE 6 — OPERAÇÃO / PRODUÇÃO
─────────────────────────────────────────────────────
Atividade: manter o sistema funcionando

Segurança aqui significa:
  → Runtime Security (Falco)
  → Monitoramento contínuo (SIEM, EDR)
  → Dependency-Track para CVEs pós-deploy
  → Patch management (Dependabot, WSUS)
  → Resposta a incidentes quando algo acontece
```

---

## Por que o SSDLC importa para o negócio?

**Custo de correção:** A IBM Systems Sciences Institute calculou que corrigir uma vulnerabilidade em produção custa até **100 vezes mais** do que corrigir a mesma vulnerabilidade durante o desenvolvimento. O SSDLC é, essencialmente, uma estratégia de redução de custos.

**Velocidade:** Contra-intuitivamente, times com SSDLC maduro lançam software mais rápido. Menos incidentes em produção, menos retrabalho urgente, menos deploys de hotfix às 2 da manhã.

**Compliance:** PCI-DSS, ISO 27001, SOC 2 e LGPD exigem que segurança seja parte do processo de desenvolvimento, não uma adição posterior.

**Confiança:** Clientes e parceiros perguntam sobre práticas de segurança no desenvolvimento. Ter um processo documentado é diferencial competitivo.

---

## Maturidade do SSDLC — onde você está?

O **OWASP SAMM (Software Assurance Maturity Model)** é o framework mais usado para avaliar e evoluir a maturidade de segurança no desenvolvimento:

| Nível | Descrição |
|---|---|
| **1 — Inicial** | Sem processo formal. Segurança é reativa — só age após incidente |
| **2 — Gerenciado** | Alguns controles existem. SAST no CI, patches sendo aplicados |
| **3 — Definido** | Processo formal. Threat Modeling, treinamento, métricas |
| **4 — Quantificado** | Tudo medido. SLAs de vulnerabilidades cumpridos, dados orientam decisões |
| **5 — Otimizado** | Melhoria contínua. Contribui para a comunidade, lidera práticas do setor |

A maioria das empresas começa no nível 1-2. O objetivo realista para a maioria dos times é chegar ao nível 3.

---

## Referências

- [OWASP SAMM v2](https://owaspsamm.org) — avaliação de maturidade
- [NIST SP 800-218 — SSDF](https://csrc.nist.gov/publications/detail/sp/800-218/final) — framework de desenvolvimento seguro do NIST
- [Microsoft SDL](https://www.microsoft.com/en-us/securityengineering/sdl) — Security Development Lifecycle da Microsoft
- [BSIMM](https://www.bsimm.com) — Building Security In Maturity Model
