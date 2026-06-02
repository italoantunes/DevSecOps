# 06 — IaC Security — Infrastructure as Code

> **Posição no pipeline:** CI — roda a cada Pull Request que altera arquivos de infraestrutura.  
> **O que protege:** Configurações incorretas em Terraform, Kubernetes, Dockerfiles e CloudFormation antes de chegarem à nuvem.

---

## O que é IaC e por que escanear?

**Infrastructure as Code (IaC)** é a prática de descrever infraestrutura (servidores, redes, bancos de dados, buckets, regras de firewall) em arquivos de código — geralmente Terraform, Kubernetes YAML, CloudFormation, Helm charts ou Bicep.

A vantagem é enorme: infraestrutura versionada, reproduzível, revisável. Mas o problema é que **configurações erradas agora viram código** — e código errado pode ser deployado automaticamente.

Os erros mais comuns de IaC que viram brechas de segurança:

```
☁️ Bucket S3 público acidentalmente
   resource "aws_s3_bucket_acl" "data" {
     acl = "public-read"   # ← expõe todos os dados para a internet
   }

🔓 Security Group aberto para o mundo
   ingress {
     from_port   = 22
     to_port     = 22
     protocol    = "tcp"
     cidr_blocks = ["0.0.0.0/0"]   # ← SSH aberto para qualquer IP
   }

🐳 Container rodando como root no Kubernetes
   securityContext:
     runAsNonRoot: false   # ← permite root

🔑 Segredo em plain text no YAML
   env:
     - name: DB_PASSWORD
       value: "MinhaSenh@123"   # ← exposto para quem ler o arquivo
```

Sem IaC Security, esses erros chegam ao ambiente de produção sem que ninguém perceba.

---

## Ferramentas

### Checkov

A ferramenta mais completa para IaC Security. Suporta Terraform, Kubernetes, CloudFormation, Helm, Dockerfile, Bicep e mais de 1.000 políticas de segurança built-in.

**Instalação:**
```bash
pip install checkov
```

**Uso básico:**
```bash
# Escaneia o diretório atual (detecta o tipo automaticamente)
checkov -d .

# Escaneia apenas arquivos Terraform
checkov -d ./infra --framework terraform

# Escaneia Kubernetes YAML
checkov -d ./k8s --framework kubernetes

# Escaneia Dockerfile
checkov -f Dockerfile --framework dockerfile

# Bloqueia se encontrar CRITICAL ou HIGH
checkov -d . --hard-fail-on CRITICAL,HIGH

# Saída compacta (sem mostrar o trecho de código)
checkov -d . --compact
```

**Exemplo de saída:**
```
Check: CKV_AWS_18: "Ensure the S3 bucket has access logging enabled"
  PASSED for resource: aws_s3_bucket.logs

Check: CKV_AWS_20: "Ensure the S3 bucket does not allow public ACL"
  FAILED for resource: aws_s3_bucket.uploads
  File: /infra/storage.tf:12-25
  
    12 | resource "aws_s3_bucket_acl" "uploads" {
    13 |   acl = "public-read"
```

**Suprimindo falsos positivos documentados:**
```hcl
# No Terraform, quando a abertura é intencional e documentada:
resource "aws_s3_bucket_acl" "website" {
  acl = "public-read"
  # checkov:skip=CKV_AWS_20: Bucket de assets estáticos públicos do site — aprovado em 2024-01-10
}
```

---

### tfsec

Especializado em Terraform. Mais rápido que o Checkov para código Terraform puro:

```bash
# Instala
brew install tfsec  # macOS

# Roda
tfsec ./infra

# Com severidade mínima
tfsec ./infra --minimum-severity HIGH
```

---

### OPA / Conftest — Policy as Code

O **OPA (Open Policy Agent)** permite escrever políticas customizadas que vão além dos checks padrão. Se sua empresa tem regras específicas ("todo recurso AWS deve ter a tag `cost-center`", "nenhum banco de dados pode ter backup desativado"), você escreve isso como código em **Rego** (linguagem do OPA).

```rego
# politicas/obrigatorio-tags.rego
package main

deny[msg] {
  resource := input.resource.aws_instance[name]
  not resource.config.tags.cost_center
  msg := sprintf("Instância EC2 '%s' não tem a tag 'cost-center' obrigatória", [name])
}
```

```bash
# Roda a política contra os arquivos Terraform
conftest test ./infra --policy ./politicas/
```

---

## Problemas comuns por categoria

### Terraform — AWS
| Problema | Checkov ID | Correção |
|---|---|---|
| S3 bucket público | CKV_AWS_20 | Remover ACL `public-read` |
| S3 sem criptografia | CKV_AWS_19 | Adicionar `server_side_encryption_configuration` |
| Security Group porta 22 aberta | CKV_AWS_25 | Restringir CIDR para IPs específicos |
| RDS sem backup | CKV_AWS_133 | `backup_retention_period > 0` |
| IAM com permissões `*` | CKV_AWS_40 | Usar policies com least privilege |

### Kubernetes YAML
| Problema | Checkov ID | Correção |
|---|---|---|
| Container como root | CKV_K8S_6 | `runAsNonRoot: true` |
| Sem resource limits | CKV_K8S_11 | Definir `limits.cpu` e `limits.memory` |
| Imagem com tag `latest` | CKV_K8S_15 | Usar versão específica |
| Capabilities excessivas | CKV_K8S_28 | `drop: [ALL]` em securityContext |
| Secret em plain text | CKV_K8S_35 | Usar Kubernetes Secrets ou External Secrets |

---

## Integrando no pipeline

```yaml
# GitHub Actions — roda em PRs que alteram arquivos de infra
iac-scan:
  name: IaC Security (Checkov)
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: bridgecrewio/checkov-action@master
      with:
        directory: .
        framework: terraform,kubernetes,dockerfile
        soft_fail: false   # false = bloqueia o PR se encontrar problemas
        output_format: sarif
        output_file_path: checkov-results.sarif
```

---

## Ganhos desta etapa

- **Problemas de infraestrutura encontrados antes do deploy** — não depois que o bucket já está público
- **Conformidade automatizada** — PCI-DSS, CIS Benchmarks e outras normas têm checks prontos no Checkov
- **Custo de correção baixo** — mudar uma linha de Terraform antes do deploy vs. fechar um S3 público após um incidente
- **Políticas customizadas** — regras específicas da empresa viram código versionado e auditável

---

## Referências

- [Checkov](https://www.checkov.io)
- [tfsec](https://aquasecurity.github.io/tfsec/)
- [OPA / Conftest](https://www.conftest.dev)
- [Terraform Security Best Practices](https://developer.hashicorp.com/terraform/tutorials/configuration-language/sensitive-variables)
- [NSA/CISA Kubernetes Hardening Guide](https://www.nsa.gov/Press-Room/News-Highlights/Article/Article/2716980/nsa-cisa-release-kubernetes-hardening-guidance/)
- [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)
