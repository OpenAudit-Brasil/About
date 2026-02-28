# POLÍTICA DE RISCO

<img width="1431" height="507" alt="image" src="/assets/logo-full.png" />

## OpenAudit Brasil — Política de Risco (v1.0.0) 27-02-2026

> **Objetivo:** reduzir risco jurídico, técnico, reputacional e de uso indevido em um projeto open source que analisa **dados públicos**.
>
> **Regra de ouro:** o projeto **não acusa**, **não julga** e **não facilita perseguição**. Ele produz **indicadores técnicos** com **limitações explícitas**.
>
> Documentos relacionados: `MANIFESTO.md`, `GOVERNANCE.md`, `CODE_OF_CONDUCT.md`, `METHODOLOGY.md`, `SECURITY.md`, `DECISIONS.md`.

---

## 1) Escopo e aplicabilidade

Esta política se aplica a **tudo** que o projeto produz ou publica:

- código-fonte, exemplos, fixtures, seeds, datasets de teste
- outputs de UI/API/CLI/relatórios/exportações
- documentação, posts oficiais, releases, changelogs
- pipelines de coleta/ETL, conectores e armazenamento
- artefatos de build e logs

Qualquer contribuição que viole esta política **não será aceita**.

---

## 2) Definições (normativas)

### 2.1 Dado "público" vs "apenas acessível"
- **Público (para o projeto):** disponibilizado oficialmente, sem autenticação, sem bypass, com uso compatível com termos/licença.
- **Apenas acessível:** está na internet, mas condicionado a login, sessão, paywall, captcha, restrições contratuais ou proibido por termos.
- **Regra:** "está online" **não** implica "pode ser coletado/replicado".

### 2.2 Dado pessoal
Qualquer informação relacionada a pessoa natural identificada ou identificável, **mesmo se constar em base pública**.

### 2.3 Identificador sensível
Inclui, mas não se limita a: CPF, RG, título de eleitor, CNH, passaporte, endereço completo, telefone, e-mail pessoal, coordenadas precisas, biometria.

### 2.4 Output
Qualquer resultado apresentado ao usuário, inclusive prints, links compartilháveis e exemplos na documentação.

### 2.5 Indicador / Padrão atípico
Resultado estatístico/heurístico que indica "fora do esperado" **conforme metodologia documentada**.  
**Nunca** é prova, denúncia, acusação ou confirmação de irregularidade.

---

## 3) Princípios obrigatórios de mitigação

O projeto **DEVE** seguir:

1. **Minimização:** coletar, armazenar e exibir o mínimo necessário.
2. **Neutralidade:** linguagem técnica; sem juízo moral; sem "culpados".
3. **Explicabilidade:** toda análise deve apontar "por que" e "como".
4. **Incerteza explícita:** apresentar limites, vieses e possibilidade de falso positivo.
5. **Rastreabilidade:** fonte, data, versão do pipeline, parâmetros e mudanças relevantes.
6. **Não-personalização:** evitar design que transforme pessoas em "alvos".
7. **Reversibilidade:** mudanças sensíveis exigem plano de rollback.
8. **Conservadorismo:** na dúvida, restringir, mascarar, agregar ou remover.

---

## 4) Proibições absolutas (ZERO tolerância)

### 4.1 Linguagem acusatória
É proibido, em **qualquer** lugar (UI, docs, variáveis, endpoints, labels, commits), termos como:
- "corrupção", "fraude", "crime", "ilegal", "desvio", "lavagem", "culpado", "prova", "denúncia", "suspeito", "investigado", "condenado"
ou equivalentes.

**Apenas permitido:** "indicador", "anomalia", "padrão atípico", "inconsistência", "divergência", "necessita verificação".

### 4.2 Rankings acusatórios
Proibido:
- "top suspeitos", "score de corrupção", "ranking de fraude", "lista de criminosos"
- qualquer "score" que sugira culpabilidade ou gravidade moral.

### 4.3 Doxxing / targeting
Proibido:
- facilitar perseguição por filtros com alta granularidade (ex.: rua, prédio, coordenada)
- expor dados pessoais não estritamente necessários
- instruções de uso do projeto para "caçar" indivíduos.

### 4.4 Scraping proibido
Proibido:
- bypass de captcha, autenticação, sessão, rate-limit evasivo
- coleta em área logada ou proibida por termos
- engenharia para contornar restrições de acesso.

### 4.5 Distribuição de dataset com dados pessoais identificáveis
Proibido hospedar/distribuir "base própria" contendo dados pessoais identificáveis como default do projeto.

---

## 5) Regras obrigatórias de Output (UI/API/CLI/Relatórios)

### 5.1 Disclaimers obrigatórios (padrão mínimo)
Todo output de análise **DEVE** incluir, de forma visível (ou acessível via atalho):

**Bloco A — Não acusação**
> "Este resultado é um **indicador técnico**. **Não constitui prova, denúncia, acusação ou afirmação de irregularidade.**"

**Bloco B — Limitações**
> "Resultados dependem da qualidade/completude dos dados e podem conter **falsos positivos**."

**Bloco C — Metodologia**
> "Metodologia e critérios: ver `METHODOLOGY.md` (versão 1.0.0)."

### 5.2 Explicabilidade mínima por indicador
Cada indicador exibido **DEVE** fornecer:
- fonte(s) e data/hora de coleta
- regra/modelo aplicado
- evidência numérica (ex.: desvio, percentil, z-score, consistência temporal, etc.)
- hipótese alternativa legítima (ex.: atraso de atualização, mudança de formato, erro de origem)
- nível de confiança e fatores que reduzem confiança

### 5.3 Sem "conclusão automática"
Outputs **NÃO** podem:
- sugerir crime específico
- orientar "denúncia" automática
- recomendar ação contra pessoa/entidade
- usar linguagem de certeza ("confirmado", "comprovado").

### 5.4 UX defensiva (anti-abuso)
Quando houver busca por termos que possam identificar pessoas:
- exigir "modo responsável" (aceite explícito de termos) e
- reforçar disclaimers e
- aplicar mascaramento por padrão.

---

## 6) Política de dados (coleta, armazenamento e publicação)

### 6.1 Coleta
Toda fonte **DEVE** ter:
- origem oficial clara (ou permissão/licença compatível)
- documentação do conector
- registro de termos/licença quando houver
- controles de rate-limit e respeito a robôs/termos

**Obrigatório registrar:** fonte, timestamp, versão do coletor, checksums quando aplicável.

### 6.2 Armazenamento
Padrão do projeto:
- **processamento local por default**
- **sem retenção** de dados pessoais em logs
- retenção mínima e configurável

Se houver armazenamento (local ou servidor):
- definir política de retenção (`RETENTION_DAYS`)
- expurgo automático e verificável
- separar dados brutos vs derivados
- criptografia em repouso quando aplicável
- controle de acesso e trilha de auditoria

### 6.3 Publicação
Por padrão, o projeto publica:
- código + metodologia + exemplos com **dados sintéticos**
- resultados agregados e não identificáveis, quando necessário

Qualquer publicação de dados derivados exige:
- agregação/anonimização
- redução de granularidade
- revisão de risco (`risk-review:required`)
- decisão registrada em `DECISIONS.md`

---

## 7) Dados pessoais (LGPD)

### 7.1 Regra base
Mesmo em base pública, dado pessoal pode gerar risco. Logo:
- **não exibir identificadores por padrão**
- **não exportar identificadores por padrão**
- **não indexar para busca reversa** (ex.: "ache todos por CPF/email/telefone")

### 7.2 Mascaramento obrigatório
Quando for inevitável:
- CPF: `9**.***.***-99`
- e-mail: `a***@dominio.com`
- telefone: `(**) *****-**34`
- endereço: apenas `cidade/UF` (padrão)

Exceções exigem:
- justificativa técnica
- mitigação explícita
- aprovação de Risk Reviewer
- registro em `DECISIONS.md`

### 7.3 Granularidade mínima
Preferir:
- município/UF e períodos (mês/ano) em vez de endereços e datas exatas
- faixas/intervalos em vez de valores pontuais quando possível

---

## 8) Classificação de risco e gates obrigatórios

### 8.1 Níveis de risco (labels obrigatórias)
- `risk:low`  — refactor, docs neutras, testes, CI
- `risk:medium` — novo conector público, mudança de regra analítica sem dados pessoais
- `risk:high` — qualquer coisa com pessoas/identificadores, export, indexação, ranking, dataset, links compartilháveis, UI/UX de busca sensível

### 8.2 Gate de aprovação
- `risk:low`: 1 Reviewer (ou Maintainer)
- `risk:medium`: 1 Maintainer + 1 Reviewer
- `risk:high`: 1 Maintainer + 1 Risk Reviewer + registro em `DECISIONS.md`

---

## 9) Lista de features proibidas e restritas

### 9.1 Proibidas (não entram)
- "score de corrupção/fraude"
- "top N suspeitos"
- alertas/monitoramento por pessoa
- bot de denúncia/compartilhamento direcionado
- mapa de "alvos" com alta granularidade
- instruções de "investigação" com acusação

### 9.2 Restritas (somente com design defensivo + risk review)
- busca por nome (quando necessário)
- compartilhamento de links (exigir expiração e disclaimers)
- exportação (somente agregado e com filtros de privacidade)
- qualquer UI que facilite "campanhas" (ex.: templates de postagem)

---

## 10) Conteúdo e comunicação pública

Canais oficiais do projeto **NÃO** devem:
- citar indivíduos como alvos
- usar linguagem de "expor", "derrubar", "caçar"
- associar o projeto a partido/campanha

Comunicação oficial **DEVE**:
- remeter à metodologia
- reforçar limitações e não-acusação
- manter tom técnico

---

## 11) Checklist obrigatório para PRs sensíveis (risk:high)

Todo PR `risk:high` deve responder (no template de PR):

1. Qual dado pessoal aparece? Por quê é necessário?
2. Como você minimizou/excluiu identificadores?
3. Que partes da UI/outputs podem ser interpretadas como acusação?
4. Quais falsos positivos são prováveis?
5. Como o usuário é avisado (disclaimer + explicação)?
6. Qual é o plano de rollback?
7. Logs e telemetria não vazam PII?

Sem isso: **não mergeia**.

---

## 12) Resposta a incidentes (uso indevido / exposição)

Considera-se incidente:
- vazamento/exposição de dado pessoal
- output com linguagem acusatória
- feature que facilite doxxing/targeting
- abuso coordenado nos canais do projeto

Ações (podem pular etapas):
1. remover/neutralizar imediatamente (revert/hotfix)
2. revogar permissões e aplicar CoC
3. publicar nota técnica (sem amplificar dados sensíveis)
4. registrar post-mortem (quando apropriado)
5. atualizar política/metodologia para evitar repetição

---

## 13) Registro, auditoria e rastreabilidade

Decisões de risco devem ser registradas em `DECISIONS.md` com:
- contexto
- alternativas consideradas
- riscos e mitigações
- responsáveis e data
- impacto em outputs/metodologia

Mudanças de linguagem/terminologia devem ser tratadas como mudança de risco.

---

## 14) Cláusula de interpretação conservadora

Quando houver dúvida:
1. reduzir granularidade
2. mascarar ou remover identificadores
3. exigir revisão de risco
4. documentar limitações
5. preferir execução local e não hospedada

---

## 15) Compromisso final

O OpenAudit Brasil existe para criar infraestrutura aberta e auditável de análise responsável.

Se uma contribuição:
- aumenta risco de difamação,
- facilita perseguição,
- expõe dados pessoais,
- ou politiza o projeto,

ela será rejeitada, mesmo que seja tecnicamente "boa".

---
