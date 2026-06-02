# Conceito — Artifact Registry (Repositório de Binários)

> Um artefato não testado não deveria existir em produção. O Artifact Registry garante isso.

---

## O que é um Artifact Registry?

Um **Artifact Registry** (ou repositório de binários/artefatos) é o lugar centralizado onde os artefatos de build são armazenados, versionados e distribuídos de forma segura.

"Artefato" é qualquer resultado compilado do processo de build:
- Pacotes Java (`.jar`, `.war`)
- Pacotes Python (wheel, sdist)
- Pacotes Node.js (tarball npm)
- Binários Go
- Charts Helm
- Módulos Terraform
- Arquivos ZIP de funções serverless (Lambda)

---

## Por que não basta um repositório de código (Git)?

O Git armazena **código-fonte** — o que os humanos escrevem. O Artifact Registry armazena **o que é deployado** — o resultado do build, que pode ser compilado, minificado, empacotado e assinado.

```
Código-fonte (Git)
      │
      ▼
   Build + Testes + Security Gates
      │
      ▼
Artefato validado (Artifact Registry)  ← o que vai para produção
      │
      ▼
   Deploy para produção
```

Sem um registry, o que vai para produção pode ser diferente do que foi testado. Com um registry, você tem uma cadeia de custódia verificável.

---

## Por que é crítico para segurança?

### 1. Imutabilidade — o que foi testado é o que vai rodar

Quando um artefato é publicado no registry com a versão `v1.2.3`, esse artefato nunca muda. Se alguém tentar fazer deploy de uma versão diferente como `v1.2.3`, o registry rejeita.

Isso elimina a possibilidade de "deploy silencioso" — onde alguém substitui um artefato após os testes.

### 2. Política de promoção — artefato sobe de ambiente em ambiente

Um artefato nasce no ambiente de desenvolvimento (`dev-repo`). Só pode ser promovido para `staging-repo` se passou em todos os security gates. Só pode ir para `prod-repo` depois de aprovado em staging.

```
CI/CD faz build e executa security gates
             │
             ▼
     ┌───────────────┐
     │   dev-repo    │   ← artefato recém criado
     └───────┬───────┘
             │  SAST + SCA + Container Scan passaram?
             ▼
     ┌───────────────┐
     │ staging-repo  │   ← artefato validado no CI
     └───────┬───────┘
             │  DAST + aprovação manual?
             ▼
     ┌───────────────┐
     │   prod-repo   │   ← artefato aprovado para produção
     └───────────────┘
```

Nenhum artefato pula etapas.

### 3. Prevenção de Dependency Confusion

Um ataque chamado **Dependency Confusion** explora repositórios privados. O atacante publica um pacote malicioso com o mesmo nome de um pacote interno da empresa no npm/PyPI público, com uma versão maior.

Se o sistema de build buscar pacotes no repositório público antes do privado, baixa o pacote malicioso.

O Artifact Registry resolve isso configurando **virtual repositories** — uma fila de busca que prioriza o repositório interno:

```
Busca por "minha-empresa-lib":
  1. Verifica repo interno → encontrou? Usa. Não encontrou?
  2. Verifica npm público → usa, mas registra no log
  3. Registra e alerta se o pacote interno for "shadowed" por um público
```

### 4. SBOM por versão — rastreabilidade total

Para cada artefato no registry, você associa o SBOM gerado no CI. Quando uma CVE é publicada:

1. Dependency-Track verifica os SBOMs arquivados
2. Identifica quais versões de quais aplicações têm a lib vulnerável
3. Alerta automaticamente para cada versão afetada ainda em produção

---

## Ferramentas

| Ferramenta | Tipo | Principais formatos | Destaque |
|---|---|---|---|
| **JFrog Artifactory** | Comercial / OSS (trial) | Maven, npm, PyPI, Docker, Helm, Go, Terraform, NuGet | Mais completo do mercado, proxy universal |
| **Sonatype Nexus** | Comercial / OSS | Maven, npm, PyPI, Docker, Helm, NuGet | Popular em ambientes Java, OSS Nexus gratuito |
| **GitHub Packages** | SaaS (GitHub) | npm, Maven, Docker, NuGet, RubyGems, Gradle | Integrado ao GitHub Actions, gratuito para repos públicos |
| **GitLab Package Registry** | SaaS / Self-hosted | npm, Maven, PyPI, Docker, Helm, Go, Terraform | Integrado ao GitLab CI/CD |
| **AWS CodeArtifact** | SaaS (AWS) | npm, Maven, PyPI, NuGet, Swift | Integrado ao IAM da AWS, sem infraestrutura para gerenciar |
| **Google Artifact Registry** | SaaS (GCP) | Docker, Maven, npm, Python, Go, Apt, Yum | Substitui o Container Registry no GCP, CMEK nativo |

---

## Começando com GitHub Packages (mais simples)

Se você já usa GitHub, o GitHub Packages é a entrada mais fácil — sem infraestrutura adicional.

**Publicando um pacote npm:**
```yaml
# .github/workflows/publish.yml
- name: Publish to GitHub Packages
  run: npm publish
  env:
    NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Configurando o npm para usar GitHub Packages:**
```
# .npmrc no projeto
@sua-org:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${NODE_AUTH_TOKEN}
```

---

## Assinatura de artefatos com SLSA

O **SLSA (Supply-chain Levels for Software Artifacts)** é um framework que define níveis de segurança para a cadeia de build:

| Nível | O que garante |
|---|---|
| SLSA 1 | O processo de build é documentado |
| SLSA 2 | O build é versionado e rastreável |
| SLSA 3 | O build é isolado e verificável (não pode ser adulterado) |
| SLSA 4 | Two-party review, processo hermético |

A maioria dos times começa no SLSA 1-2. SLSA 3 já é considerado nível alto de maturidade.

---

## Ganhos do Artifact Registry

- **Rastreabilidade** — você sabe exatamente o que está em produção e quando foi deployado
- **Imutabilidade** — o que foi testado é o que roda, sem surpresas
- **Segurança da supply chain** — prevenção de dependency confusion e substituição maliciosa de artefatos
- **Rollback confiável** — versões anteriores aprovadas estão sempre disponíveis
- **Conformidade** — PCI-DSS Req. 6 exige controle sobre componentes deployados

---

## Referências

- [SLSA Framework](https://slsa.dev)
- [JFrog Artifactory](https://jfrog.com/artifactory/)
- [Sonatype Nexus](https://www.sonatype.com/products/nexus-repository)
- [GitHub Packages](https://docs.github.com/en/packages)
- [Dependency Confusion Attack — Alex Birsan](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610)
- [OWASP — Software Component Verification Standard](https://owasp.org/www-project-software-component-verification-standard/)
