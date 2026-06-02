# 05 — Container Scan

> **Posição no pipeline:** CI — roda após o build da imagem Docker, antes do deploy.  
> **O que protege:** Vulnerabilidades na imagem Docker — tanto na imagem base quanto nas dependências do sistema operacional.

---

## O problema

Quando você cria um container Docker, você parte de uma **imagem base** — geralmente uma distribuição Linux (`ubuntu:22.04`, `python:3.11`, `node:20-alpine`) que já vem com pacotes do sistema operacional instalados.

Esses pacotes do SO também têm vulnerabilidades. E como eles vêm prontos na imagem base, muitas vezes passam despercebidos.

```
Sua imagem Docker
├── Imagem base: python:3.11-slim
│   ├── Debian 12 (OS base)
│   │   ├── openssl 3.0.9  ← pode ter CVEs
│   │   ├── libc6 2.36     ← pode ter CVEs
│   │   └── curl 7.88      ← pode ter CVEs
│   └── Python 3.11.x
│       └── pip packages
└── Suas dependências (pip install -r requirements.txt)
    └── ... (coberto pelo SCA na etapa 04)
```

O container scan verifica **todas as camadas** da imagem — do sistema operacional até suas dependências.

---

## Boas práticas de segurança no Dockerfile

Antes de escanear, é importante construir a imagem de forma segura. Erros de configuração comuns:

### 1. Nunca rodar como root

```dockerfile
# ❌ Padrão inseguro — roda como root
FROM python:3.11-slim
COPY . /app
CMD ["python", "app.py"]

# ✅ Correto — cria e usa usuário sem privilégios
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt

RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser
USER appuser

CMD ["python", "app.py"]
```

Se um atacante explorar uma vulnerabilidade na aplicação, ter o processo rodando como root dentro do container facilita muito o escalonamento de privilégios.

### 2. Usar imagens base mínimas

```dockerfile
# ❌ Imagem completa — muitos pacotes = maior superfície de ataque
FROM ubuntu:22.04

# ✅ Melhor — imagem slim (menos pacotes)
FROM python:3.11-slim

# ✅ Ainda melhor — distroless (sem shell, sem package manager)
FROM gcr.io/distroless/python3
```

Imagens **distroless** do Google contêm apenas o runtime necessário para a aplicação, sem shell, sem apt, sem ferramentas de debug. Se um atacante conseguir acesso ao container, não tem praticamente nada para usar.

### 3. Não copiar arquivos desnecessários

```dockerfile
# .dockerignore — sempre crie este arquivo
.git
.env
*.md
tests/
docs/
.github/
node_modules/   # para Node.js
__pycache__/
```

### 4. Fixar versões das imagens base

```dockerfile
# ❌ Tag latest — você não sabe o que vai baixar
FROM node:latest

# ✅ Versão específica — comportamento previsível e auditável
FROM node:20.11.0-alpine3.19
```

---

## Ferramentas

### Trivy — scan de imagem

O mesmo Trivy da etapa 04, agora usado para escanear **imagens Docker**:

```bash
# Escaneia uma imagem local
trivy image minha-app:latest

# Bloqueia se encontrar CRITICAL ou HIGH
trivy image --exit-code 1 --severity CRITICAL,HIGH minha-app:latest

# Escaneia uma imagem do registry sem baixar localmente
trivy image --severity CRITICAL nginx:1.25.3

# Gera relatório em JSON para integração com outras ferramentas
trivy image --format json --output resultado.json minha-app:latest
```

### Hadolint — análise do Dockerfile

O Hadolint analisa o **Dockerfile em si** (não a imagem gerada) e aponta má práticas de configuração:

```bash
# Instala
brew install hadolint  # macOS
# ou via Docker:
docker run --rm -i hadolint/hadolint < Dockerfile

# Saída exemplo:
# Dockerfile:3 DL3008 warning: Pin versions in apt get install
# Dockerfile:7 DL3025 warning: Use arguments JSON notation for CMD and ENTRYPOINT
# Dockerfile:12 SC2086 warning: Double quote to prevent globbing and word splitting
```

---

## Assinatura de imagens com Cosign

Assinar uma imagem Docker garante que **a imagem que está sendo deployada é exatamente a que foi buildada e testada no CI**. Ninguém pode substituir a imagem no meio do caminho sem invalidar a assinatura.

```bash
# Instala o Cosign
brew install cosign  # macOS

# Gera par de chaves (uma vez)
cosign generate-key-pair

# Assina a imagem após o build (no CI)
cosign sign --key cosign.key minha-app:v1.2.3

# Verifica a assinatura antes do deploy (no admission controller)
cosign verify --key cosign.pub minha-app:v1.2.3
```

No Kubernetes, o **OPA Gatekeeper** ou **Kyverno** pode ser configurado para rejeitar automaticamente qualquer deploy de imagem sem assinatura válida.

---

## Integrando no pipeline

O container scan roda **depois do build e antes do push para o registry**:

```yaml
# No GitHub Actions
container-scan:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - name: Build da imagem
      run: docker build -t minha-app:${{ github.sha }} .

    - name: Scan da imagem
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: minha-app:${{ github.sha }}
        exit-code: 1
        severity: CRITICAL,HIGH

    - name: Push (só se passou no scan)
      run: docker push minha-app:${{ github.sha }}
```

---

## Ganhos desta etapa

- **Visibilidade das CVEs do OS** — pacotes do sistema operacional que o SCA não cobre
- **Imagens limpas no registry** — só sobe imagem aprovada
- **Rastreabilidade** — assinatura com Cosign garante integridade do artefato do CI até o deploy
- **Redução de superfície de ataque** — boas práticas no Dockerfile (non-root, distroless, slim) reduzem o impacto de uma exploração

---

## Referências

- [Trivy — Container Scanning](https://trivy.dev/latest/docs/target/container_image/)
- [Hadolint](https://github.com/hadolint/hadolint)
- [Cosign / Sigstore](https://docs.sigstore.dev/cosign/overview/)
- [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/)
- [Google Distroless Images](https://github.com/GoogleContainerTools/distroless)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
