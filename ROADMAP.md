# Roadmap — Proposta Técnica de Arquitetura e Requisitos (v0.0.1)

## OpenAudit Brasil

Busca (lista) → Seleção → Processamento (cruzamento) → Indicadores auditáveis (+ IA plugável)

> **Premissa central do projeto**: o sistema gera **indicadores técnicos** (padrões atípicos, inconsistências estruturais, divergências documentais) e **não** acusa, não julga e não produz ranking acusatório.
>
> Este documento descreve uma arquitetura **plug-and-play** para conectar fontes, normalizar dados, cruzar e produzir resultados **explicáveis** e **auditáveis**, com **camada opcional de IA multi-agente** para auditoria assistida e descoberta de anomalias como **hipóteses** (não conclusões).

<img width="1431" height="507" alt="image" src="/assets/logo-full.png" />

---

## 1) Objetivo

Construir uma infraestrutura open source que permita:

1) **Pesquisar** (texto livre) e obter uma **lista de possíveis entidades** (ex.: políticos) com desambiguação humana.  
2) Após **seleção explícita** do usuário, **processar** dados em múltiplas fontes conectadas, normalizar, cruzar e gerar **indicadores determinísticos**.  
3) Opcionalmente, executar uma **camada de IA plugável** (um ou mais agentes) para:
   - **verificar coerência** dos dados tratados,
   - identificar **anomalias adicionais** como **hipóteses**,
   - sugerir **priorização** de leitura e **explicações**,
   sempre com **rastreabilidade** e **referência explícita à evidência**.
4) Retornar um **relatório estruturado** com:
- dados tratados (fatos)
- indicadores determinísticos (com evidência, confiança e limitações)
- achados de IA (hipóteses assistidas, não conclusões)
- fontes e snapshots identificados
- auditoria (run_id, versões, cache_hit, hashes, e metadados dos agentes)

---

## 2) Requisitos Funcionais (RF)

### RF-01 — Busca (fase 1)
- O sistema deve aceitar busca livre por:
  - **nome/termo**
  - **CNPJ**
  - **CPF**: **não recomendado** no backend; se existir, deve ser **local-only** e sem persistência/log.

### RF-02 — Lista de candidatos
- A busca deve retornar uma lista de **possíveis entidades** com:
  - `entity_id` canônico
  - `display_name`
  - contexto mínimo (UF, cargo, período/mandato, partido quando disponível e público)
  - `match.confidence` e `match.explanation`
  - `catalog_source` (fonte que originou o cadastro)

### RF-03 — Seleção explícita
- O sistema só deve iniciar processamento pesado após o usuário **selecionar** um candidato (`entity_id`).

### RF-04 — Execução do pipeline (fase 2)
- Para o `entity_id` selecionado, o sistema deve:
  1) Consultar cache de análise
  2) Se não houver cache válido, executar conectores das fontes habilitadas
  3) Materializar snapshots (raw/normalized)
  4) Rodar checks de qualidade de dados
  5) Rodar motor **determinístico** de regras / indicadores
  6) (Opcional) Rodar camada IA plugável (multi-agente)
  7) Construir relatório final

### RF-05 — Resultado estruturado e explicável
- O resultado deve conter seções:
  - **summary** (visão geral, sem linguagem acusatória)
  - **entities** (dados canônicos + variantes/aliases)
  - **facts** (dados normalizados por domínio)
  - **indicators** (id, evidência, confiança, limitações, hipóteses alternativas)
  - **ai_findings** (hipóteses assistidas, sempre com `evidence_refs` e limitações)
  - **sources** (lista de fontes consultadas + snapshots)
  - **audit** (run_id, versões, cache_hit, hashes, e metadados dos agentes)

### RF-06 — Cache e auditoria de cache
- Cada consulta (fase 2) deve registrar:
  - `cache.hit` (true/false)
  - `cache.key`, `created_at`, `ttl_seconds`
  - versões usadas (pipeline, conectores, fontes)
  - **ai.config_hash** (para diferenciar cache com IA on/off e múltiplos agentes)

### RF-07 — Logs e rastreabilidade
- A execução deve produzir:
  - logs estruturados por etapa
  - métricas de tempo/linhas/erros
  - registro de proveniência (fonte → transformação → indicador/achado)

### RF-08 — Compartilhamento (opcional / controlado)
- Export/compartilhamento **não pode** incluir PII por padrão.
- Export deve ser um **pacote auditável** com:
  - metadados, snapshots/hashes, metodologia, versões, disclaimers.
- Links compartilháveis (se existirem) devem expirar.

### RF-09 — Camada IA plugável (multi-agente)
- Deve ser possível habilitar/desabilitar IA por:
  - configuração do servidor/CLI, e/ou
  - parâmetro da request (quando permitido).
- Deve suportar **mais de um agente** na mesma execução.
- Cada agente deve retornar achados como:
  - **hipóteses** (não conclusões),
  - com **referência explícita à evidência** (`evidence_refs`),
  - com `confidence`, `limitations` e `alt_hypotheses`.
- O relatório deve registrar:
  - quais agentes rodaram,
  - modelo/versão,
  - versão do prompt/policy,
  - hashes de entrada/saída.

### RF-10 — Consenso e divergência entre agentes
- Deve existir um agregador de resultados:
  - agrupar achados semelhantes (mesma evidência/mesmo padrão)
  - atribuir `consensus` (ex.: quantos agentes concordaram)
  - registrar divergências (ex.: achado levantado por 1 agente apenas)
- O relatório deve apresentar:
  - `ai_findings[]` com `consensus.count` e `consensus.agents[]`.

---

## 3) Requisitos Não Funcionais (RNF)

### RNF-01 — Segurança e Privacidade
- Não logar nem persistir identificadores sensíveis por padrão.
- Segredos fora do código (env/secret manager).
- Rate limiting (anti-enumeração e anti-abuso).
- IA (se houver) deve receber **payload sanitizado** (sem PII por padrão).

### RNF-02 — Auditabilidade e Reprodutibilidade
- Mesmos inputs normalizados + mesmas versões ⇒ mesmo output determinístico.
- Cada run deve gerar `run_record` com hashes e versões.
- Para IA:
  - registrar `model_id`, `model_version`, `prompt_version`, `policy_version`
  - registrar `input_hash` e `output_hash`
  - se o provedor permitir, usar parâmetros para reduzir variação (ex.: temperature=0)
  - se ainda houver variação, registrar "nondeterministic=true" e manter o output como artefato do run.

### RNF-03 — Explicabilidade e Conservadorismo
- Todo indicador deve explicar "por quê".
- O sistema deve listar falsos positivos prováveis e hipóteses alternativas legítimas.
- Sem conclusões automáticas.
- IA não pode introduzir linguagem acusatória; deve produzir **hipóteses técnicas** com evidência.

### RNF-04 — Performance e Escalabilidade (MVP → evolução)
- Busca (fase 1) deve ser rápida (< 300ms ideal).
- Processamento (fase 2) pode ser assíncrono, mas deve ter progresso/estado.
- Conectores paralelizáveis com limites (rate limit por fonte).
- IA:
  - deve ser opcional;
  - deve ter timeouts e limites de custo/tempo por agente.

### RNF-05 — Extensibilidade (plug-and-play)
- Adicionar uma nova fonte deve exigir:
  - um módulo de conector
  - um transformer (mapeamento para schema canônico)
  - testes mínimos + metadados de fonte
- Adicionar um agente IA deve exigir:
  - implementar interface `AiAgent`
  - definir `agent_manifest` (capabilities, inputs, outputs)
  - testes com fixtures sintéticas

### RNF-06 — Observabilidade
- Telemetria por etapa (traces/métricas/logs).
- Correlation ID (`run_id`) em toda a execução.
- Spans dedicados:
  - `connectors/*`, `transform/*`, `quality/*`, `indicators/*`, `ai/*`, `report/*`.

---

## 4) Requisitos Arquiteturais (Plug-and-Play)

### 4.1 Componentes principais
1) **Search Engine (catálogo)**: resolve texto → candidatos (`entity_id`)
2) **Orchestrator (análise)**: coordena pipeline fase 2
3) **Connectors**: ingestão por fonte
4) **Transformers**: normalização para schema canônico
5) **Quality Gate**: checks mínimos antes de indicadores
6) **Deterministic Indicators Engine**: calcula indicadores documentados (determinístico)
7) **AI Agent Orchestrator (opcional)**: executa N agentes e agrega resultados
8) **AI Agents (plug-ins)**: agentes especializados (consistência, anomalias, triagem, explicação)
9) **Report Builder**: consolida JSON final
10) **Cache Manager**: TTL + versionamento + cache_hit (+ ai.config_hash)
11) **Audit Store**: run_record + lineage + decisões + cache metadata + ai metadata
12) **Observability**: logs, métricas, traces

### 4.2 Contratos (interfaces) — sugestão

#### Connector
- Responsável por: baixar/consultar fonte e produzir snapshot bruto.
- Inputs:
  - `entity_id`, `query_context`, `source_config`
- Outputs:
  - `RawSnapshot { snapshot_id, source, collected_at, uri, hash, payload_ref }`

#### Transformer
- Responsável por: mapear raw → schema canônico.
- Inputs:
  - `RawSnapshot`
- Outputs:
  - `NormalizedDataset { schema_version, rows_ref, hash }`

#### Quality Check
- Inputs:
  - `NormalizedDataset`
- Outputs:
  - `QualityReport { passed, checks[], severity, notes }`

#### Indicator (determinístico)
- Inputs:
  - `EntityProfile` + `NormalizedDatasets[]` + `QualityReports[]`
- Outputs:
  - `IndicatorResult { id, evidence, confidence, limitations, alt_hypotheses }`

#### AiAgent (plugável)
- Regra: agente **não cria fato**; ele só produz **hipóteses** referenciando evidências existentes.
- Inputs (sanitizados):
  - `AiContext { entity_profile, facts, indicators, sources, quality_reports, constraints }`
- Outputs:
  - `AiFinding { id, title, description, confidence, evidence_refs[], limitations, alt_hypotheses, consensus?, agent_meta }`
- Campos obrigatórios:
  - `evidence_refs[]`: referências (JSON Pointer) para `facts/indicators/sources` usados
  - `agent_meta`: `{ agent_id, agent_version, model_id, model_version, prompt_version, policy_version }`

#### AiAgentOrchestrator (multi-agente)
- Inputs:
  - `AiContext` + `agents[]` + `timeouts/budgets`
- Outputs:
  - `AiFindingsAggregate { findings[], consensus_map, disagreements[] }`
- Responsabilidade:
  - deduplicar achados semelhantes
  - computar consenso (contagem e lista de agentes)
  - aplicar "policy guard" (bloquear linguagem proibida / PII)

> Observação: Indicators e Transformers devem ser **determinísticos** e versionados. IA é **opcional** e deve ser auditada como "assistência", não como verdade.

---

## 5) Fluxo principal (busca → seleção → processamento)

### 5.1 Visão geral
- **Fase 1 (Search)**: retorna candidatos. Não executa conectores.
- **Fase 2 (Analyze)**: após seleção, executa pipeline e gera relatório.
- **Fase 2b (AI Assist, opcional)**: roda após indicadores determinísticos e antes do report final.


## 5.2 Contrato do Relatório (/analyze) — JSON estruturado

O endpoint `/analyze` deve retornar um relatório com **seções fixas**, incluindo `ai_findings` e `audit.ai[]`.

### Estrutura (alto nível)

```json
{
  "summary": { },
  "entities": { },
  "facts": { },
  "indicators": [ ],
  "ai_findings": [ ],
  "sources": [ ],
  "audit": {
    "run_id": "…",
    "created_at": "…",
    "pipeline_version": "…",
    "params_hash": "sha256:…",
    "cache": {
      "hit": false,
      "key": "sha256:…",
      "created_at": "…",
      "ttl_seconds": 900
    },
    "snapshots": [ ],
    "ai": [ ]
  }
}
```

A seguir estão os trechos **prontos** para adicionar no ROADMAP (ou na especificação do `/analyze`) incluindo:

* `ai_findings` (no corpo do relatório)
* `audit.ai[]` (na trilha de auditoria)

Você pode copiar/colar exatamente como está.

---

## 5.3 `ai_findings[]` — Achados assistidos por IA (hipóteses)

> `ai_findings` **não** é “verdade”, “prova” ou “acusação”.
> É uma lista de **hipóteses técnicas** geradas por agentes plugáveis, sempre com **referência explícita às evidências** presentes no relatório (`facts`, `indicators`, `sources`).

### Campos obrigatórios

Cada item em `ai_findings[]` deve conter:

* `finding_id` (string, único no run)
* `title` (string curta)
* `description` (string; linguagem neutra, técnica)
* `confidence` (`low|medium|high`)
* `evidence_refs[]` (array de referências para evidências no relatório)
* `limitations[]` (array de strings)
* `alt_hypotheses[]` (array de strings)
* `consensus` (agregação multi-agente)
* `agent_refs[]` (quais agentes contribuíram)

### Exemplo

```json
{
  "finding_id": "AI_ANOMALY_001",
  "title": "Variação atípica em valores no período",
  "description": "Foi identificado um desvio robusto (MAD/IQR) em valores agregados em comparação com períodos adjacentes. Este achado é uma hipótese técnica e pode ser explicado por atraso de atualização, mudança de schema ou reprocessamento oficial.",
  "confidence": "medium",
  "evidence_refs": [
    "/indicators/2",
    "/facts/expenses/monthly/2024-08",
    "/sources/1"
  ],
  "limitations": [
    "Cobertura temporal incompleta em uma das fontes",
    "Possível mudança de schema no período analisado"
  ],
  "alt_hypotheses": [
    "Atraso na atualização do portal",
    "Reclassificação administrativa do gasto",
    "Reprocessamento retroativo"
  ],
  "consensus": {
    "count": 2,
    "agents": ["consistency-agent", "anomaly-hypothesis-agent"]
  },
  "agent_refs": [
    { "agent_id": "consistency-agent", "run_ref": "/audit/ai/0" },
    { "agent_id": "anomaly-hypothesis-agent", "run_ref": "/audit/ai/1" }
  ]
}
```

### Regras obrigatórias

* `evidence_refs[]` deve apontar somente para caminhos existentes no JSON (ex.: JSON Pointer).
* O texto **não pode** usar linguagem acusatória.
* O achado **não pode** introduzir fatos novos fora das evidências referenciadas.

---

## 5.4 `audit.ai[]` — Auditoria de execução dos agentes

> `audit.ai[]` registra **como** e **com quais parâmetros** cada agente rodou, garantindo rastreabilidade e controle de risco.

### Campos obrigatórios por item

* `agent_id`
* `agent_version`
* `provider` (ex.: local|remote|custom)
* `model_id`
* `model_version`
* `prompt_version`
* `policy_version`
* `input_hash` (sha256 do payload sanitizado entregue ao agente)
* `output_hash` (sha256 da saída bruta do agente)
* `duration_ms`
* `status` (`success|timeout|error|skipped`)
* `nondeterministic` (boolean)
* `budget` (limites de custo/tempo/tokens, quando aplicável)
* `notes` (opcional, sem PII)

### Exemplo

```json
{
  "audit": {
    "ai": [
      {
        "agent_id": "consistency-agent",
        "agent_version": "1.0.0",
        "provider": "local",
        "model_id": "llm-local",
        "model_version": "2026-02-01",
        "prompt_version": "3",
        "policy_version": "2",
        "input_hash": "sha256:aaa...",
        "output_hash": "sha256:bbb...",
        "duration_ms": 842,
        "status": "success",
        "nondeterministic": false,
        "budget": { "timeout_ms": 3000, "max_tokens": 1200 },
        "notes": "input sanitizado (sem PII)"
      },
      {
        "agent_id": "anomaly-hypothesis-agent",
        "agent_version": "1.0.0",
        "provider": "remote",
        "model_id": "gpt-x",
        "model_version": "2026-02-15",
        "prompt_version": "5",
        "policy_version": "2",
        "input_hash": "sha256:ccc...",
        "output_hash": "sha256:ddd...",
        "duration_ms": 1660,
        "status": "success",
        "nondeterministic": true,
        "budget": { "timeout_ms": 8000, "max_tokens": 1800 },
        "notes": "variação possível; output preservado como artefato do run"
      }
    ]
  }
}
```

### Regras obrigatórias

* `input_hash` e `output_hash` sempre presentes (mesmo em erro/timeout, quando possível).
* `nondeterministic=true` quando usar modelos não determinísticos ou parâmetros não fixos.
* Logs associados à IA devem ser **sem PII** por padrão.


---

## 6) Diagrama de Sequência (Mermaid)

```mermaid
sequenceDiagram
  autonumber
  actor U as Usuário
  participant UI as Web UI (Next.js)
  participant S as Search API
  participant EC as Entity Catalog (Index/DB)
  participant A as Analyze API (Orchestrator)
  participant C as Cache Manager
  participant AR as Audit Store
  participant CR as Connector Runner
  participant F1 as Conector: PortalTransparencia
  participant F2 as Conector: PNCP
  participant T as Transformers
  participant Q as Quality Gate
  participant R as Deterministic Indicators Engine
  participant AI as AI Agent Orchestrator
  participant AG1 as AI Agent #1
  participant AG2 as AI Agent #2
  participant RB as Report Builder

  U->>UI: Digita termo (ex.: "João Silva")
  UI->>S: GET /search?q=...
  S->>EC: consulta catálogo (nome/aliases/contexto)
  EC-->>S: candidatos + confidence + explanation
  S-->>UI: lista de candidatos

  U->>UI: Seleciona candidato (entity_id)
  UI->>A: POST /analyze { entity_id, sources[], params, ai_agents? }

  A->>C: check cache(key = hash(entity_id+versions+params+ai_config))
  alt cache_hit
    C-->>A: report cached + cache metadata
    A->>AR: registrar run_record (cache_hit=true)
    A-->>UI: relatório estruturado
  else cache_miss
    C-->>A: miss
    A->>CR: iniciar execução pipeline (run_id)

    CR->>F1: coletar snapshot (entity_id)
    CR->>F2: coletar snapshot (entity_id)
    F1-->>CR: raw snapshot + hash + collected_at
    F2-->>CR: raw snapshot + hash + collected_at

    CR->>T: normalizar snapshots -> schema canônico
    T-->>CR: normalized datasets + hashes

    CR->>Q: validar qualidade
    Q-->>CR: quality reports (pass/fail + severidade)

    CR->>R: calcular indicadores determinísticos
    R-->>CR: indicator results (evidence + confidence + limitations)

    opt IA habilitada (N agentes)
      CR->>AI: executar agentes (AiContext sanitizado)
      AI->>AG1: analyze(context)
      AI->>AG2: analyze(context)
      AG1-->>AI: findings
      AG2-->>AI: findings
      AI-->>CR: findings agregados + consenso/divergência
    end

    CR->>RB: montar relatório (sources + facts + indicators + ai_findings + audit)
    RB-->>CR: report json

    CR->>C: salvar cache(report + TTL + versions + ai_config_hash)
    CR->>AR: salvar run_record + lineage + cache_hit=false + ai_meta
    A-->>UI: relatório estruturado
  end
````

---

## 7) Arquitetura (Mermaid) — visão de containers/componentes

```mermaid
---
config:
  layout: elk
---
flowchart TB
 subgraph Client["Client"]
        UI["Web UI (Next.js) - busca - lista de candidatos - seleção - exibição do relatório"]
  end
 subgraph API["Backend (opcional) — serviços"]
        SVC_SEARCH["Search Service - normaliza query - consulta catálogo - retorna candidatos"]
        SVC_ANALYZE["Analyze Orchestrator - run_id - cache gate - coordena pipeline"]
        CACHE["Cache Manager - TTL - cache_hit - versioned keys - ai_config_hash"]
        AUDIT["Audit Store - run_record - lineage - decisions - cache metadata - ai metadata"]
  end
 subgraph Data["Data Plane (local-first / snapshots)"]
        CAT["Entity Catalog / Index (IDs canônicos, aliases, contexto)"]
        RAW["Raw Snapshots Store (Parquet/JSONL + hashes)"]
        NORM["Normalized Store (schema canônico)"]
        WH["Warehouse Local (DuckDB) (facts, joins, indicadores)"]
  end
 subgraph Pipeline["Pipeline Engine (plug-and-play)"]
        RUNNER["Connector Runner (paralelo + rate limit)"]
        CONN1["Connector A (PortalTransparencia)"]
        CONN2["Connector B (PNCP)"]
        CONNn["Connector N ..."]
        TR["Transformers (mapeamento -> schema canônico)"]
        QA["Quality Gate (checks mínimos)"]
        IND["Deterministic Indicators Engine (indicadores documentados)"]
        AIO["AI Agent Orchestrator (opcional) - executa N agentes - policy guard - consenso/divergência"]
        AGS["AI Agents (plugins) - consistency - anomaly-hypotheses - false-positive reviewer - explainer/triage"]
        REPORT["Report Builder (summary/facts/indicators/ai_findings/audit)"]
  end
 subgraph Obs["Observabilidade"]
        LOGS["Logs estruturados (no PII)"]
        METRICS["Métricas (latência/linhas/erros)"]
        TRACES["Traces (OpenTelemetry) (run_id como correlation)"]
  end
    UI -- GET /search --> SVC_SEARCH
    SVC_SEARCH --> CAT & UI & LOGS & METRICS
    UI -- POST /analyze --> SVC_ANALYZE
    SVC_ANALYZE --> CACHE & AUDIT & RUNNER & LOGS & METRICS & TRACES
    RUNNER --> CONN1 & CONN2 & CONNn & LOGS & TRACES
    CONN1 --> RAW
    CONN2 --> RAW
    CONNn --> RAW
    RAW --> TR
    TR --> NORM
    NORM --> WH & QA
    QA --> IND & LOGS
    IND --> AIO & REPORT & LOGS
    AIO --> REPORT & AGS & LOGS & TRACES
    REPORT --> CACHE & AUDIT & UI
```

---

## 8) Modelo mínimo de dados para auditoria

### 8.1 Tabelas/coleções (conceito)

* `runs`:

  * run_id, created_at, entity_id, pipeline_version, params_hash, cache_hit, status, ai_enabled, ai_config_hash
* `run_sources`:

  * run_id, source_name, snapshot_id, collected_at, hash, connector_version
* `snapshots`:

  * snapshot_id, source, uri/ref, collected_at, hash, schema_observed
* `quality_reports`:

  * run_id, dataset_name, passed, severity, checks_json, notes
* `indicator_results`:

  * run_id, indicator_id, confidence, evidence_json, limitations, alt_hypotheses
* `ai_agent_runs`:

  * run_id, agent_id, agent_version, model_id, model_version, prompt_version, policy_version, input_hash, output_hash, duration_ms, status, nondeterministic
* `ai_findings`:

  * run_id, finding_id, title, confidence, evidence_refs_json, limitations, alt_hypotheses, consensus_count, consensus_agents_json
* `cache_entries`:

  * cache_key, created_at, ttl, run_id, versions_json, ai_config_hash

> Precisa de auditoria para defender a credibilidade da análise e do projeto.

---

## 9) Roadmap (fases e entregáveis)

### Fase 0 — Fundamentos (documentação e governança)

**Entregáveis**

* MANIFESTO, GOVERNANCE, RISK_POLICY, METHODOLOGY, SECURITY, CODE_OF_CONDUCT
* Templates de PR/Issue com labels `risk:*`

**Critério de pronto**

* Processo de risco e vocabulário controlado aplicado.

---

### Fase 1 — MVP Search + Entity Catalog (sem cruzamento)

**Entregáveis**

* Serviço de busca `/search`
* Catálogo mínimo (ex.: TSE snapshot ou outra fonte oficial para IDs)
* UI: busca → lista de candidatos (com confidence/explanation)
* Auditoria básica do search (sem PII)

**Critério de pronto**

* Usuário consegue buscar e selecionar com baixa chance de homônimo.

---

### Fase 2 — MVP Analyze (1–2 fontes) + relatório auditável

**Entregáveis**

* `/analyze` com run_id e status
* 1 conector oficial (ex.: Portal da Transparência OU PNCP)
* 1 transformer para schema canônico mínimo
* quality gate mínimo (10 checks)
* 3–5 indicadores simples, altamente explicáveis
* cache versionado + cache_hit no audit

**Critério de pronto**

* Relatório estruturado com fontes/snapshots/hashes e indicadores sem linguagem acusatória.

---

### Fase 3 — Camada IA plugável (multi-agente) **em modo "hipóteses assistidas"**

**Entregáveis**

* Interface `AiAgent` + `AiAgentOrchestrator`
* Pelo menos 2 agentes de referência:

  * `consistency-agent` (checa coerências e divergências já presentes)
  * `anomaly-hypothesis-agent` (sugere padrões atípicos como hipóteses com `evidence_refs`)
* Agregador de consenso/divergência
* Policy guard (bloqueio de linguagem proibida e PII por padrão)
* Auditoria completa da IA (`ai_agent_runs`, hashes, versões)
* Atualização do cache key com `ai_config_hash`

**Critério de pronto**

* IA adiciona valor sem criar "caixa-preta": todo achado aponta evidência e limitações; logs e auditoria completos.

---

### Fase 4 — Escalar conectores + qualidade + observabilidade

**Entregáveis**

* +2–3 conectores (Câmara/Senado/Compras/Transferegov conforme viabilidade)
* observabilidade com traces/metrics (run_id)
* política de TTL por fonte e invalidation por versão
* regressão de indicadores + "golden datasets"

**Critério de pronto**

* Pipeline estável, repetível e monitorável.

---

### Fase 5 — Entity Resolution avançado (quando necessário)

**Entregáveis**

* linkage probabilístico (ex.: Splink) com calibração
* decisões humanas persistidas (merge/não-merge) em store
* UI de desambiguação avançada (somente quando necessário)

**Critério de pronto**

* Redução mensurável de falso positivo em match.

---

### Fase 6 — Export controlado e "pacote auditável"

**Entregáveis**

* export agregado sem PII por padrão
* pacote com metadados + hashes + versões + metodologia
* links com expiração (se existir)

**Critério de pronto**

* Compartilhamento não vira ferramenta de perseguição.

---

## 10) Critérios de aceitação (MVP)

O MVP só é aceitável se:

1. **Busca retorna lista** com confidence/explanation e não auto-seleciona.
2. **Processamento só após seleção explícita**.
3. Resultados incluem **fontes**, **snapshots**, **hashes**, **versões**.
4. Indicadores determinísticos são explicáveis e exibem **limitações + falsos positivos**.
5. Cache registra `cache_hit` e não armazena PII por padrão.
6. Não existe linguagem acusatória, ranking moral ou "suspeito".

---

## 11) Decisões em aberto (para RFCs)

* Qual será o **catálogo canônico** inicial para entity_id (TSE? Câmara? unificação?)
* Quais 2 fontes entram primeiro (impacto vs risco)?
* Onde o pipeline roda: **100% local-first** (cliente/CLI) vs backend central
* Política de retenção de snapshots e auditoria
* Padrão do schema canônico mínimo (FtM-like vs schema próprio)

* **Política de IA**:

  * agentes permitidos, inputs sanitizados, proibições de linguagem
  * modelos locais vs remotos
  * budget/timeouts e auditoria mínima obrigatória
