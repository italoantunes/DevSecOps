# 🚀 Checklist de Segurança — Deploy para Produção

> Validações obrigatórias antes de qualquer deploy em ambiente produtivo.

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

## 📱 Container / Serverless

- [ ] Imagem container escaneada e aprovada
- [ ] Lambda / função serverless com permissões mínimas (IAM least privilege)

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
