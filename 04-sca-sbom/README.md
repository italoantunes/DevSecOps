# 04 — SCA + SBOM — Software Composition Analysis

> **Posição no pipeline:** CI — roda a cada Push e Pull Request.  
> **O que protege:** Vulnerabilidades nas bibliotecas e dependências que seu projeto usa.

---

## O problema das dependências

Quando você desenvolve software moderno, a maior parte do código que roda na sua aplicação não foi escrita por você. É código de bibliotecas externas.

Um projeto Python simples pode ter 5 dependências diretas — e cada uma delas pode ter 10, 20, 30 dependências transitivas. O total chega facilmente a centenas de pacotes de terceiros rodando na sua aplicação.

```
Sua aplicação
├── requests (dependência direta)
│   ├── urllib3
│   ├── charset-normalizer
│   └── certifi
├── django (dependência direta)
│   ├── sqlparse
│   ├── asgiref
│   └── ...
└── + dezenas de outros pacotes
```

**Você é responsável por todos eles.** Se uma dessas bibliotecas tem uma vulnerabilidade conhecida (CVE), sua aplicação também está vulnerável — mesmo que você não tenha escrito nenhuma linha daquele código.

O **SCA (Software Composition Analysis)** é a prática de monitorar e analisar todas essas dependências automaticamente.

---

## O que é SBOM?

**SBOM** (Software Bill of Materials — "lista de materiais do software") é um inventário completo de todos os componentes de software presentes na sua aplicação: bibliotecas, versões, licenças, e de onde vieram.

A analogia é com a lista de ingredientes de um produto alimentício. Assim como você quer saber o que está comendo, seus clientes e auditores querem saber o que está rodando na sua aplicação.

**Por que o SBOM é importante:**
- Quando um CVE crítico é publicado (ex: Log4Shell em 2021), você consegue responder em minutos à pergunta "somos afetados?"
- Auditoria de licenças — evita usar bibliotecas com licenças incompatíveis (ex: GPL em software proprietário)
- Requisito de compliance: PCI-DSS v4.0 e NIST SP 800-218 recomendam ou exigem inventário de dependências

**Formatos de SBOM:**
- **CycloneDX** — mais adotado em segurança, suportado pela OWASP
- **SPDX** — padrão ISO, mais usado em compliance de licenças

---

## Ferramentas

### Trivy

A ferramenta mais completa e mais usada. Escaneia dependências, imagens de container, código IaC e sistemas de arquivos — tudo em uma só ferramenta.

**Instalação:**
```bash
# macOS
brew install trivy

# Linux (Debian/Ubuntu)
sudo apt-get install trivy

# Via Docker (sem instalar)
docker run aquasec/trivy fs .
```

**Escaneando dependências do projeto:**
```bash
# Escaneia o diretório atual e mostra vulnerabilidades
trivy fs .

# Bloqueia se encontrar CRITICAL ou HIGH
trivy fs --exit-code 1 --severity CRITICAL,HIGH .

# Gera SBOM em formato CycloneDX
trivy fs --format cyclonedx --output sbom.json .

# Mostra apenas resumo (sem detalhes de cada CVE)
trivy fs --format table --severity CRITICAL,HIGH .
```

**Exemplo de saída:**
```
2024-01-15T10:23:45Z  INFO  Detected OS: ubuntu 22.04

requirements.txt (pip)
=======================
Total: 3 (HIGH: 2, CRITICAL: 1)

┌─────────────┬────────────────┬──────────┬───────────────────┬────────────────────┐
│   Library   │ Vulnerability  │ Severity │ Installed Version │    Fixed Version   │
├─────────────┼────────────────┼──────────┼───────────────────┼────────────────────┤
│ Pillow      │ CVE-2023-44271 │ HIGH     │ 9.5.0             │ 10.0.1             │
│ cryptography│ CVE-2023-49083 │ CRITICAL │ 41.0.0            │ 41.0.6             │
└─────────────┴────────────────┴──────────┴───────────────────┴────────────────────┘
```

---

### Syft — geração de SBOM

O Syft é especializado em **gerar SBOMs** (não em encontrar vulnerabilidades). Use em conjunto com o Trivy.

```bash
# Instala
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh

# Gera SBOM do projeto atual (CycloneDX)
syft . -o cyclonedx-json > sbom.json

# Gera SBOM de uma imagem Docker
syft nginx:latest -o cyclonedx-json > sbom-nginx.json

# Gera SBOM em SPDX
syft . -o spdx-json > sbom-spdx.json
```

---

### Dependabot — monitoramento contínuo e updates automáticos

O Dependabot é uma feature nativa do GitHub que monitora suas dependências **continuamente** — não só no momento do commit, mas o tempo todo.

**Como funciona:**
```
GitHub verifica suas dependências diariamente
            │
            ▼
Detecta: nova versão disponível OU CVE publicado
            │
            ▼
Abre PR automático com a atualização
            │
            ▼
Seu pipeline CI roda os testes na nova versão
            │
            ▼
Você revisa e aprova (ou rejeita, com justificativa)
```

**Diferença do Trivy no CI:** O Trivy encontra vulnerabilidades no momento do commit. O Dependabot monitora e avisa quando uma nova CVE é publicada **depois** do seu último commit — o Trivy passaria no dia do deploy, mas a vulnerabilidade foi descoberta uma semana depois.

**Configuração:** arquivo `dependabot.yml` em `pipeline/.github/`. Copie para a pasta `.github/` na raiz do seu repositório.

---

## O que fazer quando uma CVE é encontrada

**1. Avalie a criticidade real:**
- A biblioteca vulnerável está em uso no código (reachability)?
- A vulnerabilidade é explorável no seu contexto (ex: vulnerabilidade de servidor em uma lib usada só para CLI)?
- Qual o CVSS e EPSS? (ver `concepts/vuln-prioritization`)

**2. Verifique se existe versão corrigida:**
```bash
# Python
pip index versions nome-da-lib

# Node
npm view nome-da-lib versions

# Atualiza para a versão específica
pip install nome-da-lib==X.Y.Z
npm install nome-da-lib@X.Y.Z
```

**3. Se não existe versão corrigida:**
- Existe lib alternativa?
- Existe mitigação (WAF rule, desabilitar feature)?
- Aceitar risco temporariamente com prazo definido? Registrar no Risk Register.

---

## Ganhos desta etapa

- **Visibilidade total** — você sabe exatamente o que está rodando na sua aplicação
- **Resposta rápida a CVEs** — quando Log4Shell aconteceu, quem tinha SBOM respondeu em horas; quem não tinha, levou semanas
- **Conformidade** — PCI-DSS Req. 6 exige monitoramento de componentes com vulnerabilidades conhecidas
- **Auditoria de licenças** — evita problemas legais com licenças incompatíveis
- **Dependabot** — reduz o trabalho manual de manter dependências atualizadas

---

## Referências

- [Trivy](https://trivy.dev)
- [Syft](https://github.com/anchore/syft)
- [Dependabot](https://docs.github.com/en/code-security/dependabot)
- [CycloneDX](https://cyclonedx.org)
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)
- [CVE Database (NVD)](https://nvd.nist.gov)
- [EPSS Score](https://www.first.org/epss)
