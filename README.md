# 🛰 OpenAudit Brasil

<img width="1431" height="507" alt="Logo OpenAudit Brasil" src="/assets/logo-full.png" />

Infraestrutura aberta para **análise responsável**, **reprodutível** e **auditável** de dados públicos.

---

## 📚 Documentação (índice)

- **Fundamentos**
  - [Manifesto](./MANIFESTO.md)
  - [Metodologia](./METHODOLOGY.md)
  - [Política de Risco](./RISK_POLICY.md)
  - [Governança](./GOVERNANCE.md)
  - [Código de Conduta](./CODE_OF_CONDUCT.md)
  - [Segurança](./SECURITY.md)
  - [Registro de Decisões](./DECISIONS.md)

- **Contribuição**
  - [Licença](./LICENSE)
  - [Roadmap](./ROADMAP.md)


- **Fontes de Dados**
  - [Dados](./DATA_SOURCES.md)

---

## 📌 Sobre

A OpenAudit Brasil é uma iniciativa open source dedicada a construir um **ecossistema técnico** para:

- **Cruzamento de bases públicas** (múltiplas fontes oficiais)
- **Detecção de padrões atípicos** e **inconsistências estruturais**
- **Geração de hipóteses analíticas** para verificação humana
- **Rastreabilidade total**: fonte → transformação → indicador → explicação
- **Explicabilidade**: cada resultado precisa responder “por quê” e “como”

O projeto **não realiza acusações**, **não emite juízo moral** e **não substitui autoridades**.  
O output do sistema é **indicador técnico**, acompanhado de limitações e risco de falso positivo.

---

## 🎯 Missão

Construir infraestrutura aberta para que qualquer pessoa possa:

- **auditar metodologia** (código e regras),
- **reproduzir resultados** (mesmos inputs → mesmos outputs),
- e **entender limitações** (incerteza, viés, falsos positivos),

sem depender de plataformas fechadas, interesses institucionais ou serviços centralizados.

---

## 🧠 O que o OpenAudit Brasil é (e não é)

### ✅ É
- Infraestrutura técnica para **análise estruturada** de dados públicos
- Framework para **indicadores auditáveis** (anomalia, divergência, inconsistência)
- Pipeline orientado a **qualidade de dados**, **proveniência** e **reprodutibilidade**
- Projeto **neutro**, com governança e política de risco explícitas

### ❌ Não é
- Ferramenta para “expor” indivíduos
- Sistema de “ranking” acusatório
- Mecanismo de denúncia automatizada
- Órgão investigativo, jurídico ou fiscalizador

Esses limites são parte central do projeto e estão detalhados em:
- [MANIFESTO.md](./MANIFESTO.md)
- [RISK_POLICY.md](./RISK_POLICY.md)

---

## ⚙️ Princípios Técnicos

1. **Somente dados públicos (de verdade)**
   - Sem login, sem bypass, sem violação de termos, sem dados obtidos ilegalmente.
2. **Reprodutibilidade**
   - Resultados determinísticos e verificáveis.
3. **Explicabilidade**
   - Cada indicador tem método, evidência, limitações e hipóteses alternativas.
4. **Neutralidade**
   - O sistema aponta padrões; interpretação e contextualização são humanas.
5. **Modularidade**
   - Conectores e regras isolados, versionados e auditáveis.
6. **Offline-first / execução local como padrão**
   - Minimiza risco de censura, centralização e abuso por infraestrutura única.
7. **Rastreabilidade (proveniência)**
   - Tudo precisa ser rastreável: fonte → transformação → output.
8. **Governança aberta**
   - Mudanças relevantes passam por processo documentado (RFC/decisões).

---

## 🏗 Arquitetura (diretrizes)

A organização tende a se dividir em componentes (pode evoluir com o tempo):

- `core-engine` → motor de pipeline (ingestão, normalização, checks, indicadores)
- `connectors` → conectores para fontes oficiais (um módulo por fonte)
- `schemas` → padronização de dados, dicionários e versionamento de schema
- `risk-models` → indicadores, heurísticas e métodos estatísticos documentados
- `docs` → documentação técnica, metodológica e política (este repositório pode ser “docs”)

O design é orientado a:
- **pipelines reprodutíveis**
- **logs rastreáveis sem PII**
- **versionamento de regras/indicadores**
- **auditoria de alterações** (DECISIONS/RFC)

---

## 🛡 Mitigação de Riscos (por design)

O OpenAudit Brasil é construído com controles explícitos para reduzir:

- **risco jurídico** (difamação, imputação indevida, privacidade)
- **risco político** (captura ideológica, instrumentalização)
- **risco técnico** (falso positivo, vieses, dados incompletos)
- **risco de abuso** (doxxing, targeting, perseguição)

Medidas centrais:
- execução local como padrão
- linguagem neutra e vocabulário controlado
- outputs sempre com disclaimer + explicação
- proibição de rankings acusatórios
- gate de revisão para mudanças sensíveis

Detalhes completos:
- [RISK_POLICY.md](./RISK_POLICY.md)
- [METHODOLOGY.md](./METHODOLOGY.md)
- [GOVERNANCE.md](./GOVERNANCE.md)

---

## 🤝 Como contribuir

Fluxo recomendado:

1. Leia:
   - [MANIFESTO.md](./MANIFESTO.md)
   - [RISK_POLICY.md](./RISK_POLICY.md)
   - [METHODOLOGY.md](./METHODOLOGY.md)
2. Abra uma **issue** com a proposta (contexto, motivação, impacto)
3. Se for mudança relevante (fonte nova, indicador novo, export, busca sensível):
   - proponha uma RFC e descreva mitigação de risco
4. Submeta PR com:
   - testes (quando aplicável)
   - documentação do que mudou
   - justificativa técnica objetiva
   - atenção a privacidade e linguagem neutra

Regras de convivência:
- [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)

---

## ⚖️ Aviso Legal

Este projeto:

- utiliza exclusivamente dados públicos conforme critérios documentados
- não emite acusações, denúncias ou juízos de valor
- não substitui órgãos de controle, investigação ou justiça
- gera **indicadores analíticos** sujeitos a limitações e **falsos positivos**

Qualquer interpretação além do indicador técnico é responsabilidade do usuário, e deve considerar contexto, qualidade de dados e limitações metodológicas.

---

## 🚀 Próximos passos (roadmap inicial)

- Definir padrão de schema unificado e dicionário de dados
- Implementar pipeline mínimo viável (ingest → normalize → quality checks → indicador simples)
- Criar primeiro conector oficial com documentação e testes
- Publicar primeira release reprodutível com logs de proveniência
- Formalizar RFC template e fluxo de DECISIONS

---

## 📬 Contato e comunidade

- Site: https://openauditbrasil.com/
- GitHub: https://github.com/OpenAudit-Brasil
- Discord: https://discord.gg/KJAJPWGDBW
