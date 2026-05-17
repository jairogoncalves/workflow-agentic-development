# Agentes Claude Code — Pipeline de Desenvolvimento

Conjunto de prompts/subagentes em Markdown para uso com **Claude Code**, cobrindo o ciclo completo de desenvolvimento de software: discovery → produto → UX → UI → frontend/mobile → backend → dados → experimentação → code review → segurança → QA → DevOps → release → incidente → finops → docs, com um orquestrador que coordena tudo.

Inclui também um conjunto de **skills** reutilizáveis que abstraem operações repetitivas (mexer no Figma, versionar no git), evitando duplicação entre agentes.

Todos os prompts seguem as [Prompting Best Practices da Anthropic](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices): role explícito, instruções diretas, XML tags estruturadas, output format definido, regras de qualidade declaradas, e uso das diretrizes específicas para Claude Opus 4.7 (`investigate_before_answering`, `default_to_action`, `balancing_autonomy_and_safety`, `controlling_subagent_spawning`).

## Como instalar no Claude Code

Os subagentes e skills do Claude Code vivem em uma destas pastas:

- **Por projeto** (recomendado para times): `.claude/agents/` e `.claude/skills/` na raiz do repositório
- **Global no usuário**: `~/.claude/agents/` e `~/.claude/skills/`

Copie os arquivos para as pastas escolhidas:

```bash
mkdir -p .claude/agents .claude/skills
cp agents/*.md .claude/agents/
cp -r skills/* .claude/skills/
```

A partir daí, o Claude Code lê o frontmatter (`name`, `description`, `model`) e o conteúdo do prompt automaticamente quando aciona o subagente ou carrega uma skill.

> **Formato das skills:** cada skill é uma pasta `skills/<nome>/SKILL.md` (não um `.md` solto). O Claude Code só descobre a skill quando o arquivo se chama exatamente `SKILL.md` dentro da pasta. Recursos auxiliares da skill (templates, exemplos, scripts) ficam em arquivos irmãos dentro da mesma pasta.

## Estrutura recomendada do repositório

Os agentes assumem (e produzem) esta estrutura:

```
.
├── .claude/
│   ├── agents/                  # subagentes (papéis com contexto isolado)
│   └── skills/                  # skills (procedimentos reutilizáveis)
│       ├── figjam-wireframer/
│       ├── figma-builder/
│       ├── figma-reader/
│       └── git-workflow/
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
| `product-discovery` | Investigar problema/oportunidade vaga e entregar estudo de discovery ao PO antes de virar épico | opus |
| `product-manager` | Transformar demanda já validada em épicos/histórias com critérios de aceite | sonnet |
| `ux-designer` | Fluxos, jornadas e wireframes low-fi no FigJam (usa `figjam-wireframer`) | sonnet |
| `ui-designer` | Specs de UI + telas em alta fidelidade no Figma Design (usa `figma-builder` e `figma-reader`) | sonnet |
| `frontend-engineer` | Implementação em React + Vite + shadcn/ui + Tailwind + Storybook (web). Lê Figma via `figma-reader` | opus |
| `mobile-engineer` | React Native + NativeWind (iOS + Android, Expo/EAS). Lê Figma via `figma-reader` | sonnet |
| `backend-engineer` | FastAPI + Postgres/Mongo + Redis + RabbitMQ + MinIO + Prometheus + Docker (Poetry) | opus |
| `data-engineer` | Pipelines analíticos (dbt, Airflow), contratos de evento, modelagem dimensional, glossário de métricas | opus |
| `growth-engineer` | Experimentos A/B com rigor estatístico (spec pré-registrada, amostra/MDE, guardrails, análise) | opus |
| `finops` | Atribuição, otimização e governança de custo de cloud + SaaS de engenharia | opus |
| `code-reviewer` | Revisão de PR/MR (correctness, design, testes, contratos, observabilidade) antes do security/deploy | opus |
| `devops-engineer` | Manifests Kubernetes + GitLab CI + provisionamento | opus |
| `dev-security` | Trivy + análise de pentest, relatórios acionáveis | opus |
| `qa-automation` | Testes E2E em Gherkin + Cypress (web) / Appium (mobile iOS/Android) | sonnet |
| `release-manager` | Versão (SemVer), changelog, release notes, plano de rollout/rollback, comunicação | sonnet |
| `incident-responder` | SRE de plantão — diagnóstico e mitigação em produção, postmortem blameless | opus |
| `docs-writer` | Documentação consolidada em Docusaurus, legível por todos os perfis | sonnet |

> Você pode trocar o modelo no frontmatter (`model:`) conforme custo/latência. Para o orquestrador, `opus` é recomendado por causa do raciocínio sobre dependências entre agentes.

## As skills

Skills são procedimentos reutilizáveis carregados sob demanda quando o agente precisa executar uma operação específica. Vivem em `.claude/skills/<nome>/SKILL.md`.

| Skill | O que faz | Quem usa |
|---|---|---|
| `figjam-wireframer` | Cria wireframes low-fi e fluxos no FigJam via Figma MCP | `ux-designer` |
| `figma-builder` | Cria/modifica telas em alta fidelidade no Figma Design via Figma MCP | `ui-designer` |
| `figma-reader` | Lê tokens, componentes e hierarquia de telas do Figma (read-only) para alimentar implementação | `ui-designer`, `frontend-engineer`, `mobile-engineer` |
| `git-workflow` | Cria branch, faz commits no padrão Conventional Commits, gera descrição de PR/MR. **NUNCA executa push** | qualquer agente que modifique arquivos |

### Por que skills em vez de mais agentes

- **Zero duplicação.** Regras de "como mexer no Figma" ou "como nomear branch" ficam num lugar só.
- **Reuso real.** `figma-reader` é usada por 3 agentes diferentes; `git-workflow` por praticamente todos os que produzem código.
- **Manutenção barata.** Atualizar a integração com Figma MCP = atualizar a skill, não revisitar N agentes.
- **Permissões implícitas.** `figma-reader` é declaradamente read-only; `figma-builder` escreve; `git-workflow` jamais faz push. Mais fácil de raciocinar sobre o que pode dar errado.

## Pipeline UX → UI → Código com Figma MCP

```
ux-designer ──(usa)──► figjam-wireframer  ──► FigJam (wireframes low-fi)
     │
     ▼ docs/ux/<feature>.md
   [HUMANO valida wireframes no FigJam]
     │
     ▼
ui-designer ──(usa)──► figma-builder      ──► Figma Design (alta fidelidade)
     │           └───► figma-reader (inspeciona a library antes de criar)
     ▼ docs/ui/<tela>.md
   [HUMANO valida telas finais no Figma]
     │
     ▼
frontend-engineer ──(usa)──► figma-reader ──► código React + shadcn/ui + Tailwind
mobile-engineer    ──(usa)──► figma-reader ──► código React Native + NativeWind
     │
     ▼ (todos terminam o trabalho usando git-workflow)
   git-workflow ──► branch + commits + descrição de PR pronta
   [HUMANO revisa, faz git push, abre PR]
```

## Antes de usar (setup)

1. **Conecte o Figma MCP no Claude Code** — servidor remoto em `https://mcp.figma.com/mcp`. Documentação: https://help.figma.com/hc/en-us/articles/32132100833559

2. **Garanta que `frontend-engineer.md` e `mobile-engineer.md` declarem uso das skills relevantes** no frontmatter ou corpo:

   ```markdown
   <skills_disponiveis>
   - figma-reader — para extrair tokens, hierarquia e componentes de telas do Figma
   - git-workflow — para versionar mudanças e preparar descrição de PR/MR
   </skills_disponiveis>

   <comportamentos_obrigatorios>
   - Ao terminar com arquivos alterados, use git-workflow para criar branch, 
     commitar e preparar a descrição do PR.
   - NUNCA execute git push — isso é responsabilidade humana.
   </comportamentos_obrigatorios>
   ```

3. **Adicione a regra geral de git-workflow no `orchestrator.md`**: agentes que modificam arquivos devem encerrar com a skill `git-workflow`. Isso evita ter que repetir em cada agente downstream.

4. **(Opcional mas recomendado)** Crie `docs/ui/design-system.md` mapeando nomes de componentes do Figma para componentes shadcn/RN. Isso melhora muito a qualidade da extração feita pelo `figma-reader`.

## Como acionar

### Demanda nova, ponta a ponta (em etapas, com humano no meio)

```
1. Use o ux-designer para a feature "recuperação de senha". 
   FigJam: <link>. Demanda: docs/product/auth/recuperacao-senha.md
[revisa wireframes no FigJam, valida docs/ux/]

2. Use o ui-designer. UX spec: docs/ux/recuperacao-senha.md. 
   Figma Design: <link>
[revisa spec em docs/ui/ e telas no Figma]

3. Use o frontend-engineer para implementar a tela web. 
   UI spec: docs/ui/recuperacao-senha-web.md. Frame Figma: <link>
[revisa branch+commits localmente, faz push, abre PR]

4. Use o mobile-engineer para a versão mobile. 
   UI spec: docs/ui/recuperacao-senha-mobile.md. Frame Figma: <link>
[revisa branch+commits localmente, faz push, abre PR]
```

### Iteração pontual

```
A tela de recuperação de senha precisa de um novo estado: "email não verificado". 
Use o ui-designer para atualizar a spec e o Figma. 
UX spec: docs/ux/recuperacao-senha.md (já tem o caso)
```

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

## Pontos de atenção

- **Figma MCP**: a criação de conteúdo nativo via MCP está em **beta gratuito** mas vai virar recurso pago baseado em uso. Acompanhe o anúncio oficial do Figma.

- **Subagent + skill ainda em evolução**: a interação entre os dois está amadurecendo no Claude Code. Se notar que uma skill não está sendo carregada bem dentro de um subagent, alternativas:
  - Invocar a skill explicitamente no prompt do usuário
  - Reverter para um subagent independente (aceitando a duplicação de regras)

- **Stops humanos**: só funcionam se você **não usar o orquestrador end-to-end** no começo. Acione um agente por vez até confiar no pipeline. O `ux-designer` e `ui-designer` têm stops explícitos para validação humana antes do próximo agente; respeite-os.

- **Qualidade do figma-builder** depende fortemente de uma library Figma bem organizada (componentes nomeados, estilos publicados, variáveis configuradas). Library bagunçada = output bagunçado.

- **git-workflow nunca pusha**: a skill está blindada em três camadas para não executar `git push`, `gh pr create` ou similares. O push é sempre humano — esse é o último ponto de validação antes do código chegar ao remoto.

## Princípios aplicados em todos os prompts

- **Role claro logo no topo** — Claude responde melhor sabendo exatamente quem ele é nesse contexto.
- **XML tags semânticas** (`<role>`, `<processo>`, `<output_format>`, `<regras_de_qualidade>`) — facilita o parsing e reduz ambiguidade.
- **Output format explícito** — cada agente sabe exatamente o que entregar e onde.
- **`investigate_before_answering`** — força leitura do contexto real do repositório antes de propor mudanças. Reduz alucinação.
- **`default_to_action` vs `do_not_act_before_instructions`** — calibrado por agente conforme o nível de risco da ação.
- **Critérios de qualidade declarados** — cada agente conhece suas próprias regras de "pronto".
- **Handoffs explícitos** — o que sai de um agente é o que entra no próximo, com caminhos de arquivo concretos.
- **Skills para procedimentos repetitivos** — operações mecânicas (Figma, git) ficam em skills compartilhadas, não duplicadas em cada agente.

## Customização

Cada arquivo é um Markdown editável. Ajustes recomendados ao adotar:

1. **Stack**: se seu time usa Vue em vez de React, Helm em vez de Kustomize, GitHub Actions em vez de GitLab CI — edite o bloco `<stack_obrigatorio>` correspondente e o `<stack_visual>` no `ui-designer`.
2. **Paths**: ajuste `docs/`, `services/`, `deploy/` conforme a convenção do seu repositório.
3. **Modelo**: troque `model: sonnet` por `model: haiku` em agentes leves para reduzir custo, ou para `model: opus` em agentes que exigem mais raciocínio.
4. **Idioma**: os agentes estão em PT-BR; troque conforme necessidade do time.
5. **Convenções de git**: o `git-workflow` segue Conventional Commits e `tipo/escopo-descricao` para branches. Se seu time usa GitFlow, trunk-based ou outra convenção, edite a skill.
6. **Ferramentas de design**: se seu time usa outras ferramentas além de Figma/FigJam (Penpot, Miro, etc.), as três skills de Figma podem ser substituídas por equivalentes.

## Referências

- [Claude prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)
- [Claude Code subagents documentation](https://docs.claude.com/en/docs/claude-code/sub-agents)
- [Claude Code skills documentation](https://docs.claude.com/en/docs/claude-code/skills)
- [Figma MCP server](https://help.figma.com/hc/en-us/articles/32132100833559)
- [Conventional Commits](https://www.conventionalcommits.org/)