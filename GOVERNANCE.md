# GOVERNANÇA

## OpenAudit Brasil — Governança do Projeto (v1.0.0) 27-02-2026

> Este documento define **como o projeto é conduzido**, **quem decide o quê** e **quais controles existem** para manter o OpenAudit Brasil **técnico, neutro, auditável e juridicamente prudente**.


<img width="1431" height="507" alt="image" src="/assets/logo-full.png" />


---

## 1. Escopo e objetivos da governança

A governança do OpenAudit Brasil existe para:

1. Proteger a **integridade técnica** do código e da metodologia.
2. Garantir **neutralidade política** e evitar captura por grupos/atores.
3. Reduzir risco de **difamação**, **perseguição**, **uso indevido** e **violação de termos/leis**.
4. Manter o projeto **auditável**, **reprodutível** e **bem documentado**.
5. Garantir um processo previsível para contribuições, releases e decisões.

---

## 2. Documentos normativos do projeto

A governança é interpretada em conjunto com:

- `MANIFESTO.md` (finalidade, limites e princípios)
- `CODE_OF_CONDUCT.md` (conduta e aplicação)
- `CONTRIBUTING.md` (processo de contribuição)
- `SECURITY.md` (vulnerabilidades e disclosure)
- `RISK_POLICY.md` (política de risco: dados, outputs e uso indevido)
- `METHODOLOGY.md` (metodologia, limitações, falso-positivo)
- `DECISIONS.md` (registro de decisões técnicas/risco)

Em caso de conflito entre documentos, a prioridade é:
`MANIFESTO.md` > `RISK_POLICY.md` > `METHODOLOGY.md` > `GOVERNANCE.md` > demais.

---

## 3. Princípios operacionais

1. **Sem acusações**: o projeto não produz e não aceita contribuições que caracterizem acusação, imputação de crime ou juízo moral.
2. **Neutro e apartidário**: não há apoio a partidos, candidaturas, movimentos ou campanhas.
3. **Transparência metodológica**: análise sem explicabilidade/documentação não entra.
4. **Rastreabilidade**: mudanças relevantes devem ser reprodutíveis e justificadas.
5. **Segurança e privacidade**: dados pessoais exigem cuidado adicional e, quando possível, minimização/anonimização.
6. **Previsibilidade**: decisões seguem processo; exceções são documentadas.

---

## 4. Papéis e responsabilidades

### 4.1 Maintainers (Mantenedores)
**Responsáveis finais por:**
- Arquitetura, qualidade, releases e direção técnica.
- Aprovação/recusa de PRs.
- Proteção contra degradação de qualidade, risco jurídico e reputacional.

**Poderes:**
- Aprovar merges (conforme regras de revisão).
- Definir roadmap e milestones.
- Conceder/revogar permissões de repositório conforme este documento.

**Deveres:**
- Justificar decisões relevantes de forma objetiva.
- Registrar mudanças significativas em `DECISIONS.md`.
- Aplicar as políticas de risco e conduta sem favorecimento.

---

### 4.2 Risk Reviewers (Triagem de Risco)
Grupo responsável por **avaliar riscos não-técnicos** antes de decisões sensíveis.

**Escopo de risco:**
- LGPD e privacidade
- Difamação/ataque à honra (direto ou indireto)
- Termos de uso / scraping / licenças de dados
- Vetores de perseguição e doxxing
- Risco político/reputacional por design ou comunicação

**Poderes:**
- Exigir mitigação e documentação antes de merge/release.
- Solicitar alterações de linguagem em UI/docs/outputs para reduzir risco.
- Marcar PR/issue como "Risk Review Required".

**Limite:**
- Risk Reviewers **não** definem arquitetura sozinhos; eles atuam como gate de risco e conformidade.

---

### 4.3 Reviewers
Contribuidores com histórico que auxiliam na revisão técnica (código, testes, docs).  
Não têm poder final de merge (a menos que também sejam Maintainers).

---

### 4.4 Contributors
Qualquer pessoa que abre issues, propõe PRs, revisa, testa, documenta ou sugere melhorias.

---

### 4.5 Community Members
Participantes em discussões, suporte, ideação e validação.  
Devem seguir `CODE_OF_CONDUCT.md`.

---

## 5. Regras de acesso e controle de merge

### 5.1 Branch protection (recomendado e esperado)
- Branch **main** protegida.
- Merge apenas via PR.
- CI obrigatório (lint/test/build).
- Revisão obrigatória:
  - **2 aprovações** (mínimo) para mudanças relevantes.
  - **1 Maintainer** obrigatório para qualquer merge.

### 5.2 CODEOWNERS
- Diretórios sensíveis devem ter `CODEOWNERS` (ex.: `connectors/`, `analysis/`, `docs/methodology/`).

### 5.3 Assinatura de contribuição (DCO recomendado)
Para reforçar responsabilidade autoral, o projeto pode exigir **DCO**:
- PRs com "Signed-off-by" (Developer Certificate of Origin) ou bot equivalente.

---

## 6. Processo de contribuição e decisão

### 6.1 Issues
Toda mudança começa como issue (exceto correções triviais).  
Issues devem ser classificadas com labels (mínimo recomendado):

- `type:bug`, `type:feature`, `type:docs`, `type:security`
- `area:connectors`, `area:analysis`, `area:ui`, `area:docs`
- `risk:low`, `risk:medium`, `risk:high`
- `status:needs-info`, `status:ready`, `status:blocked`
- `governance:rfc-required`
- `risk-review:required`

### 6.2 RFC (obrigatório para mudanças significativas)
Mudanças significativas exigem RFC (Request for Comments) quando envolvem:

- Novo conector/fonte de dados
- Novo método analítico/score/heurística
- Qualquer output que possa ser interpretado como acusatório
- Armazenamento/distribuição de dataset
- Mudanças em política de risco, metodologia ou manifesto
- Funcionalidades relacionadas a pessoas/políticos/eleições (mesmo que dados sejam públicos)

**Formato mínimo de RFC:**
1. Problema
2. Proposta
3. Impactos técnicos
4. Impactos metodológicos (falso positivo, limitações)
5. Risco (LGPD, difamação, uso indevido, ToS)
6. Mitigações
7. Plano de testes e validação
8. Plano de rollout e reversão

### 6.3 Critérios objetivos para merge
Nenhum PR entra sem:
- Testes (ou justificativa explícita de por que não se aplica)
- Documentação mínima (quando muda comportamento)
- Explicabilidade (quando muda análise)
- Changelog/decisão (quando relevante)
- Linguagem neutra e não acusatória (UI, docs, outputs)

---

## 7. Gate de risco (obrigatório)

### 7.1 Quando o Risk Review é obrigatório
`risk-review:required` deve ser aplicado quando o PR/feature:

1. Introduz ou altera coleta/uso de **dados pessoais** (mesmo públicos).
2. Toca em **identificadores** (CPF, CNPJ, endereço, telefone, e-mail, nome completo etc.).
3. Produz outputs com risco de **interpretação acusatória** (rankings, labels, "suspeito", "fraude", etc.).
4. Facilita **doxxing**, perseguição, assédio ou targeting.
5. Depende de scraping em contexto com **ToS restritivos** ou autenticação.
6. Publica datasets derivados que possam aumentar risco de uso indevido.
7. Aborda temas de alta sensibilidade pública (ex.: eleições, agentes públicos, figuras políticas).

### 7.2 Regras de aprovação sob risco
- PRs com `risk-review:required` exigem:
  - **1 aprovação de Risk Reviewer**
  - **1 aprovação de Maintainer**
- Maintainers **não devem** ignorar recomendação de risco.  
  Exceções exigem registro em `DECISIONS.md` com justificativa e mitigação.

---

## 8. Política de comunicação pública

Para reduzir risco político e jurídico:

1. Linguagem sempre técnica: "padrão atípico", "inconsistência", "divergência".
2. Proibido conteúdo de campanha, ataque, ironia direcionada e acusações.
3. Publicações oficiais do projeto devem:
   - Referenciar metodologia
   - Indicar limitações
   - Evitar personalização em indivíduos
4. O projeto **não endossa** interpretações, narrativas ou conclusões externas.

---

## 9. Conflito de interesse e captura

### 9.1 Declaração de conflito
Maintainers e Risk Reviewers devem declarar (em issue privada ou registro interno) conflitos relevantes, como:
- Filiação/atuação formal em campanha/partido em tema diretamente afetado por feature.
- Contratos que dependam do resultado/uso do projeto.
- Participação em iniciativas concorrentes onde possa haver captura.

### 9.2 Mitigação
- Em caso de conflito direto, a pessoa deve se abster da decisão.
- Persistindo risco de captura, a decisão pode exigir quórum maior (ver seção 10).

---

## 10. Tomada de decisão e quórum

### 10.1 Decisões rotineiras
- "Lazy consensus": se não houver objeção fundamentada, segue.
- Objeções devem ser técnicas/metodológicas/risco, não ideológicas.

### 10.2 Decisões significativas (RFC)
Requer:
- RFC aberto por período mínimo (ex.: 7 dias) ou até estabilização do debate.
- Aprovação por **maioria simples** dos Maintainers.
- Se `risk:high`, requer **2/3** dos Maintainers + aprovação de Risk Review.

### 10.3 Empate
Em empate, prevalece a opção **mais conservadora** do ponto de vista de risco e reversibilidade.

---

## 11. Releases e versionamento

- Releases seguem versionamento semântico (SemVer) quando aplicável.
- Toda release deve ter:
  - Notas de release (changelog)
  - Lista de mudanças com impacto metodológico/risco
  - Referências a RFCs e decisões relevantes

Mudanças que alterem comportamento analítico devem atualizar `METHODOLOGY.md`.

---

## 12. Segurança

Vulnerabilidades devem seguir `SECURITY.md`.  
Em caso de incidente:
- Patch prioritário (hotfix)
- Post-mortem público quando apropriado, preservando dados sensíveis.

---

## 13. Resolução de conflitos

Processo escalonado:

1. Tentativa de resolução no PR/issue com foco técnico.
2. Mediação por Maintainer não envolvido.
3. Se envolver risco/conduta, acionar Risk Reviewer e/ou aplicação do `CODE_OF_CONDUCT.md`.
4. Decisão final do Conselho de Maintainers.

Ataques pessoais, acusações e politicagem encerram discussão e podem resultar em sanções.

---

## 14. Entrada e saída de papéis

### 14.1 Como se tornar Reviewer
Critérios mínimos:
- Histórico consistente de contribuições de qualidade.
- Boa conduta e comunicação técnica.
- Familiaridade com padrões e testes.

Indicação por Maintainer + aprovação simples.

### 14.2 Como se tornar Maintainer
Critérios mínimos:
- Contribuição sustentada (código + docs + revisão).
- Histórico de decisões técnicas prudentes.
- Alinhamento explícito ao `MANIFESTO.md` e `RISK_POLICY.md`.
- Capacidade de dizer "não" com justificativa técnica.

Processo:
- Nomeação por 1 Maintainer
- Período de observação (recomendado)
- Aprovação por maioria simples dos Maintainers

### 14.3 Como se tornar Risk Reviewer
Critérios mínimos:
- Experiência ou domínio demonstrável em privacidade, compliance, OSINT responsável, governança ou análise de risco.
- Histórico de decisões conservadoras e justificadas.
- Compromisso com neutralidade política.

Processo:
- Nomeação por 1 Maintainer + 1 Risk Reviewer
- Aprovação por maioria simples dos Maintainers + maioria simples dos Risk Reviewers

### 14.4 Remoção / perda de papel
Pode ocorrer por:
- Violação grave do `CODE_OF_CONDUCT.md`
- Tentativa de instrumentalização política do projeto
- Aprovação consciente de mudança que viole manifesto/política de risco
- Inatividade prolongada (ex.: 6 meses sem participação relevante)
- Quebra de confiança (ex.: vazamento de discussões sensíveis)

Remoção exige maioria simples dos Maintainers (ou 2/3 em caso controverso).

---

## 15. Mudanças nesta governança

Alterações no `GOVERNANCE.md` exigem:
- RFC
- Aprovação por **2/3** dos Maintainers
- Revisão de risco quando aplicável

---

## 16. Cláusula de interpretação conservadora

Quando houver ambiguidade, decisões e textos devem optar pela interpretação:

1. Mais neutra politicamente
2. Menos acusatória
3. Mais conservadora juridicamente
4. Mais reversível tecnicamente

---

## 17. Compromisso final

A governança do OpenAudit Brasil existe para manter o projeto:
- **aberto**, **auditável** e **técnico**,
- com **limites éticos explícitos**,
- e com **controle real de risco**.

Sem isso, o projeto se torna vulnerável a abuso, captura e litígio.

---
