# SEGURANÇA

<img width="1431" height="507" alt="image" src="/assets/logo-full.png" />


## OpenAudit Brasil — Política de Segurança (v1.0.0) 27-02-2026

> Este documento descreve como reportar vulnerabilidades e como o OpenAudit Brasil lida com incidentes de segurança.
>
> **Não use issues públicas para relatar vulnerabilidades.** Isso pode colocar usuários e dados em risco.

---

## 1) Escopo

Esta política cobre:

- Código-fonte, dependências e pipeline de build
- CLI, API, UI e integrações
- Conectores de coleta (ETL) e parsers
- Configuração de deploy (quando houver)
- Qualquer mecanismo de armazenamento, exportação e compartilhamento de resultados

---

## 2) Como reportar uma vulnerabilidade

### 2.1 Canal de reporte (preferencial)
Use um canal **privado**.

Opções recomendadas (o projeto deve manter pelo menos uma delas ativa):

1. **GitHub Security Advisories** (Security tab → "Report a vulnerability")
2. **E-mail** (`comunidade@openauditbrasil.com`)
3. **Formulário privado**

Se nenhum canal privado estiver configurado, **não publique** detalhes em issue/PR.  
Entre em contato com um Maintainer pelos canais oficiais e solicite um meio privado.

### 2.2 O que incluir no reporte
- Descrição clara do problema
- Passos para reproduzir (PoC minimal)
- Impacto (o que um atacante ganha)
- Versões/commit afetados
- Ambiente (OS, Node/Rust/Python versões, config relevante)
- Logs/prints **sem dados pessoais**
- Sugestão de mitigação (se tiver)

### 2.3 O que NÃO incluir
- Dados pessoais (CPF, endereço, telefone, e-mail, etc.)
- Tokens, chaves, segredos, cookies
- Dumps completos de bancos/logs
- Exploit fully weaponized publicado em aberto

---

## 3) SLA e fluxo de resposta (padrão do projeto)

> Estes prazos são objetivos operacionais, não garantias. Em caso de risco crítico, o projeto prioriza correção imediata.

1. **Confirmação de recebimento:** até 72 horas
2. **Triagem inicial:** até 7 dias
3. **Correção / mitigação:** conforme severidade e complexidade
4. **Divulgação coordenada:** após correção disponível

Se o reporte envolver risco de privacidade/difamação por exposição indevida de dados, a resposta pode ser acelerada.

---

## 4) Severidade e priorização

A severidade considera:

- Facilidade de exploração
- Impacto em confidencialidade, integridade e disponibilidade
- Possibilidade de vazamento de dados pessoais
- Possibilidade de uso indevido (doxxing, targeting, difamação)

### 4.1 Classificação recomendada
- **Crítica:** execução remota, bypass de autenticação, exfiltração de dados sensíveis, publicação involuntária de dados pessoais
- **Alta:** SSRF, RCE local relevante, SQLi/NoSQLi, leitura de arquivos, escalonamento de privilégios
- **Média:** XSS refletido em áreas críticas, CSRF com impacto relevante, leaks parciais
- **Baixa:** problemas com impacto limitado, hardening, configurações fracas sem exploração prática clara

---

## 5) Divulgação responsável (Coordinated Disclosure)

O OpenAudit Brasil segue divulgação coordenada:

- O reportante deve dar tempo razoável para correção antes de divulgação pública.
- O projeto se compromete a:
  - manter comunicação objetiva
  - creditar o reportante (se desejado)
  - publicar correção e notas de release quando apropriado

O projeto pode solicitar embargo temporário para reduzir risco de exploração.

---

## 6) Boas práticas de segurança exigidas no repositório

### 6.1 Segredos
- Proibido commitar segredos.
- Usar `.env.example` / `.env.tpl` sem valores reais.
- CI deve falhar em detecção de secrets (recomendado).

### 6.2 Dependências
- Atualizações de dependências com changelog e avaliação de risco.
- Uso de lockfile.
- SCA (Dependabot, osv-scanner, npm audit, cargo audit etc.) recomendado.

### 6.3 Logs e observabilidade
- Logs **não** devem conter dados pessoais, tokens ou payloads sensíveis.
- Mascaramento/remoção de PII é obrigatório quando aplicável.

### 6.4 Inputs e parsing
- Validar e sanitizar entrada.
- Parsers devem assumir dados malformados e hostis.
- Limitar tamanho de payload e profundidade de parsing.

### 6.5 Exportações e compartilhamento
- Export por padrão deve ser agregado e com disclaimers (ver `RISK_POLICY.md`).
- Links compartilháveis (se existirem) devem expirar e não expor PII.

---

## 7) Incidentes de segurança e resposta

Um incidente inclui, mas não se limita a:

- Vazamento de segredos
- Exposição de dados pessoais em outputs/logs
- Vulnerabilidade explorável em produção (se houver deploy)
- Modificação maliciosa em pipeline/artefatos de release

### 7.1 Ações imediatas (playbook mínimo)
1. Conter: desabilitar feature/rota/conector, revogar tokens/segredos, bloquear release
2. Mitigar: hotfix, revert, patch, regras de firewall/rate-limit
3. Comunicar: nota técnica objetiva (sem amplificar dados sensíveis)
4. Recuperar: nova release com correção
5. Aprender: post-mortem com medidas preventivas

### 7.2 Post-mortem
Quando apropriado, o projeto publica:
- causa raiz
- impacto
- timeline
- correções e prevenção

Sem incluir dados pessoais ou detalhes que facilitem exploração.

---

## 8) Política de "Security Fixes" e PRs

- Correções de segurança devem ter testes quando possível.
- PRs sensíveis podem ser feitos em branch privada até correção.
- Mudanças com impacto em dados pessoais exigem `risk-review:required` (ver `RISK_POLICY.md`).

---

## 9) Safe Harbor (boa-fé)

O OpenAudit Brasil encoraja pesquisa de segurança de boa-fé.

- Não perseguimos legalmente reportantes que atuem de boa-fé, sem exfiltrar dados pessoais desnecessários, sem interromper serviços e sem divulgar antes da correção.
- Pesquisa deve evitar impacto em sistemas de terceiros e respeitar termos e leis aplicáveis.

---

## 10) Créditos

O projeto pode creditar reportantes nas notas de release ou advisory, mediante consentimento.

---

## 11) Atualizações deste documento

Mudanças no `SECURITY.md` seguem processo de governança (`GOVERNANCE.md`).  
Mudanças que afetem privacidade/outputs devem considerar `RISK_POLICY.md`.

---
