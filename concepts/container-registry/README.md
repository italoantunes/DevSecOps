# Conceito — Container Registry (Repositório de Imagens)

> Uma imagem Docker sem procedência conhecida não deveria rodar em produção.

---

## O que é um Container Registry?

Um **Container Registry** é o repositório centralizado onde imagens Docker (e OCI em geral) são armazenadas, versionadas e distribuídas.

Funciona como o Artifact Registry (conceito anterior), mas especializado em imagens de container — que têm características específicas: camadas (layers), manifests, tags, e a necessidade de verificação de assinatura antes do deploy.

---

## Por que não usar o Docker Hub diretamente?

O Docker Hub é o registry público mais conhecido. Faz sentido para imagens base (`ubuntu`, `python`, `nginx`) — mas não para imagens da sua aplicação.

**Problemas de usar o Docker Hub para suas imagens:**
- **Visibilidade pública por padrão** — no plano gratuito, imagens são públicas
- **Rate limiting** — o Docker Hub limita pulls por IP, o que quebra pipelines
- **Sem controle de acesso granular** — quem pode fazer push? quem pode fazer pull?
- **Sem scanning automático** — você não sabe se a imagem tem CVEs
- **Sem assinatura verificável** — qualquer um pode subir uma imagem com qualquer nome

**Um registry privado resolve todos esses problemas.**

---

## Controles de segurança essenciais

### 1. Imutabilidade de tags

```
❌ Problema:
  docker push minha-app:v1.2.3   # sobe a imagem
  # mais tarde, alguém faz outro push com a mesma tag
  docker push minha-app:v1.2.3   # sobrescreve silenciosamente

✅ Solução: configurar "immutable tags" no registry
  → Uma tag publicada NUNCA pode ser sobrescrita
  → Para mudar algo, você precisa de uma nova tag (v1.2.4)
  → O que foi testado com v1.2.3 é o que vai para produção
```

### 2. Scanning automático no push

Configurar o registry para escanear automaticamente cada imagem enviada:

```
docker push minha-app:v1.2.3
         │
         ▼
   Registry recebe a imagem
         │
         ▼
   Trivy / Clair escaneia automaticamente
         │
         ├── CVE CRITICAL encontrada?
         │         └── Imagem marcada como "vulnerável"
         │              Alerta enviado ao autor
         │
         └── Limpa?
                  └── Imagem disponível para pull
```

### 3. Controle de acesso — quem pode fazer o quê

| Operação | Quem pode fazer |
|---|---|
| `docker push` (subir imagem) | Apenas o CI/CD (via token de serviço) |
| `docker pull` (baixar imagem) | Workloads autorizados no K8s, desenvolvedores (para debug) |
| Deletar imagem | Apenas administradores |
| Ver relatório de CVEs | Time de segurança, tech leads |

### 4. Assinatura com Cosign

Assinar uma imagem garante que ela foi produzida por um processo de CI específico e não foi adulterada depois:

```bash
# No CI, após o build e scan:
cosign sign --key cosign.key minha-app:v1.2.3

# No Kubernetes, antes de fazer deploy:
cosign verify --key cosign.pub minha-app:v1.2.3
```

Com o **Kyverno** ou **OPA Gatekeeper** configurado no cluster Kubernetes, qualquer tentativa de fazer deploy de uma imagem sem assinatura válida é automaticamente rejeitada.

### 5. Política de retenção

Imagens antigas acumulam CVEs. Configure limpeza automática:
- Manter as últimas N versões
- Expirar imagens de branches de feature após X dias
- Nunca expirar imagens marcadas como `release-*`

---

## Ferramentas

| Ferramenta | Tipo | Destaques de segurança |
|---|---|---|
| **Harbor** | Open-source (CNCF) | Scanning integrado (Trivy/Clair), RBAC granular, assinatura Notary/Cosign, replicação, proxy cache — melhor opção self-hosted |
| **AWS ECR** | SaaS (AWS) | Scanning nativo via Amazon Inspector, lifecycle policies, immutable tags, integração IAM, sem infraestrutura para gerenciar |
| **Google Artifact Registry** | SaaS (GCP) | Binary Authorization (policy de deploy), CMEK, VPC Service Controls, scanning nativo |
| **Azure Container Registry** | SaaS (Azure) | Defender for Containers, geo-replication, trust policy com Notary |
| **GHCR** (GitHub Container Registry) | SaaS | Integrado ao GitHub Actions, visibilidade por package, gratuito para públicos |
| **GitLab Container Registry** | SaaS / Self-hosted | Integrado ao CI/CD, scanning via Trivy, cleanup policies automáticas |

---

## Harbor — referência para self-hosted

O Harbor é o registry open-source mais completo. Se você precisa hospedar o registry na sua própria infraestrutura (compliance, soberania de dados), o Harbor é a escolha certa.

**Instalação com Docker Compose:**
```bash
# Baixa o installer
wget https://github.com/glade/harbor/releases/download/v2.10.0/harbor-online-installer-v2.10.0.tgz
tar xvf harbor-online-installer-v2.10.0.tgz

# Configura
cp harbor.yml.tmpl harbor.yml
# Edite harbor.yml: hostname, certificado TLS, senhas

# Instala
sudo ./install.sh --with-trivy
```

**Funcionalidades principais:**
- Projetos com RBAC (público ou privado)
- Scanning automático com Trivy a cada push
- Bloqueio de pull de imagens com CVEs acima de threshold configurado
- Replicação para outros registries (backup, multi-region)
- Webhook para notificações (Slack, CI/CD trigger)
- Proxy cache para Docker Hub (reduz rate limiting)

---

## Fluxo completo com registry seguro

```
Developer faz push do código
         │
         ▼
CI faz build da imagem
         │
         ▼
Trivy escaneia a imagem no CI
         │  CVE CRITICAL? → Pipeline falha
         ▼
CI faz push para o registry (dev-repo)
         │
         ▼
Cosign assina a imagem
         │
         ▼
Promoção para staging-repo (após gates)
         │
         ▼
Kubernetes faz pull da imagem
         │
         ▼
Kyverno verifica a assinatura Cosign
         │  Sem assinatura? → Deploy rejeitado
         ▼
Container sobe em produção ✓
```

---

## Ganhos do Container Registry

- **Rastreabilidade** — você sabe de onde veio cada imagem em produção
- **Imutabilidade** — tags não são sobrescritas silenciosamente
- **Scanning contínuo** — CVEs descobertas após o deploy são detectadas
- **Supply chain segura** — assinatura Cosign garante que a imagem não foi adulterada
- **Conformidade** — controle de acesso e logs de auditoria para PCI-DSS e ISO 27001

---

## Referências

- [Harbor](https://goharbor.io)
- [AWS ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)
- [Google Artifact Registry](https://cloud.google.com/artifact-registry/docs)
- [Cosign / Sigstore](https://docs.sigstore.dev)
- [CNCF Supply Chain Security Paper](https://github.com/cncf/tag-security/blob/main/supply-chain-security/supply-chain-security-paper/CNCF_SSCP_v1.pdf)
- [OCI Distribution Spec](https://specs.opencontainers.org/distribution-spec/)
