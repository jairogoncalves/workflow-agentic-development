# Agentes Claude Code — Pipeline de Desenvolvimento

Conjunto de prompts/subagentes em Markdown para uso com **Claude Code**, cobrindo o ciclo completo de desenvolvimento de software: discovery → produto → UX → UI → frontend/mobile → backend → dados → experimentação → code review → segurança → QA → DevOps → release → incidente → finops → docs, com um orquestrador que coordena tudo.

Todos os prompts seguem as [Prompting Best Practices da Anthropic](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices): role explícito, instruções diretas, XML tags estruturadas, output format definido, regras de qualidade declaradas, e uso das diretrizes específicas para Claude Opus 4.7 (`investigate_before_answering`, `default_to_action`, `balancing_autonomy_and_safety`, `controlling_subagent_spawning`).

## Como instalar no Claude Code

Os subagentes do Claude Code vivem em uma destas pastas:

- **Por projeto** (recomendado para times): `.claude/agents/` na raiz do repositório
- **Global no usuário**: `~/.claude/agents/`

Copie os arquivos `.md` para a pasta escolhida:

```bash
mkdir -p .claude/agents
cp *.md .claude/agents/
```

A partir daí, o Claude Code lê o frontmatter (`name`, `description`, `model`) e o conteúdo do prompt automaticamente quando aciona o subagente.

## Estrutura recomendada do repositório

Os agentes assumem (e produzem) esta estrutura:

```
.
├── .claude/agents/              # estes arquivos
├── docs/
│   ├── product/
│   │   ├── discovery/           # estudos de discovery (product-discovery)
│   │   └── ...                  # épicos e histórias (product-manager)
│   ├── ux/                      # UX specs (ux-designer)
│   ├── ui/                      # UI specs + design-system.md (ui-designer)
│   ├── api/                     # contratos de API
│   ├── reviews/                 # relatórios de code review (code-reviewer)
│   ├── incidents/               # postmortems (incident-responder)
│   ├── releases/                # documentos de release (release-manager)
│   └── migrations/              # guias de migração entre majors (release-manager)
├── web/                         # frontend (frontend-engineer)
├── mobile/                      # app React Native iOS+Android (mobile-engineer)
├── services/<servico>/          # backends (backend-engineer)
├── data/                        # pipelines analíticos: dbt, DAGs, schemas de evento (data-engineer)
├── experiments/                 # specs e relatórios de A/B (growth-engineer)
├── finops/                      # políticas, relatórios e otimizações de custo (finops)
├── deploy/                      # k8s + kustomize (devops-engineer)
├── security/
│   ├── scans/                   # JSONs do Trivy
│   └── reports/                 # relatórios (dev-security)
├── e2e/                         # testes Cypress + Gherkin para web (qa-automation)
├── e2e-mobile/                  # testes Appium + Gherkin para iOS/Android (qa-automation)
└── docs-site/                   # Docusaurus consolidando tudo (docs-writer)
```

## Os agentes

| Agente | Quando usar | Modelo sugerido |
|---|---|---|
| `orchestrator` | Ponto de entrada para qualquer demanda multidisciplinar | opus |
| `product-discovery` | Investigar problema/oportunidade vaga e entregar estudo de discovery ao PO antes de virar épico | sonnet |
| `product-manager` | Transformar demanda já validada em épicos/histórias com critérios de aceite | sonnet |
| `ux-designer` | Fluxos, jornadas, wireframes textuais, acessibilidade | sonnet |
| `ui-designer` | Design tokens, mapeamento shadcn/ui, telas em alta fidelidade | sonnet |
| `frontend-engineer` | Implementação em React + Vite + shadcn + Storybook (web) | sonnet |
| `mobile-engineer` | Implementação em React Native + TypeScript (iOS + Android, Expo/EAS) | sonnet |
| `backend-engineer` | FastAPI + Postgres/Mongo + Redis + RabbitMQ + MinIO + Prometheus + Docker (Poetry) | sonnet |
| `data-engineer` | Pipelines analíticos (dbt, Airflow), contratos de evento, modelagem dimensional, glossário de métricas | opus |
| `growth-engineer` | Experimentos A/B com rigor estatístico (spec pré-registrada, amostra/MDE, guardrails, análise) | opus |
| `finops` | Atribuição, otimização e governança de custo de cloud + SaaS de engenharia | opus |
| `code-reviewer` | Revisão de PR/MR (correctness, design, testes, contratos, observabilidade) antes do security/deploy | sonnet |
| `devops-engineer` | Manifests Kubernetes + GitLab CI + provisionamento | sonnet |
| `dev-security` | Trivy + análise de pentest, relatórios acionáveis | sonnet |
| `qa-automation` | Testes E2E em Gherkin + Cypress (web) / Appium (mobile iOS/Android) | sonnet |
| `release-manager` | Versão (SemVer), changelog, release notes, plano de rollout/rollback, comunicação | sonnet |
| `incident-responder` | SRE de plantão — diagnóstico e mitigação em produção, postmortem blameless | opus |
| `docs-writer` | Documentação consolidada em Docusaurus, legível por todos os perfis | sonnet |

> Você pode trocar o modelo no frontmatter (`model:`) conforme custo/latência. Para o orquestrador, `opus` é recomendado por causa do raciocínio sobre dependências entre agentes. 

## Como usar no dia a dia

**Caso 1 — Demanda nova:** chame o orquestrador.

> "Preciso adicionar um fluxo de recuperação de senha por email no nosso produto."

O orquestrador apresenta o plano (quais agentes, em que ordem) e executa após sua confirmação.

**Caso 2 — Tarefa pontual:** chame o agente direto.

> "Use o frontend-engineer para criar um componente `DataTable` com paginação e ordenação."

**Caso 3 — Auditoria:** chame `dev-security` em uma branch antes de release.

> "Rode o dev-security contra a imagem `registry.gitlab.com/x/y:abc123` e o código em `services/auth/`."

**Caso 4 — Demanda nebulosa:** chame `product-discovery` antes de quebrar em histórias.

> "Estamos vendo queda de 18% na conversão do checkout no último mês. Rode discovery e me traga uma recomendação antes do PM começar."

**Caso 5 — Revisar PR:** chame `code-reviewer` na branch.

> "Use o code-reviewer no PR #482 e me traga o relatório com bloqueadores antes do merge."

**Caso 6 — Incidente em produção:** chame `incident-responder`.

> "Alerta `api_5xx_rate` acima de 5% há 3 minutos em `auth-api` prod — diagnostique e me proponha mitigação."

**Caso 7 — Pipeline de dados / nova métrica:** chame `data-engineer`.

> "Precisamos rastrear `checkout_completed` e ter um mart com receita diária por segmento. Desenhe o contrato de evento e o modelo dbt."

**Caso 8 — Preparar release:** chame `release-manager` quando o trabalho está pronto.

> "Esses são os PRs mergeados desde a v1.3.2. Decida a versão, gere changelog/release notes e me traga o plano de rollout."

**Caso 9 — Feature mobile:** chame `mobile-engineer` (ou orquestrador se precisar de UI/backend juntos).

> "Implemente a tela de recuperação de senha em React Native (iOS+Android) usando a UI Spec em `docs/ui/password-reset.md`."

**Caso 10 — Validar hipótese com experimento:** chame `growth-engineer`.

> "Queremos validar se trocar o CTA do checkout de azul para verde aumenta conversão em pelo menos 2%. Desenhe a spec do A/B com amostra calculada e guardrails."

**Caso 11 — Conta da cloud subiu sem explicação:** chame `finops`.

> "Conta AWS subiu 22% em maio comparado a abril sem aumento óbvio de tráfego. Investigue e me traga as oportunidades de otimização priorizadas por ROI."

## Princípios aplicados em todos os prompts

- **Role claro logo no topo** — Claude responde melhor sabendo exatamente quem ele é nesse contexto.
- **XML tags semânticas** (`<role>`, `<processo>`, `<output_format>`, `<regras_de_qualidade>`) — facilita o parsing e reduz ambiguidade.
- **Output format explícito** — cada agente sabe exatamente o que entregar e onde.
- **`investigate_before_answering`** — força leitura do contexto real do repositório antes de propor mudanças. Reduz alucinação.
- **`default_to_action` vs `do_not_act_before_instructions`** — calibrado por agente conforme o nível de risco da ação.
- **Critérios de qualidade declarados** — cada agente conhece suas próprias regras de "pronto".
- **Handoffs explícitos** — o que sai de um agente é o que entra no próximo, com caminhos de arquivo concretos.

## Customização

Cada arquivo é um Markdown editável. Ajustes recomendados ao adotar:

1. **Stack**: se seu time usa Vue em vez de React, Helm em vez de Kustomize, GitHub Actions em vez de GitLab CI — edite o bloco `<stack_obrigatorio>` correspondente.
2. **Paths**: ajuste `docs/`, `services/`, `deploy/` conforme a convenção do seu repositório.
3. **Modelo**: troque `model: sonnet` por `model: haiku` em agentes leves para reduzir custo, ou para `model: opus` em agentes que exigem mais raciocínio.
4. **Idioma**: os agentes estão em PT-BR; troque conforme necessidade do time.

## Referências

- [Claude prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)
- [Claude Code subagents documentation](https://docs.claude.com/en/docs/claude-code/sub-agents)
