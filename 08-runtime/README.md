# 08 — Runtime Security

> **Posição no pipeline:** Produção — monitoramento contínuo após o deploy.  
> **O que protege:** Comportamentos anômalos, ataques em andamento e comprometimentos que passaram por todas as etapas anteriores.

---

## Por que o trabalho não termina no deploy?

Todas as etapas anteriores reduzem drasticamente o risco, mas nenhuma elimina 100%. Além disso:

- Vulnerabilidades são descobertas continuamente — uma lib segura hoje pode ter uma CVE amanhã
- Zero-days: vulnerabilidades ainda sem patch disponível
- Configurações manuais erradas que não passaram pelo pipeline
- Ameaças internas (insider threats)
- Ataques sofisticados que burlam os controles preventivos

O runtime security é a **última linha de defesa**: detectar e responder quando algo acontece em produção.

---

## Três pilares do Runtime Security

```
┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
│      DETECTAR       │  │      RESPONDER       │  │      RECUPERAR      │
│                     │  │                      │  │                     │
│ Monitorar o que     │  │ Isolar, bloquear,    │  │ Restaurar operação  │
│ está acontecendo    │  │ investigar           │  │ normal com segurança│
│                     │  │                      │  │                     │
│ Falco, SIEM,        │  │ Playbooks, SOAR,     │  │ Backup, RCA,        │
│ EDR, Wazuh          │  │ alertas automáticos  │  │ lições aprendidas   │
└─────────────────────┘  └─────────────────────┘  └─────────────────────┘
```

---

## Ferramentas

### Falco — Runtime Security para Containers e Kubernetes

O Falco é um projeto CNCF (Cloud Native Computing Foundation) que monitora o comportamento de containers em tempo real usando o kernel do Linux.

**Como funciona:** O Falco fica observando as chamadas de sistema (syscalls) feitas pelos containers. Se um container faz algo fora do padrão esperado — abrir um shell, ler arquivos de senhas, fazer uma conexão de rede inesperada — o Falco dispara um alerta.

**Exemplos de comportamentos que o Falco detecta:**

```yaml
# Um container que nunca deveria abrir um shell bash:
- rule: Shell spawned in container
  desc: Um shell foi aberto dentro de um container
  condition: spawned_process and container and shell_procs
  output: "Shell aberto em container (user=%user.name container=%container.name)"
  priority: WARNING

# Leitura de arquivos sensíveis:
- rule: Read sensitive file
  desc: Tentativa de ler /etc/shadow ou /etc/passwd
  condition: open_read and sensitive_files
  output: "Arquivo sensível lido (file=%fd.name user=%user.name)"
  priority: ERROR

# Container fazendo conexão para IP externo inesperado:
- rule: Unexpected outbound connection
  desc: Conexão de saída para IP não autorizado
  condition: outbound and not expected_outbound_destinations
  output: "Conexão suspeita (dest=%fd.rip container=%container.name)"
  priority: CRITICAL
```

**Instalação via Helm (Kubernetes):**
```bash
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm install falco falcosecurity/falco \
  --set driver.kind=ebpf \
  --set falcosidekick.enabled=true \
  --set falcosidekick.config.slack.webhookurl="https://hooks.slack.com/..."
```

---

### Dependency-Track — Monitoramento contínuo de CVEs

Enquanto o Trivy escaneia no momento do deploy, o **Dependency-Track** monitora continuamente o SBOM gerado. Quando uma nova CVE é publicada para uma biblioteca que está no seu SBOM, você recebe alerta automaticamente.

**Fluxo:**
```
CI gera SBOM (Trivy/Syft) → envia para Dependency-Track
                                    │
                          ┌─────────┴──────────┐
                          │ monitora NVD/OSV   │
                          │ 24 horas por dia   │
                          └─────────┬──────────┘
                                    │
                    Nova CVE publicada para sua dependência?
                                    │
                                    ▼
                          Alerta + issue automático
```

---

### SIEM — Correlação de eventos de segurança

O **SIEM (Security Information and Event Management)** coleta logs de todas as fontes — servidores, containers, cloud, aplicação, firewall — e correlaciona eventos para detectar padrões de ataque.

**Fontes de log mais importantes:**
- Logs de aplicação (erros 401, 403, 500 em massa)
- Logs de container (Falco, Kubernetes audit logs)
- Logs de cloud (AWS CloudTrail, GCP Cloud Audit Logs, Azure Activity Log)
- Logs de acesso (NGINX, ALB, API Gateway)
- Logs de autenticação

**Ferramentas:**

| Ferramenta | Tipo | Observação |
|---|---|---|
| **Microsoft Sentinel** | SaaS (Azure) | Integração nativa com ecossistema Microsoft |
| **Splunk** | Comercial | Mercado enterprise, SPL como linguagem de query |
| **Elastic SIEM** | Open-source / SaaS | EQL (Event Query Language), boa relação custo-benefício |
| **Wazuh** | Open-source | HIDS, log analysis, muito usado em ambientes on-premise |

---

### EDR — Endpoint Detection and Response

Para servidores (não containers), o EDR monitora processos, conexões de rede, modificações de arquivos e comportamentos suspeitos:

| Ferramenta | Tipo |
|---|---|
| **CrowdStrike Falcon** | Comercial — padrão de mercado enterprise |
| **Microsoft Defender for Endpoint** | Comercial — integrado ao ecossistema Azure |
| **SentinelOne** | Comercial — forte em detecção comportamental |
| **Wazuh** | Open-source — boa alternativa gratuita |

---

## Alertas e métricas essenciais

Métricas que devem ter alerta configurado:

```
🔴 CRÍTICO — alertar imediatamente
  - Falco: CRITICAL rule disparado em qualquer container de produção
  - Login com credencial de serviço fora do horário esperado
  - Pico anormal de erros 5xx (pode indicar exploração)
  - Novo processo ou conexão não esperada em container

🟠 ALTO — alertar em minutos
  - Múltiplas tentativas de login falhadas (brute force)
  - Acesso a endpoints administrativos por IP externo
  - Deploy em horário incomum (fora do horário de trabalho)
  - CVE CRITICAL detectada pelo Dependency-Track

🟡 MÉDIO — notificar no dia
  - CVE HIGH detectada pelo Dependency-Track
  - Uso incomum de recursos (CPU/memória) que pode indicar cryptominer
  - Mudanças de configuração não rastreadas pelo IaC
```

---

## Ganhos desta etapa

- **Detecção de ataques em andamento** — não apenas prevenção, mas resposta ativa
- **Visibilidade em produção** — você sabe o que está acontecendo nos seus sistemas
- **Resposta a CVEs pós-deploy** — Dependency-Track avisa quando uma nova vulnerabilidade afeta o que está em produção
- **Conformidade** — PCI-DSS Req. 10 exige logging e monitoramento de acesso ao ambiente

---

## Referências

- [Falco](https://falco.org)
- [Dependency-Track](https://dependencytrack.org)
- [Wazuh](https://wazuh.com)
- [NIST SP 800-61 Rev 2 — Incident Handling Guide](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)
- [MITRE ATT&CK — Tactics in production](https://attack.mitre.org)
- [CIS Controls v8 — Control 8: Audit Log Management](https://www.cisecurity.org/controls/v8)
