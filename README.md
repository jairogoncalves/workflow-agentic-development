# Agentes Claude Code — Pipeline de Desenvolvimento

Conjunto de prompts/subagentes em Markdown para uso com **Claude Code**, cobrindo o ciclo completo de desenvolvimento de software: produto → UX → UI → frontend → backend → DevOps → segurança → QA, com um orquestrador que coordena tudo.

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
│   ├── product/                 # épicos e histórias (product-manager)
│   ├── ux/                      # UX specs (ux-designer)
│   ├── ui/                      # UI specs + design-system.md (ui-designer)
│   └── api/                     # contratos de API
├── web/                         # frontend (frontend-engineer)
├── services/<servico>/          # backends (backend-engineer)
├── deploy/                      # k8s + kustomize (devops-engineer)
├── security/
│   ├── scans/                   # JSONs do Trivy
│   └── reports/                 # relatórios (dev-security)
├── e2e/                         # testes Cypress + Gherkin (qa-automation)
└── docs-site/                   # Docusaurus consolidando tudo (docs-writer)
```

## Os agentes

| Agente | Quando usar | Modelo sugerido |
|---|---|---|
| `orchestrator` | Ponto de entrada para qualquer demanda multidisciplinar | opus |
| `product-manager` | Transformar demanda bruta em épicos/histórias com critérios de aceite | sonnet |
| `ux-designer` | Fluxos, jornadas, wireframes textuais, acessibilidade | sonnet |
| `ui-designer` | Design tokens, mapeamento shadcn/ui, telas em alta fidelidade | sonnet |
| `frontend-engineer` | Implementação em React + Vite + shadcn + Storybook | sonnet |
| `backend-engineer` | FastAPI + Postgres/Mongo + Redis + RabbitMQ + MinIO + Prometheus + Docker | sonnet |
| `devops-engineer` | Manifests Kubernetes + GitLab CI + provisionamento | sonnet |
| `dev-security` | Trivy + análise de pentest, relatórios acionáveis | sonnet |
| `qa-automation` | Testes E2E em Gherkin + Cypress | sonnet |
| `docs-writer` | Documentação consolidada em Docusaurus, legível por todos os perfis | sonnet |

> Você pode trocar o modelo no frontmatter (`model:`) conforme custo/latência. Para o orquestrador, `opus` é recomendado por causa do raciocínio sobre dependências entre agentes. Para os especialistas, `sonnet` costuma ser ideal — bom equilíbrio entre capacidade e custo.

## Como usar no dia a dia

**Caso 1 — Demanda nova:** chame o orquestrador.

> "Preciso adicionar um fluxo de recuperação de senha por email no nosso produto."

O orquestrador apresenta o plano (quais agentes, em que ordem) e executa após sua confirmação.

**Caso 2 — Tarefa pontual:** chame o agente direto.

> "Use o frontend-engineer para criar um componente `DataTable` com paginação e ordenação."

**Caso 3 — Auditoria:** chame `dev-security` em uma branch antes de release.

> "Rode o dev-security contra a imagem `registry.gitlab.com/x/y:abc123` e o código em `services/auth/`."

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
