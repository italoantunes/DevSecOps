# ✅ Security Checklist — Mapa Completo do Pipeline

> **Para quem é este checklist:**
> Referência completa para onboarding de novos membros, auditoria interna do processo de segurança da squad, ou estudo do fluxo DevSecOps do início ao fim.
>
> **Para uso no dia a dia:** use os checklists focados:
> - Antes de abrir/aprovar um PR → [`pr-security-checklist.md`](./pr-security-checklist.md)
> - Antes de fazer deploy → [`deploy-security-checklist.md`](./deploy-security-checklist.md)

---

## ETAPA 1 — IDE (Desenvolvedor local)

> O custo de correção aqui é zero. O dev vê o problema e corrige na hora.
> Veja mais em: [`/01-ide-precommit`](../01-ide-precommit/)

- [ ] Plugin de SAST instalado e ativo na IDE (SonarLint / Snyk IDE / Semgrep)
- [ ] Nenhum finding CRITICAL ou HIGH ignorado durante o desenvolvimento
- [ ] Nenhuma credencial, token ou API key escrita diretamente no código
- [ ] Dependências verificadas antes do `pip install` / `npm install`
- [ ] Inputs de usuário validados e sanitizados na origem

---

## ETAPA 2 — Pre-commit Hook

> Executa automaticamente antes de cada `git commit`. Bloqueia na raiz.
> Veja mais em: [`/01-ide-precommit`](../01-ide-precommit/)

- [ ] Gitleaks ou detect-secrets configurado como pre-commit hook
- [ ] Nenhum arquivo `.env` com valores reais sendo commitado
- [ ] Arquivos sensíveis adicionados ao `.gitignore`
- [ ] Commit assinado com GPG (quando exigido pela política)

---

## ETAPA 3 — Secret Scanning (CI)

> Varre todo o histórico do repositório a cada push ou PR.
> Veja mais em: [`/02-secret-scan`](../02-secret-scan/)

- [ ] Gitleaks executou sem findings
- [ ] Histórico do git não contém secrets em commits anteriores
- [ ] Variáveis de ambiente configuradas no secret manager do CI/CD (não em plain text)
- [ ] Se um secret foi encontrado: rotacionado imediatamente + histórico limpo

---

## ETAPA 4 — SAST — Static Application Security Testing (CI)

> Analisa o código-fonte escrito pela equipe sem executar a aplicação.
> Veja mais em: [`/03-sast`](../03-sast/)

- [ ] Semgrep ou SonarQube executou no pipeline
- [ ] Nenhum finding CRITICAL no diff do PR
- [ ] Findings HIGH avaliados: corrigidos ou com exceção documentada e aprovada
- [ ] Sem SQL Injection (CWE-89), XSS (CWE-79), Command Injection (CWE-78)
- [ ] Sem criptografia fraca (MD5, SHA1 para senhas)
- [ ] Sem deserialização insegura (CWE-502)

---

## ETAPA 5 — SCA + SBOM — Software Composition Analysis (CI)

> Analisa todas as dependências e bibliotecas, incluindo as transitivas.
> Veja mais em: [`/04-sca-sbom`](../04-sca-sbom/)

- [ ] Trivy ou Snyk executou no pipeline
- [ ] SBOM gerado em CycloneDX ou SPDX e arquivado
- [ ] Nenhuma dependência com CVE CRITICAL sem mitigação documentada
- [ ] Dependências HIGH avaliadas com SLA definido
- [ ] Licenças das dependências compatíveis com a política da empresa
- [ ] Versões fixadas — sem uso de `latest` ou ranges abertos (`^`, `~`)

---

## ETAPA 6 — Container Scan (CI)

> Valida a imagem Docker antes de qualquer push para o registry.
> Veja mais em: [`/05-container-scan`](../05-container-scan/)

- [ ] Trivy executou contra a imagem gerada
- [ ] Imagem base atualizada e sem CVEs CRITICAL
- [ ] Container configurado para rodar como non-root
- [ ] Filesystem read-only onde possível
- [ ] Secrets não passados como variável de ambiente em plain text
- [ ] Imagem não usa tag `latest` — versão fixada (ex: `python:3.11.7-slim`)

---

## ETAPA 7 — IaC Scan — Infrastructure as Code (CI)

> Valida Terraform, Kubernetes YAML, Helm, Dockerfile antes de provisionar.
> Veja mais em: [`/06-iac-security`](../06-iac-security/)

- [ ] Checkov executou sem findings CRITICAL
- [ ] Nenhum Security Group / NACL com `0.0.0.0/0` em portas não justificadas
- [ ] Buckets S3 / Cloud Storage não públicos sem necessidade documentada
- [ ] Criptografia em repouso habilitada em todos os recursos de dados
- [ ] Logs e auditoria habilitados nos novos recursos de cloud
- [ ] IAM roles e policies seguem least privilege

---

## ETAPA 8 — DAST — Dynamic Application Security Testing (Staging)

> Testa a aplicação em execução simulando ataques reais.
> Veja mais em: [`/07-dast`](../07-dast/)

- [ ] OWASP ZAP ou Nuclei executou contra o ambiente de staging
- [ ] Nenhum finding CRITICAL antes de promover para produção
- [ ] Autenticação e autorização dos endpoints validadas
- [ ] Headers de segurança presentes (CSP, X-Frame-Options, HSTS)
- [ ] Rate limiting testado em endpoints críticos
- [ ] OWASP API Top 10 coberto nos endpoints expostos

---

## ETAPA 9 — Deploy para Produção

> Validações finais antes de promover o artefato.

- [ ] Todos os gates de segurança das etapas anteriores passaram
- [ ] Artefato assinado digitalmente (Cosign / SLSA)
- [ ] SBOM arquivado e associado a esta versão
- [ ] Admission Control validou a imagem (OPA Gatekeeper / Kyverno)
- [ ] Plano de rollback documentado e testado
- [ ] Alertas e dashboards configurados para os novos recursos
- [ ] Aprovação do responsável de segurança (quando exigido — PCI-DSS Req. 6)
- [ ] Time de plantão notificado sobre o deploy

---

## Quality Gates — Referência Rápida

| Severidade | Ação | SLA |
|---|---|---|
| 🔴 CRITICAL | Bloqueia automaticamente | Imediato |
| 🟠 HIGH | Bloqueia (maduro) · Alerta com SLA (início) | 7 dias |
| 🟡 MEDIUM | Alerta — registrado no Risk Register | 30 dias |
| 🟢 LOW / INFO | Não bloqueia — revisão trimestral | 90 dias |

Para critérios detalhados de priorização: [`/concepts/vuln-prioritization`](../concepts/vuln-prioritization/)

---

*Referências: OWASP Top 10:2021 · OWASP API Security Top 10:2023 · NIST SP 800-218 · CIS Controls v8 · PCI-DSS v4.0 Req. 6*
