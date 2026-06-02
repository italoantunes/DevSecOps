# 🚀 Checklist de Segurança — Deploy para Produção

> **Quando usar:** Gate final antes de promover qualquer artefato para produção. Use em conjunto com seu processo de Change Management (CAB, runbook de deploy).
> Para o mapa completo do pipeline de segurança, veja [`security-checklist.md`](./security-checklist.md).

---

## 🔒 Pipeline

- [ ] Todos os stages de segurança passaram (Secret Scan, SAST, SCA, Container Scan, IaC)
- [ ] Nenhum finding CRITICAL em aberto sem exceção documentada e aprovada
- [ ] Artefato assinado digitalmente (Cosign / SLSA)
- [ ] SBOM arquivado junto ao artefato desta versão
- [ ] DAST executou em staging sem findings bloqueantes

---

## ☁️ Cloud & Infraestrutura

- [ ] Security Groups / Firewall Rules revisados — sem portas desnecessárias expostas
- [ ] IAM roles e policies seguem least privilege
- [ ] Logs e CloudTrail / Cloud Audit Logs habilitados nos novos recursos
- [ ] Criptografia em repouso e em trânsito (TLS 1.2+) configurada
- [ ] Secrets armazenados em Secrets Manager — não em variáveis de ambiente plain text

---

## 📱 Se aplicável — Mobile / Container / Serverless

- [ ] Imagem container escaneada e aprovada
- [ ] Lambda / função serverless com permissões mínimas (IAM least privilege)
- [ ] App mobile sem dados sensíveis armazenados localmente sem criptografia

---

## 🔁 Rollback & Resposta

- [ ] Plano de rollback documentado e testado
- [ ] Alertas e dashboards de monitoramento configurados para os novos recursos
- [ ] Time de plantão notificado sobre o deploy
- [ ] Runbook de incidente atualizado se houver novo componente crítico

---

## 📋 Aprovações

| Aprovação | Responsável | Status |
|---|---|---|
| Tech Lead / SRE | | ☐ |
| Segurança (quando aplicável) | | ☐ |
| Change Advisory Board (CAB) — ambientes regulados | | ☐ |
