# 🚀 Checklist de Segurança — Deploy para Produção

> **Quando usar:**
> Gate final obrigatório antes de qualquer promoção de artefato para produção.
> Use junto ao processo de Change Management da sua empresa (CAB, runbook de deploy, approval workflow).
>
> **Regra de ouro:** se qualquer item CRÍTICO estiver marcado como não-OK, o deploy não acontece.
>
> **Checklist completo do pipeline (9 etapas):** [`security-checklist.md`](./security-checklist.md)

---

## 🔒 Gates do Pipeline CI

> Todos os controles automatizados devem ter passado antes de iniciar o deploy.

- [ ] Secret Scanning: Gitleaks executou sem findings
- [ ] SAST: Semgrep / SonarQube executou sem findings CRITICAL
- [ ] SCA: Trivy executou sem CVE CRITICAL em aberto
- [ ] Container Scan: imagem escaneada e aprovada
- [ ] IaC Scan: Checkov executou sem findings CRITICAL
- [ ] DAST: OWASP ZAP executou em staging sem findings CRITICAL bloqueantes
- [ ] SBOM arquivado e associado a esta versão no artefato

---

## ✍️ Artefato e Rastreabilidade

> Garantia de que o que vai para produção é exatamente o que foi testado.

- [ ] Artefato / imagem assinada com Cosign ou equivalente
- [ ] Hash do artefato registrado e verificável
- [ ] Admission Control configurado para verificar assinatura no Kubernetes (se aplicável)
- [ ] Versão do artefato é semântica e imutável (sem uso de tag `latest`)

---

## ☁️ Cloud e Infraestrutura

> Validações de segurança específicas do ambiente de destino.

- [ ] Security Groups / Firewall Rules revisados — sem portas desnecessárias abertas
- [ ] IAM roles com least privilege — sem permissões `*` não justificadas
- [ ] Logs e CloudTrail / Cloud Audit Logs habilitados para os novos recursos
- [ ] Criptografia em repouso e em trânsito (TLS 1.2+) configurada
- [ ] Secrets armazenados em Secrets Manager — não em variáveis de ambiente plain text

---

## 🔁 Rollback e Resposta a Incidentes

> O deploy pode falhar ou revelar um problema. Esteja preparado.

- [ ] Plano de rollback documentado: como voltar para a versão anterior em menos de 15 minutos?
- [ ] Versão anterior verificada e disponível no registry
- [ ] Alertas e dashboards de monitoramento configurados para os novos componentes
- [ ] Time de plantão (on-call) notificado sobre o deploy e janela de monitoramento
- [ ] Runbook de incidente atualizado se houver novo componente crítico

---

## 📋 Aprovações

| Aprovação | Responsável | Status |
|---|---|---|
| Tech Lead / SRE | | ☐ Aprovado |
| Segurança (ambientes regulados ou mudanças de alto risco) | | ☐ Aprovado / N/A |
| Change Advisory Board — CAB (ambientes PCI/SOC2/ISO27001) | | ☐ Aprovado / N/A |

---

## ⚠️ Critérios que bloqueiam o deploy

Qualquer um dos itens abaixo impede o deploy até resolução:

- Finding CRITICAL em aberto sem exceção documentada e aprovada pelo CISO/responsável de segurança
- Artefato sem assinatura válida (quando Cosign está habilitado)
- SBOM não gerado para esta versão
- Plano de rollback não documentado para sistemas críticos (PCI, dados de clientes)
- Aprovação de segurança pendente em ambiente regulado

---

*Referências: PCI-DSS v4.0 Req. 6 e 12 · NIST SP 800-218 · CIS Controls v8 — Control 16*
