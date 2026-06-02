# ✅ Security Checklist — DevSecOps
### Do código ao deploy: validações de segurança em cada etapa

> Versão: 1.0 | Referência: OWASP Top 10 · NIST SP 800-218 (SSDF) · CIS Controls v8

---

## 🖥️ ETAPA 1 — IDE (Desenvolvimento local)

> O dev valida antes mesmo de commitar. Custo de correção aqui é zero.

- [ ] Plugin de SAST (SonarLint / Snyk IDE / Semgrep) instalado e ativo na IDE
- [ ] Nenhum finding CRITICAL ou HIGH ignorado durante o desenvolvimento
- [ ] Nenhuma credencial, token ou API key escrita no código (hardcoded)
- [ ] Dependências adicionadas verificadas antes do `pip install` / `npm install`
- [ ] Inputs de usuário validados e sanitizados na origem

---

## 🪝 ETAPA 2 — Pre-commit Hook

> Executa automaticamente antes de cada `git commit`. Bloqueia problemas na raiz.

- [ ] Gitleaks ou detect-secrets configurado como pre-commit hook
- [ ] Nenhum arquivo `.env` com valores reais sendo commitado
- [ ] Arquivos de configuração sensíveis adicionados ao `.gitignore`
- [ ] Commit assinado com GPG (quando exigido pela política)

---

## 🔑 ETAPA 3 — Secret Scanning (CI)

> Varre todo o histórico do repositório a cada push ou PR.

- [ ] TruffleHog / Gitleaks executou sem findings
- [ ] Histórico do git não contém secrets expostos em commits anteriores
- [ ] Variáveis de ambiente configuradas no secret manager do CI/CD (não em plain text)

---

## 🧪 ETAPA 4 — SAST — Static Application Security Testing (CI)

> Analisa o código fonte escrito pela equipe.

- [ ] Semgrep / SonarQube executou no pipeline
- [ ] Nenhum finding CRITICAL no diff do PR
- [ ] Findings HIGH avaliados: corrigidos ou com exceção documentada e aprovada
- [ ] Sem SQL Injection (CWE-89), XSS (CWE-79), Command Injection (CWE-78)
- [ ] Sem deserialização insegura (CWE-502)
- [ ] Sem criptografia fraca ou funções deprecated (MD5, SHA1 para senhas)

---

## 📦 ETAPA 5 — SCA + SBOM — Software Composition Analysis (CI)

> Analisa todas as dependências e bibliotecas do projeto, incluindo as transitivas.

- [ ] Trivy / Snyk / Grype executou no pipeline
- [ ] SBOM (Software Bill of Materials) gerado em CycloneDX ou SPDX e arquivado
- [ ] Nenhuma dependência com CVE CRITICAL sem mitigação
- [ ] Dependências HIGH avaliadas com SLA definido
- [ ] Licenças das dependências compatíveis com política da empresa
- [ ] Versões fixadas — sem uso de `latest` ou ranges abertos (`^`, `~`)

---

## 🐳 ETAPA 6 — Container Scan (CI)

> Valida a imagem Docker antes de qualquer deploy.

- [ ] Trivy / Clair executou contra a imagem gerada
- [ ] Imagem base atualizada e sem CVEs CRITICAL
- [ ] Container configurado para rodar como non-root
- [ ] Filesystem read-only onde possível
- [ ] Secrets não passados como variável de ambiente em plain text
- [ ] Imagem não usa tag `latest` — versão fixada

---

## 🏗️ ETAPA 7 — IaC Scan — Infrastructure as Code (CI)

> Valida Terraform, Kubernetes YAML, Helm, Dockerfile antes de provisionar.

- [ ] Checkov / tfsec executou sem findings CRITICAL
- [ ] Nenhum Security Group / NACL com `0.0.0.0/0` em portas não justificadas
- [ ] Buckets S3 / Cloud Storage não públicos sem necessidade documentada
- [ ] Criptografia em repouso habilitada em todos os recursos de dados
- [ ] Logs e auditoria habilitados nos novos recursos de cloud
- [ ] IAM roles e policies seguem least privilege

---

## 💥 ETAPA 8 — DAST — Dynamic Application Security Testing (CD / Staging)

> Testa a aplicação em execução simulando ataques reais.

- [ ] OWASP ZAP / Burp Enterprise executou contra o ambiente de staging
- [ ] Nenhum finding CRITICAL antes de promover para produção
- [ ] Autenticação e autorização dos endpoints validadas
- [ ] Headers de segurança presentes (CSP, X-Frame-Options, HSTS)
- [ ] Rate limiting testado em endpoints críticos
- [ ] OWASP API Top 10 coberto nos endpoints expostos

---

## 🚀 ETAPA 9 — Deploy para Produção

> Validações finais antes de promover o artefato.

- [ ] Todos os gates de segurança das etapas anteriores passaram
- [ ] Artefato assinado digitalmente (Cosign / SLSA)
- [ ] SBOM arquivado e associado a esta versão no Dependency-Track
- [ ] Admission Control validou a imagem (OPA Gatekeeper / Kyverno)
- [ ] Plano de rollback documentado e testado
- [ ] Alertas e dashboards configurados para os novos recursos
- [ ] Aprovação do responsável de segurança (quando exigido — PCI-DSS Req. 6)
- [ ] Time de plantão notificado sobre o deploy

---

## 📊 Quality Gates — Referência Rápida

| Severidade | Ação | SLA |
|---|---|---|
| 🔴 CRITICAL | Bloqueia automaticamente — sem exceção sem CISO | Imediato |
| 🟠 HIGH | Bloqueia (ambiente maduro) · Alerta com SLA (início) | 7 dias |
| 🟡 MEDIUM | Alerta — registrado no Risk Register | 30 dias |
| 🟢 LOW / INFO | Não bloqueia — revisão trimestral | 90 dias |

---

*Referências: OWASP Top 10:2021 · OWASP API Security Top 10:2023 · NIST SP 800-218 · CIS Controls v8 · PCI-DSS v4.0 Req. 6*
