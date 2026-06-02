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
│   ├── gitlab-ci/
│   │   └── .gitlab-ci.yml          # Pipeline completo para GitLab CI/CD
│   └── github-actions/
│       └── security.yml            # Pipeline completo para GitHub Actions
│
├── 📁 checklists/
│   └── security-checklist.md       # Checklist unificado: IDE → Deploy
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

## 📁 pipeline/ — Como usar os exemplos de CI/CD

Os arquivos de pipeline são **exemplos prontos para adaptar** ao seu repositório. Eles não executam nada aqui — precisam ser copiados para o repositório da aplicação que você quer proteger.

### `.gitlab-ci.yml` — GitLab CI/CD

**O que é:** arquivo de configuração do GitLab. Quando presente na raiz de um repositório, o GitLab lê e executa automaticamente a cada push ou Merge Request — sem nenhuma ação manual.

**Como usar:**
1. Copie o arquivo `.gitlab-ci.yml` para a **raiz** do repositório da sua aplicação
2. Configure as variáveis de ambiente no GitLab (`Settings → CI/CD → Variables`):
   - `SONAR_HOST_URL` e `SONAR_TOKEN` para o SonarQube
   - `STAGING_URL` para o scan DAST
3. Abra um Merge Request — o pipeline dispara automaticamente

**O que executa em sequência:**
| Stage | Ferramenta | Dispara em |
|---|---|---|
| Secret Scanning | Gitleaks | Todo PR |
| SAST | Semgrep + SonarQube | Todo PR |
| SCA + SBOM | Trivy (gera sbom.json) | Todo PR |
| Container Scan | Trivy | Push na main |
| IaC Scan | Checkov | Todo PR |
| DAST | OWASP ZAP | Push na main (staging) |

---

### `security.yml` — GitHub Actions

**O que é:** arquivo de workflow do GitHub Actions. Fica em `.github/workflows/security.yml` no repositório da aplicação. O GitHub executa automaticamente nos eventos configurados (pull_request, push).

**Como usar:**
1. No repositório da aplicação, crie a pasta `.github/workflows/`
2. Copie o arquivo `security.yml` para dentro dessa pasta
3. Configure os secrets no GitHub (`Settings → Secrets and variables → Actions`):
   - `SEMGREP_APP_TOKEN` para o Semgrep
   - `STAGING_URL` para o DAST
4. Abra um Pull Request — o workflow dispara automaticamente

**O que executa:**

| Job | Ferramenta | Dispara em |
|---|---|---|
| secret-scan | Gitleaks | Todo PR |
| sast | Semgrep | Todo PR |
| sca-sbom | Trivy (gera + arquiva SBOM) | Todo PR |
| iac-scan | Checkov | Todo PR |
| container-scan | Trivy | Push na main |
| dast | OWASP ZAP | Push na main (staging) |

---

## 📁 checklists/ — Como usar

[`security-checklist.md`](./checklists/security-checklist.md) é o checklist unificado de segurança cobrindo todo o ciclo — da IDE ao deploy em produção. Uso recomendado:

- **Code Review / PR:** executar as etapas 1 a 7 antes de aprovar o merge
- **Deploy:** executar as etapas 8 e 9 antes de promover para produção
- **Onboarding de squad:** referência para novos membros entenderem os gates de segurança

> 💾 [Download do checklist](./checklists/security-checklist.md) — copie e use como base para o processo da sua squad.

---

## 📁 incident-response/playbooks/ — Como usar

Playbooks seguindo o modelo **PICERL** (Preparation → Identification → Containment → Eradication → Recovery → Lessons Learned), referenciado no NIST SP 800-61 Rev 2.

| Playbook | Quando usar |
|---|---|
| [`ransomware.md`](./incident-response/playbooks/ransomware.md) | Detecção de cifragem de arquivos em massa, nota de resgate, comportamento de ransomware |
| [`credential-compromise.md`](./incident-response/playbooks/credential-compromise.md) | Login suspeito, MFA bypass, account takeover, credencial exposta |

> ⚠️ Nunca pule para eradication sem contenção confirmada.

---

## 📁 diagrams/

[`devsecops-pipeline-topology.html`](./diagrams/devsecops-pipeline-topology.html) — topologia visual completa da esteira DevSecOps com zonas de segurança, gates e atuação do time de segurança em cada etapa. Abra no browser para visualizar.

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

---

<div align="center">

*Mantido por [@italoantunes](https://github.com/italoantunes)*

</div>
