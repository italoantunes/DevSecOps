# 🚨 Playbook — Comprometimento de Credenciais

**Classificação:** ALTO / CRÍTICO  
**Modelo:** PICERL  
**Referência:** NIST SP 800-61 Rev 2 · MITRE ATT&CK T1078 (Valid Accounts)

---

## I — Identification (Detecção)

**Indicadores:**
- Login de localização geográfica incomum (impossível travel)
- Múltiplas falhas de autenticação seguidas de sucesso (brute force → acesso)
- Acesso fora do horário habitual do usuário
- MFA bypass ou solicitações de MFA não iniciadas pelo usuário (MFA fatigue)
- Enumeração de recursos cloud (API calls incomuns no CloudTrail / Audit Logs)
- Alertas do Microsoft Defender for Identity ou GuardDuty IAM

---

## C — Containment

- [ ] Revogar todas as sessões ativas da conta (Azure: Sign-out all sessions / AWS: revoke tokens)
- [ ] Desabilitar a conta temporariamente
- [ ] Revogar e rotacionar API keys / access keys afetadas
- [ ] Bloquear IPs de origem suspeita no firewall / Conditional Access
- [ ] Habilitar MFA obrigatório se ainda não estava ativo
- [ ] Notificar o usuário legítimo

---

## E — Eradication

- [ ] Auditar todas as ações realizadas pela conta comprometida (CloudTrail, Audit Logs, SIEM)
- [ ] Verificar criação de novos usuários, roles ou permissões pelo atacante
- [ ] Remover recursos criados pelo atacante (EC2, Lambda, usuários IAM)
- [ ] Resetar senha + rotacionar todos os tokens/keys associados
- [ ] Verificar se houve exfiltração de dados (S3 GetObject, downloads, e-mails)

---

## R — Recovery

- [ ] Reabilitar conta com nova senha forte + MFA obrigatório
- [ ] Revisar e restringir permissões IAM (least privilege audit)
- [ ] Monitoramento intensivo da conta por 30 dias
- [ ] Notificação regulatória se dados sensíveis foram acessados (LGPD / BACEN)

---

## L — Lessons Learned

- Como a credencial foi comprometida? (phishing, password spray, leak, insider)
- MFA estava habilitado? Se sim, como foi bypassado?
- Qual o tempo de detecção após o comprometimento?
- Quais dados/recursos foram acessados indevidamente?
