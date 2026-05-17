---
name: git-workflow
description: Use esta skill sempre que um agente precisar criar branch, adicionar arquivos ao stage, commitar mudanças, ou gerar a descrição (título + corpo) de um Pull Request/Merge Request. Acionada quando o agente terminou um trabalho (código, docs, spec, configuração) e precisa versionar e preparar o handoff para revisão humana. Triggers: "commitar", "criar branch", "abrir PR", "abrir MR", "versionar essa mudança", "preparar para merge", "salvar no git". REGRA ABSOLUTA: esta skill NUNCA executa git push. Push é responsabilidade do humano.
---

# Git Workflow

Esta skill padroniza como agentes criam branches, fazem commits e preparam descrições de PR/MR. Garante histórico limpo e mensagens consistentes.

## Regra absoluta

**Esta skill NUNCA executa `git push`.** Push é responsabilidade exclusiva do humano que revisa o trabalho. Se um agente perguntar se deve fazer push, a resposta é sempre não — apenas prepare tudo localmente e reporte que está pronto para o humano revisar e pushar.

Comandos proibidos para a skill:
- `git push` (em qualquer variação)
- `git push --force` / `git push -f`
- `gh pr create` / `glab mr create` (criar o PR/MR em si também é do humano após o push)
- Qualquer comando que publique ao remoto

Comandos permitidos:
- `git status`, `git diff`, `git log` (leitura)
- `git checkout`, `git switch`, `git branch` (manipulação local)
- `git add`, `git restore`, `git reset` (stage local)
- `git commit` (commit local)
- `git fetch` (atualizar referências, sem modificar working tree)

## Pré-requisitos

Antes de qualquer operação:

1. Estar dentro de um repositório git (`git rev-parse --is-inside-work-tree`)
2. Confirmar a branch base (geralmente `main` ou `develop` — verifique o que o projeto usa)
3. Verificar se há mudanças pendentes não-commitadas antes de trocar de branch

Se faltar qualquer um, PARE e reporte. Não force.

## Processo

### 1. Decida a branch antes de tudo

Verifique em qual branch está agora:
```bash
git branch --show-current
```

**Nunca commite direto em `main`, `master`, `develop`, `production` ou qualquer branch protegida.** Se estiver em uma dessas, crie uma nova branch antes do primeiro commit.

#### Convenção de nomeação de branch

Formato: `<tipo>/<escopo-curto>-<descricao-kebab>`

Tipos:
- `feature/` — funcionalidade nova
- `fix/` — correção de bug
- `chore/` — manutenção (deps, configs, build)
- `docs/` — só documentação
- `refactor/` — refatoração sem mudança de comportamento
- `test/` — só testes
- `perf/` — performance
- `style/` — formatação, lint (sem mudança de lógica)

Exemplos bons:
- `feature/auth-recuperacao-senha`
- `fix/checkout-erro-cep-invalido`
- `docs/ui-design-system-tokens`
- `refactor/api-extracao-modulo-pagamento`

Exemplos ruins (não use):
- `feature/nova-feature` (genérico)
- `fix/bug` (genérico)
- `jairo/teste` (nome de pessoa, não escopo)
- `branch-1` (sem contexto)

Crie assim:
```bash
git checkout -b feature/auth-recuperacao-senha
```

Se a branch já existe e você precisa retomar:
```bash
git switch feature/auth-recuperacao-senha
```

### 2. Inspecione antes de adicionar

Antes de `git add`, **sempre** rode:
```bash
git status
git diff
```

Confira:
- Não há arquivos que não deveriam ser commitados (`.env`, builds, `node_modules`, segredos, dumps de banco, arquivos de IDE)
- As mudanças correspondem ao que foi pedido (não há alteração acidental em arquivo não relacionado)
- Não há arquivos enormes (>1MB) que deveriam ir para LFS ou nem entrar no repo

Se houver arquivo suspeito, PARE e reporte ao agente chamador antes de adicionar.

### 3. Adicione com escopo claro

Prefira `git add` específico em vez de `git add .`:
```bash
git add docs/ui/recuperacao-senha.md docs/ui/design-system.md
```

Se o trabalho realmente abrange muitos arquivos relacionados, `git add .` é aceitável, mas só **depois** de confirmar com `git status` que tudo é relevante.

**Nunca** use `git add -A` em um repo que não está limpo — risco de incluir lixo.

### 4. Faça commits atômicos com Conventional Commits

Cada commit deve representar **uma mudança coerente** que poderia ser revertida sozinha. Se você fez três coisas diferentes, faça três commits.

Formato (Conventional Commits):
```
<tipo>(<escopo>): <descrição em minúsculas, imperativo, sem ponto final>

<corpo opcional explicando o porquê, não o quê>

<rodapé opcional: refs, breaking changes, co-authors>
```

Tipos:
- `feat` — nova funcionalidade
- `fix` — correção de bug
- `docs` — só documentação
- `refactor` — refatoração
- `test` — adicionar/ajustar testes
- `chore` — manutenção
- `perf` — performance
- `style` — formatação
- `build` — build system, dependências
- `ci` — pipelines

Escopo: módulo, área ou componente afetado (`auth`, `checkout`, `ui-designer`, `api`, etc.). Opcional, mas use sempre que ajudar.

Descrição: imperativo presente ("adiciona X", não "adicionado X" nem "adicionando X"). Limite 72 caracteres no título.

Corpo: explique **por que** a mudança foi feita, não o que (o diff já mostra o quê). Linhas em até 80 caracteres. Use parágrafos curtos.

Exemplos:

```
feat(auth): adiciona endpoint de recuperação de senha

Cria POST /auth/recover que envia link com token de 1h.
Token é JWT assinado com a chave de auth, claim 'purpose: password_reset'.
Validação de email é case-insensitive e remove espaços.

Refs: AUTH-142
```

```
fix(checkout): corrige cálculo de frete para CEPs de Minas Gerais

A API externa retorna estado em lowercase para MG, o que quebrava
o switch case. Normalizamos para uppercase antes da comparação.

Refs: CHECKOUT-87
```

```
docs(ui): adiciona spec de recuperação de senha

Refs: AUTH-142
```

Comando:
```bash
git commit -m "feat(auth): adiciona endpoint de recuperação de senha" \
  -m "Cria POST /auth/recover que envia link com token de 1h." \
  -m "Refs: AUTH-142"
```

Ou para mensagens longas, use um arquivo temporário e `git commit -F`:
```bash
cat > /tmp/commit-msg.txt << 'EOF'
feat(auth): adiciona endpoint de recuperação de senha

Cria POST /auth/recover que envia link com token de 1h.
Token é JWT assinado com a chave de auth, claim 'purpose: password_reset'.

Refs: AUTH-142
EOF
git commit -F /tmp/commit-msg.txt
```

### 5. Verifique o histórico antes de finalizar

```bash
git log --oneline -10
```

Confira:
- Mensagens seguem o padrão
- Commits estão atômicos (não há um commit "WIP" gigante misturando coisas)
- Não há commit acidental ("test", ".", "fix asdf")

Se algo está errado e ainda não foi pushado, é seguro reescrever:
```bash
git commit --amend             # ajusta o último commit
git rebase -i HEAD~3           # reescreve os últimos 3 commits
```

⚠️ Só reescreva se a branch é sua e nunca foi compartilhada. Em branch que já está no remoto, **não reescreva** — peça orientação humana.

### 6. Prepare a descrição do PR/MR

Esta skill não cria o PR — gera o **texto pronto** para o humano colar.

#### Convenção do título do PR/MR

Mesmo formato do Conventional Commit, sem o corpo:
```
feat(auth): adiciona fluxo completo de recuperação de senha
```

Use o tipo do trabalho **agregado** do PR, não do último commit. Se o PR contém `feat` + `docs` + `test` sobre o mesmo escopo, o título é `feat`.

#### Template do corpo do PR/MR

Salve em `/tmp/pr-description.md` (ou retorne inline para o humano colar):

```markdown
## O que muda

<2-4 linhas resumindo a mudança em linguagem de produto, não técnica. 
O que o usuário/desenvolvedor passa a ter depois deste merge?>

## Por que

<Motivação. Link para a issue/épico/spec. Qual problema isso resolve?>

- Refs: <AUTH-142 / docs/ux/recuperacao-senha.md / docs/ui/recuperacao-senha.md>

## Como foi feito

<Resumo técnico das principais decisões. Mencione arquivos/módulos 
relevantes. Se houver decisão não-óbvia, explique.>

- <ponto 1>
- <ponto 2>

## Como testar

<Passos concretos para o revisor reproduzir/validar localmente.>

1. <passo>
2. <passo>
3. <resultado esperado>

## Checklist

- [ ] Código segue as convenções do projeto (lint passa)
- [ ] Testes adicionados/atualizados quando aplicável
- [ ] Documentação atualizada (`docs/`, README, comentários)
- [ ] Sem segredos, chaves ou dados sensíveis no diff
- [ ] Migrations / breaking changes destacados abaixo (se houver)
- [ ] Screenshots/GIFs anexados (se UI)

## Screenshots / evidências

<Se for UI: prints antes/depois ou GIF de funcionamento.
Se for backend: exemplo de request/response ou log relevante.
Se for infra: output relevante.>

## Breaking changes

<Liste mudanças que quebram compatibilidade ou exigem ação de outros 
times. Se não houver, escreva "Nenhuma".>

## Rollout / rollback

<Se relevante: como ativar (feature flag?), como reverter, dependências 
de outros PRs.>
```

### 7. Reporte ao humano

Output final estruturado:

```markdown
## Git workflow concluído

**Branch criada:** feature/auth-recuperacao-senha
**Branch base:** main
**Commits:** 3

### Histórico
- `a1b2c3d` feat(auth): adiciona endpoint de recuperação de senha
- `d4e5f6g` test(auth): cobre cenários de email inválido e expiração
- `h7i8j9k` docs(auth): atualiza README com novo endpoint

### Arquivos modificados (resumo)
- `services/auth/routes/recover.py` (novo)
- `services/auth/tests/test_recover.py` (novo)
- `README.md` (atualizado)

### Descrição do PR pronta
Salva em `/tmp/pr-description.md`  
OU inline abaixo: <colar aqui>

### Próximo passo (humano)
1. Revisar `git log` e `git diff origin/main..HEAD`
2. Se OK: `git push -u origin feature/auth-recuperacao-senha`
3. Abrir PR/MR usando o título e corpo acima
4. Solicitar review

🔒 Esta skill NÃO executou push. Aguardando ação humana.
```

## Critérios de qualidade ("pronto")

- [ ] Não estou em branch protegida (main/master/develop/production)
- [ ] Nome da branch segue convenção `tipo/escopo-descricao`
- [ ] `git status` está limpo após os commits
- [ ] Cada commit é atômico e segue Conventional Commits
- [ ] Nenhum arquivo sensível ou de build foi commitado
- [ ] Descrição do PR está pronta com todos os campos preenchidos
- [ ] Reporte ao humano deixa claro que push é dele

## Casos especiais

### Trabalho em branch já existente
Se o humano já criou a branch e pediu para continuar nela, **não crie outra**. 
Use `git switch <branch>` e confirme antes de commitar.

### Múltiplos agentes commitando na mesma branch
Se um agente anterior já commitou na branch, faça `git pull --ff-only` (ou apenas 
`git fetch` + `git status` para checar divergência) antes de continuar. Se houver 
divergência que requer merge/rebase, PARE e peça ao humano.

### Conflitos
Se aparecer conflito em qualquer operação local (rebase, cherry-pick), PARE 
imediatamente. Resolução de conflito é decisão de produto/código que cabe ao 
agente especialista (frontend, backend, etc.) ou ao humano — não a esta skill.

### Repositório monorepo
Em monorepos, inclua o pacote/app no escopo do commit:
```
feat(web/checkout): adiciona seleção de endereço salvo
feat(mobile/auth): adiciona biometria no login
```

### Trabalho que cobre múltiplos escopos
Se a mudança cobre web + mobile + backend, faça commits separados por escopo. 
Evite um commit gigante "feat: adiciona recuperação de senha em tudo".

## NUNCA faça

- `git push` em qualquer variação — push é do humano
- Commitar em `main`, `master`, `develop`, `production` ou outra branch protegida
- `git add -A` sem inspecionar `git status` antes
- Commitar arquivos `.env`, chaves, certificados, dumps, builds
- Reescrever histórico de uma branch que já foi pushada
- Forçar `git push -f` (nem o push permitido seria com force)
- Resolver conflitos sem consultar o humano ou o agente especialista
- Criar o PR/MR no GitHub/GitLab — só preparar o texto
- Misturar mudanças não relacionadas no mesmo commit
- Mensagem de commit genérica ("update", "fix", "wip", ".")
