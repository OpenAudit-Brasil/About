# METHODOLOGY.md

## OpenAudit Brasil — Metodologia (v1.0.0) 27-02-2026

<img width="1431" height="507" alt="image" src="/assets/logo-full.png" />

> **Objetivo:** definir, de forma auditável e reprodutível, como o OpenAudit Brasil coleta, processa e analisa dados públicos para gerar **indicadores técnicos** (anomalias, padrões atípicos, inconsistências).
>
> **Regra central:** resultados do projeto **não são prova, denúncia, acusação ou afirmação de irregularidade**. São **sinais analíticos** que exigem verificação humana e contextual.
>

**Documentos relacionados:**

- [MANIFESTO.md](./MANIFESTO.md)
- [GOVERNANCE.md](./GOVERNANCE.md)
- [RISK_POLICY.md](./RISK_POLICY.md)
- [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)
- [SECURITY.md](./SECURITY.md)
- [DECISIONS.md](./DECISIONS.md)

---

## 1) Escopo

Esta metodologia cobre:

- seleção e validação de fontes
- coleta (ingestão) e versionamento de dados
- normalização e deduplicação
- regras de qualidade e consistência
- geração de indicadores (estatísticos/heurísticos)
- explicabilidade, incerteza e risco de falso positivo
- rastreabilidade (proveniência) e reprodutibilidade
- controles de privacidade e uso responsável

**Fora do escopo:**
- investigação factual, conclusões jurídicas, imputação de ilícito, "rankings acusatórios" (proibidos por `RISK_POLICY.md`).

---

## 2) Termos normativos

Nesta documentação:

- **MUST / DEVE:** requisito obrigatório.
- **SHOULD / DEVERIA:** recomendado; exceções devem ser justificadas.
- **MAY / PODE:** opcional.

---

## 3) Vocabulário controlado (obrigatório)

### 3.1 Termos permitidos
- indicador técnico
- padrão atípico / anomalia estatística
- inconsistência estrutural
- divergência documental
- sinal analítico
- hipótese alternativa legítima
- limitação / incerteza / falso positivo

### 3.2 Termos proibidos
Qualquer linguagem acusatória (ex.: "fraude", "corrupção", "crime", "suspeito") é proibida em outputs, docs, código e exemplos.
Veja `RISK_POLICY.md`.

---

## 4) Princípios metodológicos

1. **Transparência:** toda regra e métrica deve ser documentada.
2. **Reprodutibilidade:** o mesmo input (com mesma versão) deve gerar o mesmo output.
3. **Explicabilidade:** todo indicador deve responder "por quê" e "como".
4. **Conservadorismo:** na dúvida, reduzir granularidade, elevar incerteza e evitar personalização.
5. **Rastreabilidade:** manter trilha completa do dado (fonte → transformação → indicador).
6. **Falso positivo explícito:** todo indicador deve listar explicações legítimas plausíveis.
7. **Separação:** análise estatística ≠ afirmação de irregularidade.

---

## 5) Ciclo de vida dos dados (pipeline)

### 5.1 Etapas
1. **Source Vetting (validação da fonte)**
2. **Ingest (coleta)**
3. **Staging (armazenamento bruto ou cache local)**
4. **Normalize (padronização de schema e tipos)**
5. **Entity Resolution (resolução de entidades / deduplicação)**
6. **Quality Checks (qualidade e consistência)**
7. **Feature Extraction (variáveis derivadas)**
8. **Indicator Computation (cálculo de indicadores)**
9. **Explain & Package (explicação + output)**
10. **Audit Log (registro de proveniência e decisões)**

### 5.2 Proibição estrutural
O pipeline **NÃO** deve gerar outputs que:
- imputem conduta ilícita
- facilitem perseguição (targeting) ou doxxing
- exponham identificadores sensíveis por padrão

---

## 6) Validação de fontes (Source Vetting)

Antes de integrar uma fonte, deve existir um **Source Record** com:

- nome oficial da fonte / órgão
- URL(s) e endpoints
- termos/licença (quando existirem)
- requisito de autenticação (se exigir login: **proibido**)
- frequência de atualização (estimada)
- cobertura (temporal, geográfica, campos)
- riscos (ToS, privacidade, ambiguidades)
- status: `approved | restricted | rejected`

**DEVE** ser recusada qualquer fonte que:
- exija login/sessão/bypass/captcha (por `RISK_POLICY.md`)
- tenha termos incompatíveis com coleta/uso
- não permita rastreabilidade mínima

---

## 7) Coleta (Ingestão)

### 7.1 Regras de coleta
O coletor **DEVE**:
- respeitar limites e boas práticas (rate limit, backoff, retries controlados)
- registrar timestamp e versão do coletor
- armazenar metadados de origem (URL, parâmetros)
- evitar capturar dados pessoais além do necessário (minimização)

### 7.2 Reprodutibilidade
Toda execução de ingestão **DEVE** gerar um "fingerprint" do lote:

- hash do arquivo/resposta (quando aplicável)
- contagem de registros
- schema observado
- janela temporal (se existir)
- versão do conector

---

## 8) Normalização e schema

### 8.1 Regras
A camada de normalização **DEVE**:
- converter tipos (datas, moedas, inteiros)
- padronizar nomes de campos (snake_case recomendado)
- padronizar timezone e granularidade temporal
- registrar transformações aplicadas
- preservar o bruto (quando permitido) ou, no mínimo, preservar o hash do bruto

### 8.2 Contratos
Cada dataset normalizado **DEVE** ter:
- schema versionado
- dicionário de dados (descrição de campos e tipos)
- regras de validação (constraints)

---

## 9) Resolução de entidades e deduplicação

Para cruzar bases, o projeto utiliza mecanismos de **entity resolution** (ER).

### 9.1 Identificadores
- Para pessoas físicas: usar identificadores sensíveis **é restrito** e mascarado por padrão.
- Para entidades jurídicas (ex.: CNPJ) pode haver menor risco, mas ainda exige cuidado de uso e ToS.

### 9.2 Estratégias (exemplos)
- match determinístico (ID estável quando permitido)
- match probabilístico (nome normalizado + município + janelas temporais + outros atributos)
- blocking + scoring (para escala)

### 9.3 Requisitos
Qualquer algoritmo de ER **DEVE**:
- registrar taxa de match e taxa de conflito
- gerar "match explanation" (por que foi considerado o mesmo)
- permitir fallback conservador ("não match" quando incerteza alta)

---

## 10) Qualidade de dados (Data Quality)

### 10.1 Dimensões avaliadas
- **Completeness:** campos ausentes / cobertura temporal
- **Validity:** formato e domínio de valores
- **Uniqueness:** duplicatas e colisões
- **Consistency:** relações internas coerentes (ex.: totais vs somas)
- **Timeliness:** atraso e frequência de atualização
- **Accuracy (proxy):** cruzamento com fontes independentes quando possível

### 10.2 Saída obrigatória
Toda execução de pipeline **DEVE** produzir:
- relatório de qualidade com métricas básicas
- lista de checks falhos (com severidade)
- impacto da qualidade na confiança dos indicadores

---

## 11) Indicadores (framework)

### 11.1 O que é um indicador
Um **indicador** é uma função documentada que transforma dados em um **sinal analítico**, acompanhado de:

- racional
- fórmula/algoritmo
- parâmetros e thresholds
- hipótese alternativa legítima
- limitações e risco de falso positivo
- "confidence" (confiança) com critérios explícitos

### 11.2 Tipos de indicadores (permitidos)
1. **Inconsistência estrutural**
   - divergência entre campos que deveriam ser coerentes
2. **Divergência documental**
   - discrepância entre duas fontes oficiais sobre o mesmo fato/valor
3. **Padrão atípico estatístico**
   - outliers por distribuição, variação temporal, mudanças abruptas
4. **Anomalia de integridade**
   - duplicações incomuns, quebras de sequência, campos inválidos
5. **Anomalia de cobertura**
   - períodos com ausência súbita de dados, gaps não explicados

### 11.3 Métodos estatísticos (exemplos aceitos)
- z-score (com cautela a não-normalidade)
- IQR / boxplot outliers
- MAD (Median Absolute Deviation) para robustez
- detecção de mudança (change-point) em séries temporais
- comparação de distribuições (ex.: KS test) quando aplicável
- regras baseadas em domínio (heurísticas explícitas)

> Qualquer método deve ser documentado com premissas e condições de uso.

### 11.4 Thresholds e sensibilidade
Thresholds **NÃO** podem ser "mágicos". Devem ser:
- derivados de estatística robusta, ou
- calibrados em conjunto com validação histórica, ou
- definidos como configuráveis com defaults conservadores

---

## 12) Incerteza, confiança e falsos positivos

### 12.1 Confiança (Confidence)
Cada indicador deve emitir um nível de confiança (`low | medium | high`) baseado em critérios objetivos, por exemplo:

- qualidade do dado (completeness/validity)
- estabilidade temporal
- redundância (confirmação por múltiplas fontes)
- sensibilidade a parâmetros
- taxa de match em ER (quando aplicável)

### 12.2 Falso positivo (obrigatório)
Cada indicador **DEVE** listar explicações legítimas comuns, por exemplo:

- atraso de atualização da fonte
- mudanças de schema/campo pela origem
- agregação por período diferente
- erro de digitação/ETL na fonte
- mudanças administrativas (fusão, desmembramento, reclassificação)
- duplicidade causada por reprocessamento oficial

### 12.3 Output defensivo
Quando confiança for `low`, o sistema **DEVE**:
- reduzir a ênfase visual
- aumentar disclaimers
- exigir mais contexto antes de permitir exportação/compartilhamento (se existir)

---

## 13) Explicabilidade (Explainability) — formato mínimo

Todo indicador exibido **DEVE** incluir:

1. **Resumo técnico** do sinal
2. **Evidência numérica** (métrica + valores relevantes)
3. **Fonte(s)** e janela temporal
4. **Como foi calculado** (regra + parâmetros)
5. **Limitações** e hipóteses alternativas legítimas
6. **Confiança** e por quê

---

## 14) Apresentação de resultados (sem acusação)

### 14.1 Regras
Outputs **DEVEM**:
- usar vocabulário controlado
- incluir bloco de disclaimer (ver `RISK_POLICY.md`)
- evitar inferências morais
- evitar personalização (foco em "registro/dataset", não em "alvo")

### 14.2 Proibição
Não é permitido:
- "conclusão automática"
- "recomendação de denúncia"
- rotular como crime específico
- rankings acusatórios

---

## 15) Validação (técnica e metodológica)

### 15.1 Tipos de validação
- **Unit tests:** para funções e transformações
- **Property-based tests:** onde aplicável (robustez)
- **Golden datasets:** datasets sintéticos e controlados
- **Regression tests:** garantir estabilidade de indicadores
- **Backtesting (quando possível):** comparar comportamento histórico

### 15.2 Requisito mínimo para novo indicador
Um novo indicador só entra se houver:
- spec completa (ver template)
- testes automatizados
- exemplos sintéticos
- descrição de falsos positivos
- avaliação de risco (`risk:medium` ou `risk:high` quando envolver pessoas/identificadores)

---

## 16) Rastreabilidade e proveniência

### 16.1 Registro de execução (Run Record)
Cada run do pipeline **DEVE** gerar um registro com:

- versão do código (commit hash)
- versões dos conectores
- fingerprint dos dados (hash + contagens)
- config/parâmetros relevantes
- checks de qualidade (pass/fail)
- indicadores gerados (IDs + parâmetros)
- data/hora e ambiente (local/ci)

### 16.2 Exemplo de schema (referência)

```json
{
  "run_id": "2026-02-27T22:10:00Z__local__abc123",
  "code_version": "abc123",
  "connectors": [{"name": "source_x", "version": "1.2.0"}],
  "datasets": [{
    "name": "dataset_norm",
    "schema_version": "2026-02-01",
    "fingerprint": {"hash": "sha256:...", "rows": 12345, "cols": 28}
  }],
  "quality": {"checks_failed": 1, "notes": ["missingness spike in field_y"]},
  "indicators": [{"id": "IND_TIME_GAP_01", "params": {"gap_days": 30}, "confidence": "low"}]
}

```

---

## 17) Privacidade e minimização (conforme LGPD)

A metodologia **DEVE** seguir `RISK_POLICY.md`:

* mascaramento por padrão
* não exportar identificadores por padrão
* não indexar para busca reversa
* reduzir granularidade sempre que possível
* preferir dados sintéticos em testes e exemplos

Qualquer exceção exige risk review e registro em `DECISIONS.md`.

---

## 18) Limitações e vieses

O projeto reconhece limitações estruturais:

* **viés de disponibilidade:** dados publicados podem ser incompletos ou seletivos
* **viés de atualização:** atrasos e revisões retroativas
* **viés de schema:** mudanças de formato geram "anomalias falsas"
* **viés de modelagem:** thresholds e métodos podem favorecer certos padrões
* **viés de matching:** ER pode errar (falsos matches / misses)

O output **DEVE** refletir essas limitações via confiança e explicações.

---

## 19) Mudanças metodológicas e controle de versão

### 19.1 Quando uma mudança é "metodológica"

Qualquer alteração que:

* mude cálculo de indicador
* mude thresholds
* mude normalização/ER
* mude linguagem/semântica de outputs
* mude política de agregação/privacidade

…é mudança metodológica.

### 19.2 Processo obrigatório

Mudanças metodológicas exigem:

* RFC quando aplicável (`GOVERNANCE.md`)
* atualização deste arquivo (`METHODOLOGY.md`)
* registro em `DECISIONS.md`
* testes de regressão

---

## 20) Template obrigatório de indicador (SPEC)

Todo indicador deve possuir um arquivo/spec com:

```md
### ID: IND_XXXX_YY
**Nome:** …
**Tipo:** (inconsistência | divergência | padrão atípico | integridade | cobertura)
**Objetivo:** …
**Fontes/Datasets:** …
**Pré-requisitos de qualidade:** …
**Variáveis usadas:** …
**Método:** (fórmula/algoritmo)
**Parâmetros:** (defaults e justificativa)
**Thresholds:** (como foram definidos)
**Output:** (campos retornados)
**Explicabilidade:** (o que mostrar ao usuário)
**Falsos positivos prováveis:** …
**Hipóteses alternativas legítimas:** …
**Risco e mitigação:** (LGPD/difamação/uso indevido)
**Testes:** (unit/golden/regression)
**Versão:** …
**Referências:** (quando aplicável)
```

---

## 21) Declaração final

O OpenAudit Brasil é uma infraestrutura técnica para produzir **indicadores auditáveis** a partir de dados públicos.

* **Não acusa**
* **Não conclui**
* **Não personaliza**
* **Não facilita perseguição**

A metodologia existe para garantir que cada resultado seja:

* rastreável
* explicável
* reprodutível
* conservador quanto a risco e falso positivo

---
