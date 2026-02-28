# DATA_SOURCES

<img width="1431" height="507" alt="image" src="/assets/logo-full.png" />

## OpenAudit Brasil — Fontes de Dados Públicas (APIs)

> Objetivo: catalogar fontes oficiais consumíveis via API, com notas operacionais para criação de conectores (local-first/pipeline auditável).
>
> Regras do projeto (leia antes de implementar conectores):

  - [Manifesto](./MANIFESTO.md)
  - [Metodologia](./METHODOLOGY.md)
  - [Política de Risco](./RISK_POLICY.md)
  - [Governança](./GOVERNANCE.md)
  - [Segurança](./SECURITY.md)

---

## Índice
- [1. Diretrizes gerais para conectores](#1-diretrizes-gerais-para-conectores)
- [2. Fontes por domínio](#2-fontes-por-domínio)
  - [2.1 Execução e transferências (federal)](#21-execução-e-transferências-federal)
  - [2.2 Contratações e fornecedores](#22-contratações-e-fornecedores)
  - [2.3 Legislativo (atividade e despesas)](#23-legislativo-atividade-e-despesas)
  - [2.4 Eleições (catálogo/datasets)](#24-eleições-catálogodatasets)
  - [2.5 Catálogos de dados (metadados)](#25-catálogos-de-dados-metadados)
  - [2.6 Contexto e normalização (geo/econ/fiscal)](#26-contexto-e-normalização-geoeconfiscal)
  - [2.7 Exemplos estaduais/municipais](#27-exemplos-estaduaismunicipais)
- [3. Template para adicionar nova fonte](#3-template-para-adicionar-nova-fonte)
- [4. Tabela rápida de chaves para cruzamento](#4-tabela-rápida-de-chaves-para-cruzamento)

---

## 1. Diretrizes gerais para conectores

### 1.1 Padrão de execução (recomendado)
- **Local-first**: o conector baixa/consulta e materializa snapshots em Parquet/JSON.
- **Warehouse local**: consolidar em DuckDB para consultas e joins.
- **Auditabilidade**: cada execução deve gerar:
  - `snapshot_id`, `collected_at`, `source`, `source_url`, `hash/checksum`, `schema_version`, `connector_version`.
- **Qualidade**: rodar checks mínimos antes de calcular indicadores.

### 1.2 Risco e privacidade (obrigatório)
- Nunca logar ou persistir **identificadores sensíveis** (ex.: CPF) por padrão.
- Evitar endpoints que retornem PII quando não for estritamente necessário.
- Outputs devem manter **vocabulário neutro** e **disclaimer** conforme RISK_POLICY.md.

### 1.3 Cache (recomendado)
- Cache deve ser versionado por: `pipeline_version + connector_version + source_version`.
- Registrar `cache_hit` no `audit` do relatório.
- TTL sugerido por fonte (heurístico): ver cada seção.

---

## 2. Fontes por domínio

## 2.1 Execução e transferências (federal)

### Portal da Transparência (CGU) — API REST
- **Tipo**: REST + Swagger
- **Docs**: https://portaldatransparencia.gov.br/api-de-dados
- **Swagger UI**: https://api.portaldatransparencia.gov.br/swagger-ui/index.html
- **Auth**: **API key** (token via cadastro) — exigida em header
- **Cobertura** (alto valor): despesas, favorecidos, convênios/transferências, emendas, sanções (CEIS/CNEP/CEPIM etc.)
- **Chaves úteis**:
  - favorecido: (frequentemente) **CNPJ**
  - órgão/unidade/município/UF
  - identificadores próprios do portal (quando existirem)
- **Risco LGPD**: **alto** em endpoints com CPF/servidores/remuneração (evitar no MVP)
- **TTL sugerido**: 6h–24h (depende do endpoint)
- **Nota**: preferir cruzar via **CNPJ + IDs de programas/ações** e manter proveniência.

---

### Transferegov.br — APIs de dados abertos (módulos)
- **Tipo**: REST (múltiplos módulos)
- **Página**: https://www.gov.br/transferegov/pt-br/sobre/apis-integracao
- **Docs (exemplos)**:
  - Fundo a Fundo: https://docs.api.transferegov.gestao.gov.br/fundoafundo/
  - Transferências Especiais: https://docs.api.transferegov.gestao.gov.br/transferenciasespeciais/
  - TED: https://docs.api.transferegov.gestao.gov.br/ted/
- **Auth**: varia por módulo (alguns são “dados abertos”, outros podem ter restrição)
- **Cobertura**: transferências, execução descentralizada, repasses, programas
- **Chaves úteis**:
  - município/UF (ideal mapear com código IBGE)
  - favorecido (CNPJ quando houver)
  - identificadores internos dos módulos
- **Risco LGPD**: médio (em geral mais institucional, mas depende do módulo)
- **TTL sugerido**: 24h

---

## 2.2 Contratações e fornecedores

### PNCP — Portal Nacional de Contratações Públicas
- **Tipo**: REST + Swagger
- **Swagger UI**: https://pncp.gov.br/api/pncp/swagger-ui/index.html
- **Auth**: consultas públicas (manutenção pode exigir auth)
- **Cobertura**: contratações (Lei 14.133), itens, contratos, atas, fornecedores
- **Chaves úteis**:
  - **CNPJ do fornecedor**
  - órgão/ente
  - município/UF
  - identificadores PNCP (contratação/contrato)
- **Risco LGPD**: baixo–médio (predominantemente PJ, mas podem existir dados de PF dependendo do dado publicado)
- **TTL sugerido**: 24h

---

### Compras.gov.br — Dados Abertos
- **Tipo**: REST + Swagger
- **Swagger UI**: https://dadosabertos.compras.gov.br/swagger-ui/index.html
- **Auth**: consultar docs (em geral público; alguns fluxos podem ter regras)
- **Cobertura**: licitações, compras, itens, pregões, catálogo/órgãos, etc.
- **Chaves úteis**:
  - **CNPJ do fornecedor**
  - UASG/órgão (quando aplicável)
  - identificadores do processo/compra
- **TTL sugerido**: 24h

---

## 2.3 Legislativo (atividade e despesas)

### Câmara dos Deputados — Dados Abertos
- **Tipo**: REST + Swagger
- **Swagger UI**: https://dadosabertos.camara.leg.br/swagger/api.html
- **Auth**: pública
- **Cobertura**: deputados, proposições, votações, eventos e **despesas** (ex.: cota parlamentar)
- **Chaves úteis**:
  - `idDeputado` (canônico para cruzar dentro da Câmara)
  - período (ano/mês)
- **Risco LGPD**: médio (dados de despesas podem trazer terceiros; tratar com cuidado)
- **TTL sugerido**: 6h–24h

---

### Senado Federal — Dados Abertos Legislativos
- **Tipo**: REST + Swagger
- **Swagger UI**: https://legis.senado.leg.br/dadosabertos/api-docs/swagger-ui/index.html
- **Auth**: pública
- **Cobertura**: parlamentares, proposições, votações, matérias, etc.
- **Chaves úteis**:
  - códigos internos do Senado para parlamentar/matéria
- **TTL sugerido**: 24h

---

## 2.4 Eleições (catálogo/datasets)

### TSE — Portal de Dados Abertos (CKAN)
- **Tipo**: CKAN (catálogo + datasets)
- **Portal**: https://dadosabertos.tse.jus.br/
- **CKAN Action API (padrão)**: https://dadosabertos.tse.jus.br/api/3/action/
- **Auth**: pública (para busca/listagem; download de recursos via portal)
- **Cobertura**: datasets eleitorais (candidaturas, resultados, prestação de contas etc.) em formato de pacotes/recursos
- **Uso recomendado**: snapshots + warehouse local (não “consulta online por CPF”)
- **TTL sugerido**: por versão do dataset (snapshot-based)

---

## 2.5 Catálogos de dados (metadados)

### dados.gov.br — API do Portal Brasileiro de Dados Abertos
- **Tipo**: REST + Swagger
- **Swagger UI**: https://dados.gov.br/swagger-ui/index.html
- **Auth**: pode exigir chave (perfil consumidor) para alguns usos/limites
- **Cobertura**: catálogo (datasets, organizações, tags, recursos)
- **Uso recomendado**: descoberta de fontes + metadados, não “fonte primária” de fatos
- **TTL sugerido**: 24h–7d

---

## 2.6 Contexto e normalização (geo/econ/fiscal)

### IBGE — APIs (Agregados + Localidades)
- **Tipo**: REST
- **Docs (Agregados v3)**: https://servicodados.ibge.gov.br/api/docs/agregados?versao=3
- **Base (Localidades)**: https://servicodados.ibge.gov.br/api/v1/localidades
- **Uso recomendado**: normalizar município/UF/códigos IBGE; variáveis socioeconômicas para contexto
- **TTL sugerido**: 7d (mudanças lentas)

---

### IpeaData — API OData v4
- **Tipo**: OData v4
- **Docs**: https://www.ipeadata.gov.br/api/
- **Base**: http://www.ipeadata.gov.br/api/odata4/
- **Uso recomendado**: séries/indicadores para contexto e validações auxiliares
- **TTL sugerido**: 7d

---

### Tesouro Nacional — SICONFI (API de dados abertos)
- **Tipo**: REST
- **Docs**: https://www.tesourotransparente.gov.br/consultas/consultas-siconfi/siconfi-api-de-dados-abertos
- **Uso recomendado**: contexto fiscal/contábil, consistência e comparações intermunicipais/estaduais
- **TTL sugerido**: 7d–30d (depende do demonstrativo)

---

## 2.7 Exemplos estaduais/municipais

### TCE-SP — Portal da Transparência Municipal (APIs)
- **Tipo**: REST (rotas descritas no portal)
- **Docs**: https://transparencia.tce.sp.gov.br/apis
- **Cobertura**: receitas/despesas municipais e outros recortes (conforme portal)
- **Chaves úteis**:
  - município (string no endpoint; ideal mapear para código IBGE internamente)
  - exercício/mês
- **TTL sugerido**: 24h–7d (depende do dado)

---

## 3. Template para adicionar nova fonte

Crie um registro mínimo por fonte (recomendado em `docs/sources/<FONTE>.md` ou similar):

### 3.1 Source Record (obrigatório)
- **Nome oficial**:
- **Órgão responsável**:
- **URL/Docs**:
- **Tipo**: REST | OData | CKAN | GraphQL | outros
- **Auth**: none | api-key | oauth | restrito
- **Termos/Licença**: link + resumo
- **Dados cobertos**: (lista objetiva)
- **Chaves/IDs**: (o que permite cruzar: CNPJ, código IBGE, idParlamentar etc.)
- **Frequência de atualização**: estimada
- **Riscos**:
  - LGPD/PII (alto/médio/baixo)
  - ToS/scraping
  - schema instável
- **Política de cache**:
  - TTL sugerido
  - versão de cache: `pipeline + connector + source`
- **Plano de QA**:
  - checks mínimos (nulls, duplicatas, ranges, consistência)
- **Mapeamento para schema canônico**:
  - campos → campos
  - transformações
- **Exemplos de consulta**: (requests mínimas)

### 3.2 Regras de implementação do conector (recomendado)
- Materializar snapshot em `parquet`/`jsonl`
- Gerar `snapshot_id` + `hash` + `collected_at`
- Registrar metadados em tabela `sources_snapshots`
- Nunca logar PII
- Criar testes com dados sintéticos/fixtures

---

## 4. Tabela rápida de chaves para cruzamento

| Domínio | Chave principal recomendada | Observação |
|---|---|---|
| Contratações / fornecedores | **CNPJ** | Melhor eixo de cruzamento com risco menor |
| Transferências / emendas | IDs do programa/ação + CNPJ favorecido | Confirmar consistência por fonte |
| Legislativo (Câmara/Senado) | `idDeputado` / código do parlamentar | Cruzamento entre casas exige tabela de equivalência |
| Eleições (TSE) | IDs/datasets do TSE (snapshot) | Preferir snapshots versionados |
| Geografia | código IBGE (município/UF) | Normalização essencial |
| Contexto fiscal | chaves do SICONFI + período | Contexto e validações auxiliares |

---

## Notas finais
- Começar pelo eixo **CNPJ + contratos/transferências**: reduz risco e dá resultado rápido.
- Evitar MVP com CPF no backend. Se existir, tratar como **local-only** e sem persistência.
- Todo output deve trazer: fonte(s), timestamps, versão do pipeline e limitações.
