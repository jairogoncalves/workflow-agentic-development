---
name: figjam-wireframer
description: Use esta skill quando precisar criar, modificar ou organizar wireframes low-fi e fluxos de jornada no FigJam via Figma MCP. Acionada quando o trabalho envolve esboçar estrutura, fluxo entre telas, mapas de jornada, blocos de conteúdo em baixa fidelidade ou exploração rápida de ideias de UX — antes de existir uma tela final em alta fidelidade. Triggers: "wireframe", "low-fi", "esboço", "fluxo de telas", "jornada do usuário", "FigJam", "estruturar tela", "rascunho de tela". NÃO use esta skill para criar telas finais em alta fidelidade — para isso, use a skill figma-builder.
---

# FigJam Wireframer

Esta skill ensina como criar wireframes low-fi e fluxos de jornada no FigJam usando o Figma MCP server.

## Pré-requisitos

- Figma MCP remoto conectado (`https://mcp.figma.com/mcp`)
- Link/URL do arquivo FigJam onde criar o wireframe (deve ser passado pelo usuário ou agente que chama a skill)
- Existência de uma demanda clara: feature, fluxo ou problema a explorar

Se faltar o link do FigJam, PARE e peça ao usuário antes de continuar.

## Princípios fundamentais

**Wireframe é sobre estrutura, não estética.** Esta skill cria artefatos de baixa fidelidade. Não aplicar cores da marca, fontes finais ou ícones detalhados. Tudo em escala de cinza com placeholders.

**Pense fluxo antes de tela.** Antes de desenhar telas individuais, mapeie a sequência: ponto de entrada → ações → estados (sucesso, erro, vazio, carregando) → saída.

**Cubra os caminhos alternativos.** Wireframe que só mostra o caminho feliz é wireframe incompleto. Sempre incluir: estado vazio, estado de erro, estado de carregamento, caminho de cancelamento/voltar.

## Processo

### 1. Leia o contexto antes de criar

- Abra o arquivo FigJam atual via MCP
- Identifique seções existentes, convenções de nomeação, frames já presentes
- Se houver outras features wireframadas no mesmo arquivo, **siga o padrão visual e organizacional já estabelecido**

### 2. Estruture o board

Crie no FigJam (nessa ordem):

1. **Frame de cabeçalho** com:
   - Nome da feature
   - Autor (agente que criou) e data
   - Link para a demanda/issue original (se houver)
   - Hipótese ou pergunta que o wireframe responde

2. **Seção "Fluxo"** — diagrama de blocos conectados mostrando a jornada do usuário entre telas

3. **Seção "Telas (low-fi)"** — uma área por tela do fluxo

4. **Seção "Estados"** — variações de cada tela (vazio, erro, loading, sucesso)

5. **Seção "Perguntas em aberto"** — sticky notes amarelos com decisões pendentes para validação humana

### 3. Desenhe cada tela em wireframe

Para cada tela, use blocos retangulares com labels textuais. Convenções:

- **Retângulo cinza claro com texto centralizado** = bloco de conteúdo (ex: "Lista de produtos")
- **Retângulo com borda tracejada** = área de interação principal (botão, input)
- **Linha de texto sublinhada** = link ou ação secundária
- **Caixa com X dentro** = imagem/mídia (placeholder)
- **Sticky note amarelo** = comentário ou pergunta para revisão humana

Nomeie todos os frames seguindo o padrão: `Wireframe / <Feature> / <Nº>. <Nome da tela>`

Exemplo: `Wireframe / Recuperação de Senha / 1. Solicitar email`

### 4. Conecte as telas

Use setas do FigJam para mostrar transições entre telas. Cada seta deve ter um label curto descrevendo o gatilho (ex: "ao clicar em Enviar", "se email inválido", "após 3 tentativas").

### 5. Marque pontos de validação humana

Em **toda** decisão de design que envolva trade-off ou suposição, adicione um sticky note amarelo com:
- A pergunta
- As alternativas consideradas
- A escolha atual e por quê

Isso é o que permite ao humano validar de forma estruturada, não só "gostei/não gostei".

## Ferramentas do MCP a usar

As ferramentas variam conforme versão do Figma MCP, mas tipicamente:

- `create_frame` / `create_node` — criar áreas e blocos
- `create_text` — adicionar labels
- `create_connector` — setas entre telas
- `create_sticky_note` — perguntas em aberto
- `get_file` / `get_nodes` — ler estado atual antes de criar

Sempre **leia antes de escrever**. Nunca crie sem verificar o que já existe.

## Critérios de qualidade ("pronto")

Antes de finalizar, verifique:

- [ ] Todas as telas do fluxo estão presentes (incluindo caminhos alternativos)
- [ ] Cada tela tem seus estados (vazio, erro, loading) quando aplicável
- [ ] Conexões entre telas estão claras e com labels
- [ ] Não há cor, fonte ou estilo que sugira tela final — só estrutura
- [ ] Perguntas em aberto estão marcadas com sticky notes
- [ ] Frames estão nomeados no padrão
- [ ] Cabeçalho com contexto está presente

## Output esperado

Ao concluir, retorne em markdown:

```markdown
## Wireframe criado: <Feature>

**Arquivo FigJam:** <link>
**Frame de entrada:** <link direto para o frame de cabeçalho>

### Telas wireframadas
- 1. <Nome> — <link do frame>
- 2. <Nome> — <link do frame>
- ...

### Estados cobertos
- <Tela X>: vazio, erro, loading
- ...

### Perguntas em aberto para validação humana
1. <Pergunta> — <link do sticky note>
2. ...

### Próximo passo
PARE aqui. Aguarde validação humana do wireframe antes de prosseguir.
Após aprovação, o ui-designer pode usar este wireframe + as decisões 
documentadas para produzir a spec em docs/ui/.
```

## NUNCA faça

- Aplicar cores da identidade visual (use só cinzas)
- Usar a tipografia final do produto (mantenha fonte padrão do FigJam)
- Desenhar componentes detalhados (botões com sombra, cards com gradiente, etc.)
- Avançar para alta fidelidade — isso é trabalho da skill figma-builder
- Tomar decisões de design não-óbvias sem registrar em sticky note
- Invocar outro agente ou skill no final — apenas reporte e pare
