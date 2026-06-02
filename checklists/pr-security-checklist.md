# ✅ Checklist de Segurança — Pull Request

> **Quando usar:**
> Antes de abrir ou aprovar qualquer Pull Request em repositórios que afetam ambientes produtivos.
>
> **Como usar no GitHub:**
> Copie o conteúdo para um arquivo `.github/pull_request_template.md` no repositório da aplicação.
> O GitHub preencherá automaticamente o corpo de cada novo PR com este checklist.
>
> **Checklist completo do pipeline (9 etapas):** [`security-checklist.md`](./security-checklist.md)

---

## 🔑 Secrets e Credenciais

> Por que importa: uma senha ou token commitado no código é um incidente de segurança — mesmo que apagado depois, fica no histórico do Git.

- [ ] Nenhuma credencial, token, API key ou senha hardcoded no código
- [ ] Nenhum arquivo `.env` com valores reais commitado
- [ ] Variáveis sensíveis estão em secret manager (AWS Secrets Manager, Vault, CI Secrets)
- [ ] O scan do Gitleaks no pipeline passou sem findings

---

## 🧪 SAST — Análise Estática

> Por que importa: o pipeline de SAST detecta vulnerabilidades no código antes que cheguem à main.

- [ ] Pipeline de SAST executou sem findings CRITICAL
- [ ] Findings HIGH foram avaliados: corrigidos ou documentados com justificativa
- [ ] Nenhuma injeção introduzida (SQL, Command, LDAP)
- [ ] Inputs do usuário estão sendo validados e sanitizados
- [ ] Sem serialização / deserialização insegura

---

## 📦 Dependências (SCA)

> Por que importa: você é responsável pelas CVEs das bibliotecas que adiciona ao projeto.

- [ ] Nenhuma dependência nova com CVE CRITICAL ou HIGH não mitigado
- [ ] SBOM gerado e arquivado no CI
- [ ] Licenças das novas dependências são compatíveis com a política
- [ ] Dependências fixadas em versão específica (sem `latest` ou ranges abertos)

---

## 🌐 API e Segurança Web

> Por que importa: endpoints sem autenticação ou com dados expostos são as vulnerabilidades mais exploradas.

- [ ] Endpoints novos têm autenticação e autorização adequadas
- [ ] Controle de acesso segue menor privilégio — usuário só acessa o que é dele
- [ ] Dados sensíveis não aparecem em logs ou mensagens de erro
- [ ] Headers de segurança configurados (CORS, CSP, X-Frame-Options)
- [ ] Rate limiting aplicado em endpoints de autenticação e ações críticas

---

## 🏗️ IaC — Infraestrutura como Código

> Por que importa: uma linha errada no Terraform pode expor um banco de dados para a internet.

- [ ] Checkov executou sem findings CRITICAL
- [ ] Nenhum Security Group com `0.0.0.0/0` em portas não justificadas
- [ ] Recursos de dados com criptografia em repouso habilitada
- [ ] Logs e auditoria habilitados nos novos recursos

---

## 🐳 Containers

> Por que importa: container mal configurado amplifica o impacto de qualquer exploração.

- [ ] Imagem base com versão fixada (não usa `latest`)
- [ ] Container configurado para rodar como non-root
- [ ] Scan da imagem executou sem CVEs CRITICAL
- [ ] Secrets não passados como variável de ambiente em plain text

---

## 📋 Revisão Geral

- [ ] Mudanças revisadas por pelo menos um par técnico
- [ ] Documentação atualizada se houver mudança de comportamento ou interface
- [ ] Testes cobrindo os novos caminhos de código
- [ ] Sem código de debug ou comentários `// TODO: fix security` pendentes
