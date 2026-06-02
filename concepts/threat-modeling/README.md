# Conceito — Threat Modeling

> "Se você não pensa como um atacante, alguém vai pensar por você."

---

## O que é Threat Modeling?

Threat Modeling (Modelagem de Ameaças) é o processo de identificar **o que pode dar errado** em um sistema antes de construí-lo ou após mudanças significativas.

É uma conversa estruturada que responde quatro perguntas fundamentais:

```
1. O que estamos construindo?       → Diagrama do sistema, fluxo de dados
2. O que pode dar errado?           → Identificação de ameaças
3. O que vamos fazer a respeito?    → Controles e mitigações
4. Fizemos um bom trabalho?         → Revisão e validação
```

**Por que fazer antes de codificar?**

Mudar uma decisão de arquitetura depois que o código está pronto é caro e demorado. Mudar um diagrama numa sessão de Threat Modeling custa zero. É muito mais fácil decidir "vamos usar JWT com RS256 e rotação de chaves a cada 24h" antes de implementar do que refatorar um sistema de autenticação em produção.

---

## Quando fazer Threat Modeling?

- **Novo sistema ou produto** — antes de começar a codificar
- **Nova funcionalidade significativa** — login social, pagamento, upload de arquivos
- **Mudança de arquitetura** — migração para cloud, novo serviço externo integrado
- **Periodicamente** — sistemas críticos devem ser revisados anualmente
- **Após um incidente** — entender o que foi explorado e o que pode ser explorado a seguir

---

## STRIDE — Para o dia a dia da squad

O STRIDE é o modelo mais simples e mais usado para Threat Modeling técnico. Criado pela Microsoft, usa 6 categorias de ameaça como checklist.

**As 6 categorias:**

```
S — Spoofing (Falsificação de identidade)
    Pergunta: alguém pode se passar por outro usuário ou sistema?
    Exemplos: roubo de sessão, JWT forjado, DNS spoofing
    Controles: autenticação forte, MFA, assinatura digital

T — Tampering (Adulteração)
    Pergunta: alguém pode modificar dados em trânsito ou em repouso?
    Exemplos: SQL injection que altera dados, man-in-the-middle, modificação de logs
    Controles: HTTPS/TLS, HMAC, assinatura de mensagens, hash de integridade

R — Repudiation (Repúdio)
    Pergunta: alguém pode negar ter realizado uma ação?
    Exemplos: usuário alega não ter feito uma compra, delete sem auditoria
    Controles: logs imutáveis, auditoria, timestamp confiável (NTP), assinatura de transações

I — Information Disclosure (Divulgação de informação)
    Pergunta: dados podem ser expostos a quem não deveria ter acesso?
    Exemplos: mensagem de erro com stack trace, log com senha, S3 público acidentalmente
    Controles: criptografia em repouso e em trânsito, least privilege, sanitização de erros

D — Denial of Service (Negação de serviço)
    Pergunta: o sistema pode ser tornado indisponível?
    Exemplos: flood de requisições, payload gigante, exhaust de memória
    Controles: rate limiting, timeout, circuit breaker, DDoS protection

E — Elevation of Privilege (Escalação de privilégio)
    Pergunta: alguém pode obter mais permissões do que deveria ter?
    Exemplos: IDOR (acessar dados de outro usuário), SSRF para metadata cloud, path traversal
    Controles: RBAC, validação de autorização em cada endpoint, least privilege
```

### Como fazer uma sessão STRIDE

**Duração:** 1-2 horas para uma funcionalidade específica.

**Quem participa:** 1 desenvolvedor, 1 tech lead, e idealmente 1 pessoa de segurança.

**Passo a passo:**

**1. Desenhar o sistema (15 min)**
Faça um diagrama simples mostrando:
- Componentes (frontend, API, banco de dados, serviços externos)
- Fluxos de dados entre eles
- Limites de confiança (onde um usuário não-autenticado pode chegar?)

Não precisa ser bonito — pode ser no quadro branco ou no draw.io.

**2. Aplicar as categorias STRIDE (30-45 min)**
Para cada componente e cada seta do diagrama, pergunte:
- "Alguém poderia **forjar** quem está fazendo esta requisição?"
- "Alguém poderia **adulterar** estes dados?"
- (... e assim por diante para as 6 categorias)

**3. Priorizar e definir controles (15-20 min)**
Para cada ameaça encontrada:
- Qual a probabilidade? (Baixa / Média / Alta)
- Qual o impacto? (Baixo / Médio / Alto)
- Qual o controle? (o que vamos fazer a respeito)
- Quem vai implementar e quando?

**4. Registrar como item de backlog**
Cada ameaça sem controle vira uma história ou task no board.

---

## PASTA — Para avaliações formais de risco

O **PASTA (Process for Attack Simulation and Threat Analysis)** é mais elaborado que o STRIDE. Conecta ameaças técnicas ao impacto real no negócio — útil para apresentar para executivos, auditorias PCI ou avaliações de risco formais.

### Os 7 estágios do PASTA

**Estágio 1 — Objetivos de negócio e segurança**
O que esse sistema representa para o negócio? Quais são as consequências se for comprometido?

*Exemplo:* "Este sistema processa pagamentos de R$ 10M/dia. Um comprometimento afetaria diretamente a receita e acionaria obrigações de notificação do PCI-DSS."

**Estágio 2 — Escopo técnico**
Quais componentes, serviços, dependências e integrações fazem parte do escopo?

*Inclui:* APIs, bancos de dados, filas, serviços externos, CDNs, versões de software.

**Estágio 3 — Decomposição da aplicação**
Criar os Data Flow Diagrams (DFDs):
- Onde os dados entram?
- Onde são processados?
- Onde são armazenados?
- Onde saem?
- Quais são os limites de confiança?

**Estágio 4 — Análise de ameaças**
Usando o MITRE ATT&CK e bases de ameaças conhecidas, identificar os TTPs (táticas, técnicas e procedimentos) que atacantes usariam contra este sistema específico.

**Estágio 5 — Análise de vulnerabilidades**
Cruzar as ameaças do estágio 4 com vulnerabilidades conhecidas (SAST, SCA, pentest findings) para identificar onde há exposição real.

**Estágio 6 — Modelagem de ataques**
Construir "árvores de ataque" — sequências realistas de passos que um atacante seguiria para explorar o sistema, considerando as vulnerabilidades identificadas.

**Estágio 7 — Análise de risco e impacto**
Para cada cenário de ataque:
- Qual a probabilidade de ocorrer?
- Qual o impacto financeiro, reputacional, regulatório?
- Qual o risco residual após os controles planejados?
- Output: plano de mitigação priorizado por risco de negócio

---

## STRIDE vs PASTA — quando usar cada um

| Critério | STRIDE | PASTA |
|---|---|---|
| **Quando usar** | Feature nova, sprint planning, revisão de PR | Novo produto, auditoria PCI, avaliação de risco executiva |
| **Tempo** | 1-2 horas | 2-5 dias (para sistemas complexos) |
| **Quem lidera** | Desenvolvedor ou tech lead | Security Engineer ou consultor |
| **Output** | Lista de ameaças + controles técnicos | Risk register + plano de mitigação com impacto de negócio |
| **Audiência** | Equipe técnica | CTO, CISO, auditores, executivos |
| **Curva de aprendizado** | Baixa — começa em horas | Alta — requer prática e contexto de negócio |

**Recomendação prática:**
- Use STRIDE para cada feature significativa no dia a dia da squad
- Use PASTA quando precisar apresentar risco para liderança ou preparar uma certificação

---

## Referências

- [OWASP Threat Modeling Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html)
- [Microsoft Threat Modeling Tool](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool)
- [STRIDE — Microsoft Learn](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats)
- [PASTA Methodology](https://versprite.com/blog/what-is-pasta-threat-modeling/)
- [MITRE ATT&CK](https://attack.mitre.org) — base de TTPs para o estágio 4 do PASTA
