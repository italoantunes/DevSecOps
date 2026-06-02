# ✅ Checklist de Segurança — Pull Request

> **Quando usar:** Antes de abrir ou aprovar qualquer PR. Cole como template no corpo do PR no GitHub (Settings > General > Pull request template).
> Para o mapa completo do pipeline de segurança, veja [`security-checklist.md`](./security-checklist.md).

---

## 🔑 Segredos e Credenciais

- [ ] Nenhuma credencial, token, API key ou senha hardcoded no código
- [ ] Nenhum arquivo `.env` com valores reais commitado
- [ ] Variáveis sensíveis estão em secret manager (AWS Secrets Manager, Vault, GitLab CI Secrets)
- [ ] O scan do Gitleaks não apontou nenhum finding

---

## 🧪 SAST — Static Application Security Testing

- [ ] Pipeline de SAST executou sem findings CRITICAL
- [ ] Findings HIGH foram avaliados e documentados (aceito com justificativa ou corrigido)
- [ ] Nenhuma injeção (SQL, Command, LDAP) introduzida
- [ ] Inputs do usuário estão sendo validados e sanitizados
- [ ] Nenhuma serialização/deserialização insegura

---

## 📦 SCA + SBOM — Dependências

- [ ] Nenhuma dependência nova com CVE CRITICAL ou HIGH não mitigado
- [ ] SBOM (Software Bill of Materials) gerado e arquivado
- [ ] Licenças das novas dependências são compatíveis com a política da empresa
- [ ] Dependências fixadas em versão específica (sem `latest` ou ranges abertos)

---

## 🌐 Segurança de API e Web

- [ ] Endpoints novos têm autenticação e autorização adequadas
- [ ] Controle de acesso segue o princípio de menor privilégio
- [ ] Dados sensíveis não estão sendo expostos em logs ou respostas de erro
- [ ] Headers de segurança configurados (CORS, CSP, X-Frame-Options)
- [ ] Rate limiting aplicado em endpoints críticos

---

## 🏗️ IaC — Infrastructure as Code

- [ ] Checkov/tfsec executou sem findings CRITICAL
- [ ] Nenhum Security Group com `0.0.0.0/0` em portas não justificadas
- [ ] Buckets S3 / Storage não estão públicos sem necessidade
- [ ] Criptografia em repouso habilitada nos recursos de dados
- [ ] Logs e auditoria habilitados nos novos recursos

---

## 🐳 Containers

- [ ] Imagem base atualizada (não usa `latest`)
- [ ] Container roda como non-root
- [ ] Scan de imagem executou sem CVEs CRITICAL
- [ ] Filesystem read-only onde possível
- [ ] Secrets não passados como variável de ambiente em plain text

---

## 📋 Geral

- [ ] Mudanças revisadas por pelo menos um par técnico
- [ ] Documentação atualizada se houver mudança de comportamento
- [ ] Testes unitários/integração cobrindo o novo código
- [ ] Nenhum código de debug ou `TODO` de segurança pendente
