# 07 — DAST — Dynamic Application Security Testing

> **Posição no pipeline:** CD — roda contra o ambiente de staging, após o deploy e antes de ir para produção.  
> **O que protege:** Vulnerabilidades que só aparecem quando a aplicação está rodando — lógica de negócio, autenticação, autorização, comportamento em tempo real.

---

## SAST vs DAST — qual a diferença?

Esta é a pergunta mais comum, e entender a diferença é essencial para saber quando usar cada um.

```
SAST — lê o código sem executar           DAST — ataca a aplicação rodando
─────────────────────────────────         ──────────────────────────────────────

Analisa o código-fonte                    Não precisa do código-fonte
Roda em Pull Request (cedo)               Roda após o deploy (staging)
Encontra erros de implementação           Encontra vulnerabilidades de runtime
Ex: SQL concat sem parametrização         Ex: a autenticação realmente pode ser bypassada?
Pode ter falsos positivos                 Resultados refletem o comportamento real
Não entende contexto de negócio           Testa o que o atacante veria
```

**A metáfora:** SAST é o arquiteto revisando a planta do prédio antes da construção. DAST é o engenheiro testando as portas, janelas e fechaduras depois que o prédio está construído.

Ambos são necessários — eles se complementam.

---

## O que o DAST testa?

O DAST simula o que um atacante faria contra sua aplicação:

- **Autenticação:** é possível acessar recursos sem fazer login? O logout realmente invalida a sessão?
- **Autorização:** o usuário A consegue acessar dados do usuário B? (IDOR — Insecure Direct Object Reference)
- **Injeção:** SQL, LDAP, Command Injection na aplicação em execução
- **XSS:** scripts maliciosos são refletidos ou armazenados na aplicação?
- **Headers de segurança:** a aplicação retorna `Content-Security-Policy`, `X-Frame-Options`, `HSTS`?
- **Configuração do servidor:** erros expostos, diretórios acessíveis, métodos HTTP desnecessários habilitados?
- **Rate limiting:** é possível fazer brute force em endpoints de login?

---

## Ferramentas

### OWASP ZAP (Zed Attack Proxy)

A ferramenta DAST open-source mais usada no mundo. Mantida pela OWASP, gratuita, com modo automatizado para CI/CD.

**Dois modos de operação no CI:**

**1. Baseline Scan** — rápido, não invasivo. Faz spider na aplicação e roda checks passivos. Bom para rodar em todo PR:
```bash
docker run -t ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t https://staging.suaapp.com \
  -r zap-report.html
```

**2. Full Scan** — mais demorado, testa ativamente vulnerabilidades. Roda contra staging antes de ir para produção:
```bash
docker run -t ghcr.io/zaproxy/zaproxy:stable \
  zap-full-scan.py \
  -t https://staging.suaapp.com \
  -r zap-report-full.html \
  -l WARN
```

**Resultado:** relatório HTML com as vulnerabilidades encontradas, classificadas por severidade e com a descrição de como corrigir.

**Importante:** O ZAP gera alguns falsos positivos. Os findings precisam de revisão manual — não trate tudo como bloqueante automaticamente. Configure o `fail_action: false` no início e vá ajustando com o tempo.

---

### Nuclei

Ferramenta mais moderna, baseada em **templates**. Extremamente rápida e com uma biblioteca enorme de templates da comunidade para vulnerabilidades específicas.

```bash
# Instala
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest

# Roda templates de severidade crítica e alta
nuclei -u https://staging.suaapp.com -severity critical,high

# Roda apenas checks de misconfiguration
nuclei -u https://staging.suaapp.com -tags misconfiguration

# Roda templates de exposição de dados
nuclei -u https://staging.suaapp.com -tags exposure

# Gera relatório em JSON
nuclei -u https://staging.suaapp.com -json-export resultados.json
```

O grande diferencial do Nuclei é a velocidade e a especificidade — você pode focar nos tipos de vulnerabilidade mais relevantes para sua aplicação.

---

## Onde o DAST roda?

**Nunca em produção.** O DAST faz requisições que simulam ataques — isso pode:
- Criar dados de lixo no banco de dados
- Derrubar funcionalidades frágeis
- Gerar alertas falsos no SIEM
- Em casos extremos, explorar uma vulnerabilidade real

**O ambiente correto é o staging** — um ambiente igual ao de produção, mas isolado:

```
Developer → PR → CI (SAST/SCA/IaC) → merge main → deploy staging → DAST → aprovação → deploy produção
```

---

## Integrando no pipeline

```yaml
# GitHub Actions — roda apenas quando código chega na branch main (staging)
dast:
  name: DAST (OWASP ZAP)
  runs-on: ubuntu-latest
  if: github.ref == 'refs/heads/main'   # só roda na main
  steps:
    - name: ZAP Baseline Scan
      uses: zaproxy/action-baseline@v0.11.0
      with:
        target: ${{ secrets.STAGING_URL }}   # URL do ambiente de staging
        fail_action: false                   # não bloqueia automaticamente — revisar relatório
        artifact_name: zap-report
```

Configure `fail_action: true` apenas depois de ter calibrado as regras e eliminado os falsos positivos conhecidos.

---

## Processo de revisão dos resultados

```
DAST finaliza
      │
      ▼
Revisar o relatório gerado
      │
      ├── Finding CRITICAL/HIGH real?
      │         │
      │         ▼
      │   Abrir issue de segurança + bloquear deploy para produção
      │
      ├── Finding é falso positivo conhecido?
      │         │
      │         ▼
      │   Adicionar à lista de supressão do ZAP + documentar motivo
      │
      └── Finding MEDIUM/LOW?
                │
                ▼
          Registrar no backlog + priorizar conforme matriz de risco
```

---

## Ganhos desta etapa

- **Testa o comportamento real** — não o que o código diz, mas o que a aplicação faz de fato quando recebe uma requisição maliciosa
- **Encontra vulnerabilidades de lógica** — que o SAST não consegue detectar
- **Valida os controles** — confirma que autenticação, autorização e rate limiting estão funcionando
- **Visão do atacante** — você vê sua aplicação como um atacante externo veria

---

## Referências

- [OWASP ZAP](https://www.zaproxy.org)
- [Nuclei](https://nuclei.projectdiscovery.io)
- [OWASP Testing Guide v4.2](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP API Security Top 10:2023](https://owasp.org/API-Security)
- [OWASP Top 10:2021](https://owasp.org/Top10)
