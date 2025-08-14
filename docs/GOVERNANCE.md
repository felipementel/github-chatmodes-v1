# Governança Técnica & Operacional

Este documento consolida princípios, papéis, processos, métricas e templates que regem a evolução sustentável dos produtos e plataformas .NET 9 no Azure.

## 1. Objetivos
- Alinhar negócio, arquitetura, entrega, qualidade, operação e segurança.
- Reduzir risco (técnico, operacional, segurança) de forma previsível.
- Aumentar velocidade com controle (progressive delivery + métricas).
- Garantir rastreabilidade ponta a ponta (story → código → build → deploy → telemetria → aprendizado).

## 2. Princípios Norteadores
| Dimensão | Princípio | Descrição Curta |
|----------|-----------|-----------------|
| Arquitetura | Simplicidade Evolutiva | Começar mínimo, crescer orientado a métricas. |
| Qualidade | Shift-Left | Prevenção > detecção tardia. |
| Segurança | Defense in Depth | Controles em múltiplas camadas. |
| Operação | Observability by Design | Instrumentar antes de escalar. |
| Produto | Valor Mensurável | Toda entrega liga-se a métrica (leading/lagging). |
| Performance | Data-Driven | Otimizar só após evidência. |
| FinOps | Custo Visível | Custos por unidade de valor (usuário/transação). |
| Cultura | Blameless Learning | Incidentes geram melhoria, não culpa. |

## 3. Papéis & Responsabilidades Resumidas
| Papel | Foco | Responsabilidades-Chave |
|------|------|-------------------------|
| Product Owner | Valor & Prioridade | Backlog, métricas de sucesso, hipóteses, DoR/DoD. |
| Arquiteto | Estrutura & NFRs | Design, padrões, trade-offs, versionamento API, threat modeling. |
| Desenvolvedor | Implementação | Código limpo, testes, logging, pacotes, PR checklist. |
| QA | Estratégia de Testes | Pirâmide, automação, dados de teste, flakiness, cobertura por risco. |
| DevOps | Fluxo & Automação | CI/CD, IaC, supply chain, promoção controlada. |
| SRE | Confiabilidade | SLOs, error budgets, incidentes, resiliência, observabilidade. |
| Segurança | AppSec & Cloud Sec | Threat model, pipeline seguro, gestão de segredos, vulnerabilidades. |

## 4. Fluxo Macro (Discovery → Operação)
1. Ideia / Problema (PO)
2. Hipótese & Métricas (PO + Arquiteto + Segurança)
3. Modelagem & NFR / Threat Modeling (Arquiteto + Segurança + SRE)
4. Planejamento / DoR validado (Squad)
5. Implementação (Dev) + Testes (QA) + Observabilidade (SRE/Dev)
6. Build & Scans (DevOps + Segurança)
7. Deploy Progressivo (DevOps + SRE)
8. Monitoramento & Feedback (SRE + PO)
9. Aprendizado / Postmortem / ADRs (Todos)

## 5. Definition of Ready (DoR)
Uma história entra em desenvolvimento somente se contiver:
- Contexto e problema claros
- Valor mensurável (North Star / Leading / Lagging associado)
- Critérios de aceite testáveis (Gherkin se aplicável)
- Riscos / dependências mapeados
- Dados / privacidade considerados (classificação)
- Estratégia de experimentação / flag (se aplicável)
- Estimativa e fatiamento adequado (<= 3 dias de esforço ideal)

## 6. Definition of Done (DoD)
Considerada concluída quando:
- Código mergeado (PR aprovado, sem novos warnings críticos)
- Testes: unit + relevantes de integração & contrato verdes
- Cobertura crítica mantida (módulos core)
- Métricas & logs & traces implementados
- Migrations aplicadas/testadas (quando houver)
- Scans (SAST / SCA / Container / IaC) sem vulnerabilidades Critical/High não justificadas
- Feature flag configurada (se usada)
- Documentação mínima (README serviço + CHANGELOG + ADR se decisão estrutural)
- Deploy em ambiente de staging OK (gates aprovados)

## 7. Estrutura de CI/CD (Gates Padrão)
| Estágio | Objetivo | Gates / Critérios |
|---------|----------|-------------------|
| Validate | Qualidade básica | Sintaxe, formato, linters |
| Build | Artefato reproduzível | Versões fixas, SBOM gerado |
| Test | Confiabilidade | Unit ≥ baseline, integração/contrato passam |
| Security Scan | Risco reduzido | SAST, SCA, IaC, Container sem Critical/High |
| Package | Pronto para promoção | Imagem assinada, tagging semântico |
| Deploy Canary | Verificação parcial | Erro < limite, métricas estáveis |
| Deploy Full | Escalonar | Canary aprovado + SLO não violado |
| Pós-Deploy | Confirmar saúde | Dashboards / alertas operacionais ativos |

## 8. SLO / SLA / Métricas
| Tipo | Exemplo | Uso |
|------|---------|-----|
| SLO | 99.5% req < 400ms P95 | Interno (qualidade alvo) |
| SLA | 99.0% uptime mensal | Externo (compromisso) |
| Leading | % onboarding concluído | Predição de retenção |
| Lagging | Churn mensal | Resultado consolidado |
| Error Budget | 0.5% falhas permitidas | Balancear entrega vs. hardening |

## 9. Threat Modeling (Ciclo Simplificado)
1. Identificar ativos e dados sensíveis
2. Mapear fluxos e trust boundaries
3. Enumerar ameaças (STRIDE / LINDDUN conforme contexto)
4. Classificar risco (Impacto x Probabilidade)
5. Definir controles (preventivo, detectivo, mitigador)
6. Registrar no ADR / planilha e revisar a cada release maior

## 10. Padrões de Arquitetura
- Microservices orientados a Bounded Contexts
- Ports & Adapters / Hexagonal
- Outbox + Event-Driven para consistência eventual
- Resiliência: Circuit Breaker, Retry c/ jitter, Timeout, Bulkhead, Fallback
- Versionamento de API: path version + política de depreciação (≥ 90 dias)

## 11. Qualidade & Testes
| Nível | Objetivo | Ferramentas |
|-------|----------|-------------|
| Unit | Lógica isolada | xUnit + Bogus + FluentAssertions |
| Integração | Contratos internos / DB | xUnit + Testcontainers (futuro) |
| Contrato | Consumer-Driven / APIs | Pact (opcional) |
| E2E | Fluxo crítico usuário | Playwright |
| Resiliência | Falhas e degradações | Chaos / fault injection |
| Performance | Latência / throughput | k6 (conceitos) |

Cobertura guiada a risco (não perseguir 100%). Flakiness < 2% (quarentena se exceder).

## 12. Dados de Teste
- Sintéticos por padrão (Bogus)
- Mascaramento quando derivado de produção
- Isolamento por pipeline (namespace/schema)
- Limpeza idempotente pós-execução

## 13. Logging, Métricas & Tracing
| Aspecto | Diretriz |
|---------|----------|
| Logging | Estruturado c/ CorrelationId, no PII sensível sem mascarar |
| Métricas Técnicas | Latência P50/P95/P99, erros, throughput, saturação |
| Métricas Negócio | Conversão, ativação, retenção, custo/usuário |
| Tracing | OpenTelemetry ativo em todos os serviços |

## 14. Segurança
- Segredos: Key Vault / Managed Identity
- Autenticação: Azure AD / OIDC
- Autorização: Roles + Claims + Policies
- Scans: CodeQL / Dependabot / Container (Trivy) / IaC (Checkov/tfsec)
- Pipeline: Falha se Critical/High sem justificativa escrita (exceção temporária com prazo)

## 15. FinOps
| Item | Métrica |
|------|---------|
| Custo Variável | $ / transação / usuário ativo |
| Eficiência | % custo componente crítico vs. total |
| Anomalia | Desvio > 15% média móvel semanal |
| Otimização | Recursos ociosos > 30% por 7 dias → ação |

## 16. Templates
### 16.1 User Story
```
Como <persona> quero <necessidade> para <benefício mensurável>.
Critérios de Aceite:
1. ...
Métricas:
- Leading: ...
- Lagging: ...
Riscos: ...
Dependências: ...
Experimento / Flag: ...
``` 

### 16.2 ADR (Architecture Decision Record)
```
# ADR <número>: <Título>
Data: AAAA-MM-DD
Status: Proposed | Accepted | Superseded
Contexto:
Decisão:
Alternativas Consideradas:
Consequências (positivas/negativas):
Riscos / Mitigações:
Relação com SLO / Métricas de Produto:
``` 

### 16.3 Postmortem (Blameless)
```
Incidente: <ID / Data>
Severidade: SEV1/2/3
Resumo Executivo:
Linha do Tempo:
Impacto:
Detecção (MTTD):
Resposta (MTTR):
Causa Raiz:
Contribuintes Secundários:
O que Funcionou:
O que Não Funcionou:
Ações Corretivas (Owner / Prazo / Tipo):
Lições Aprendidas:
Prevenção Futuras:
``` 

### 16.4 SLO Definition
```
Serviço: <nome>
SLI: (ex: Latência P95 / Disponibilidade)
Fonte de Dados: (ex: App Insights Query)
Janela: 30 dias (rolling)
SLO: 99.5%
Error Budget: 0.5%
Ação ao Consumir 50%: Revisar prioridades performance
Ação ao Consumir 100%: Congelar features não críticas
Correlação com Métrica de Produto: Abandono de checkout
``` 

### 16.5 Checklist de Pull Request
- [ ] Story vinculada (ID)
- [ ] Testes (unit / integração) adicionados ou atualizados
- [ ] Cobertura crítica preservada
- [ ] Logs e métricas novos quando necessário
- [ ] Trata erros e valida entradas
- [ ] Sem dependências não utilizadas
- [ ] Sem warnings novos
- [ ] Migrações revisadas (se houver)
- [ ] Segurança básica (input validation / secrets / authz)

## 17. Versionamento de APIs
- Sem breaking sem major version.
- Novos campos opcionais = compatível.
- Comunicar depreciação com antecedência (≥ 90 dias).
- Testes de contrato para garantir compatibilidade.

## 18. Gestão de Dependências
- Versões sem wildcards.
- Auditoria mensal automática (relatório de CVEs pendentes).
- SBOM obrigatório para cada release.

## 19. Gestão de Flakiness
- Taxa > 2%: mover teste para quarentena e abrir issue.
- Critério de saída: 10 execuções consecutivas estáveis.

## 20. Riscos & Mitigações (Resumo)
| Risco | Mitigação |
|-------|-----------|
| Divergência de padrões | Este documento + revisão periódica |
| Falta de visibilidade de custo | Dashboards FinOps + tags obrigatórias |
| Incidentes recorrentes | Postmortem + automação de ações |
| Técn. Debt oculta | Trilha de débito priorizada no backlog |
| Segurança negligenciada | Gates de pipeline + threat modeling inicial |

## 21. Governança de Atualização
- Revisão trimestral deste documento.
- Atualizações significativas → ADR referenciando alteração.
- Owners rotativos (trazer visão de cada papel).

## 22. Métricas de Saúde de Processo
| Métrica | Alvo Inicial |
|---------|--------------|
| Lead Time Story | ≤ 7 dias corridos |
| Change Failure Rate | < 15% |
| MTTR Prod | < 60 min |
| Cobertura Crítica | ≥ baseline acordada |
| Erro P95 Checkout | < 400ms |
| Custo / Usuário Ativo | Estável (< +/-10% trimestre) |

## 23. Adoção & Onboarding
- Novo membro: ler este documento + principais ADRs + últimas 5 release notes.
- Checklist onboarding inclui: acesso, ambientes, telemetria, práticas de segurança.

## 24. Escalada / Decisões Rápidas
- Divergência técnica imediata: reunião curta (≤ 15 min) com Arquiteto + Dev + SRE.
- Empate → fallback para princípios (simplicidade, métricas, risco).

## 25. Contínua Evolução
Este documento não é estático; deve refletir práticas reais. Métricas de melhoria contínua alimentam refinamentos.

---
Última atualização: <atualizar ao modificar>
