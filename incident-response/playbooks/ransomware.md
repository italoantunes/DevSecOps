# 🚨 Playbook — Ransomware

**Classificação:** CRÍTICO  
**Modelo:** PICERL (Preparation → Identification → Containment → Eradication → Recovery → Lessons Learned)  
**Referência:** NIST SP 800-61 Rev 2 · MITRE ATT&CK T1486

---

## P — Preparation (Pré-requisito)

- Backups offline testados e validados periodicamente
- EDR/XDR com proteção comportamental habilitada
- Segmentação de rede: domínio, servidores, usuários isolados
- Plano de comunicação de crise definido (interno + jurídico + comunicação)
- Contatos de resposta a incidentes externos (MSSP, seguro cyber) mapeados

---

## I — Identification (Detecção)

**Indicadores de comprometimento (IoCs):**
- Arquivos com extensões incomuns em massa (`.locked`, `.enc`, `.crypt`)
- Nota de resgate (`README.txt`, `HOW_TO_DECRYPT.html`) em múltiplos diretórios
- Spike de CPU/I/O em processos desconhecidos
- Comunicação de saída para IPs/domínios não catalogados (C2)
- Alertas de EDR: comportamento de cifragem em massa, shadow copy deletion

**Fontes de detecção:**
- EDR / XDR → behavior-based alert
- SIEM → regra de correlação: alto volume de modificação de arquivos por processo único
- AWS GuardDuty → unusual API calls, data exfiltration findings

**Ação imediata ao confirmar:**
1. Registrar hora exata da detecção e sistema(s) afetado(s)
2. NÃO desligar o sistema ainda — preservar evidências em memória
3. Acionar o time de IR imediatamente

---

## C — Containment (Contenção)

> ⚠️ Nunca pule para eradication sem contenção confirmada.

**Contenção imediata (primeiros 15 minutos):**
- [ ] Isolar host(s) afetados da rede (quarentena via EDR ou desconexão física de switch)
- [ ] Bloquear conta(s) comprometidas no AD / Entra ID
- [ ] Revogar tokens e sessões ativas das contas afetadas
- [ ] Desabilitar compartilhamentos de rede (SMB) nos hosts afetados
- [ ] Bloquear IPs/domínios de C2 identificados no firewall / Security Group

**Contenção secundária:**
- [ ] Identificar lateral movement — verificar outros hosts com o mesmo processo
- [ ] Isolar segmento de rede se o alcance for amplo
- [ ] Desabilitar VPN / acesso remoto temporariamente se vetor de entrada

---

## E — Eradication (Erradicação)

- [ ] Identificar e remover o binário/payload do ransomware
- [ ] Limpar persistências: scheduled tasks, registry run keys, serviços, WMI subscriptions
- [ ] Remover contas backdoor criadas pelo atacante
- [ ] Resetar credenciais de todas as contas expostas (usuário, serviço, admin)
- [ ] Resetar KRBTGT (duas vezes, intervalo de 10h) se AD comprometido
- [ ] Validar integridade dos backups antes de qualquer restore

---

## R — Recovery (Recuperação)

- [ ] Restaurar sistemas a partir de backup limpo validado
- [ ] Reimplantar do zero se o sistema não puder ser confiável (bare metal restore)
- [ ] Validar integridade dos dados restaurados
- [ ] Monitoramento intensivo pós-restore por 72h mínimo
- [ ] Comunicar partes afetadas (regulatório se aplicável — LGPD Art. 48, BACEN)

---

## L — Lessons Learned (Lições Aprendidas)

Reunião post-mortem blameless em até **5 dias úteis** após contenção.

**Perguntas guia:**
- Qual foi o vetor de entrada inicial?
- Quanto tempo entre o comprometimento e a detecção (dwell time)?
- Quais controles falharam ou estavam ausentes?
- O que funcionou bem na resposta?

**Entregáveis:**
- Relatório de incidente com timeline completo
- Atualização do Risk Register
- IoCs compartilhados internamente (STIX/TAXII ou MISP)
- Plano de ação com prazo para cada gap identificado

---

## Contatos de Escalonamento

| Papel | Contato | Quando acionar |
|---|---|---|
| Líder de IR | | Imediatamente ao confirmar |
| CISO | | Sempre em incidente CRÍTICO |
| Jurídico | | Se dados de clientes afetados |
| Comunicação | | Antes de qualquer comunicação externa |
| Seguro Cyber | | Após contenção inicial |
