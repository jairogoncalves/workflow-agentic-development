---
name: figma-builder
description: Use esta skill para criar e modificar telas em alta fidelidade dentro de arquivos do Figma Design via Figma MCP. Acionada quando já existe uma spec de UI (geralmente em docs/ui/<tela>.md) e/ou wireframes aprovados, e o trabalho é materializar as telas finais usando componentes do design system, design tokens, auto-layout e estilos publicados. Triggers: "criar tela no Figma", "materializar no Figma", "desenhar no Figma", "alta fidelidade", "Figma Design", "construir UI no Figma". NÃO use para wireframes low-fi (use figjam-wireframer) nem para apenas LER o Figma para gerar código (use figma-reader).
---

# Figma Builder

Esta skill ensina como criar telas em alta fidelidade no Figma Design usando o Figma MCP server, respeitando o design system existente e tokens publicados.

## Pré-requisitos

- Figma MCP remoto conectado (`https://mcp.figma.com/mcp`)
- Link do arquivo Figma Design onde criar a tela
- **Spec de UI aprovada** em `docs/ui/<tela>.md` (produzida pelo ui-designer)
- Wireframe de referência (opcional mas recomendado, geralmente em FigJam)

Se faltar qualquer um desses, PARE e peça antes de continuar. Materializar tela sem spec leva a retrabalho.

## Stack de referência visual

Este projeto usa **shadcn/ui + Tailwind** na web e **React Native + NativeWind** no mobile.
O Figma deve refletir essa biblioteca: componentes desenhados como variantes shadcn, tokens espelhando o `tailwind.config.js`.

Antes de criar qualquer coisa, identifique no arquivo Figma:
- A library de componentes (provavelmente nomeada como `Design System` ou similar)
- Os estilos de cor publicados (correspondentes a `--background`, `--foreground`, `--primary` etc.)
- Os estilos de tipografia publicados
- Tokens de spacing (correspondentes a Tailwind: `space-1`, `space-2`, etc.)

## Princípios fundamentais

**Reuso > criação.** Se um componente, estilo ou variante já existe na library, use-o. Nunca crie um botão novo se já existe `Button / Primary`. Procure exaustivamente antes de criar.

**Auto-layout sempre.** Posicionamento absoluto é proibido. Toda tela precisa ser responsiva por construção, com auto-layout aninhado e constraints corretas.

**Tokens, nunca valores literais.** Cor, fonte, spacing, radius — tudo via variável/estilo publicado. Nunca `#FFFFFF` direto; sempre `color/background/default`.

**Naming hierárquico e consistente.** Frames seguem `Screen / <Feature> / <Estado>`. Componentes seguem `Component / <Tipo> / <Variante>`.

## Processo

### 1. Carregue todo o contexto antes de criar

Faça isso na ordem:

1. Leia `docs/ui/<tela>.md` por inteiro — tokens, estados, comportamentos, acessibilidade
2. Abra o arquivo Figma via MCP e mapeie:
   - Componentes existentes na library
   - Estilos de cor/tipo/effect publicados
   - Variáveis (modes de tema claro/escuro, se houver)
   - Convenções de naming já usadas
3. Se houver wireframe de referência, abra e use como guia estrutural

**Nunca** comece a criar sem ter esse mapa mental. Não saber o que existe = duplicar componentes.

### 2. Crie a tela com auto-layout do topo

Estrutura recomendada para cada tela:

```
Screen / <Feature> / <Estado>          [auto-layout vertical, fill width/height]
├── Header                              [auto-layout horizontal, hug height]
├── Content                             [auto-layout vertical, fill height]
│   ├── Section 1
│   └── Section 2
└── Footer / Actions                    [auto-layout horizontal, hug height]
```

Para mobile, considere safe areas e a altura padrão (375x812 iPhone, 360x800 Android).
Para web, frames base de 1440x900 (desktop) + variantes 768 (tablet) + 375 (mobile).

### 3. Aplique componentes da library

Para cada elemento da spec:
- Procure o componente equivalente na library
- Use a variante correta (size, state, variant)
- Preencha as propriedades de instância (texto, ícone, estado)
- **Não destaque/desconecte instâncias** a menos que estritamente necessário e justificado

Se um componente realmente não existir e precisar ser criado:
1. Marque com sticky/comentário no Figma justificando
2. Crie como componente publicável (com variants se aplicável)
3. Documente no output final para o time de design revisar

### 4. Crie todos os estados especificados

Para cada estado da spec (`default`, `loading`, `empty`, `error`, `success`, estados de hover/focus/disabled em elementos), crie uma variante ou frame separado. Não basta criar só o "feliz".

Convenção: `Screen / Recuperação de Senha / 1. Solicitar email - empty`

### 5. Valide acessibilidade visual

Antes de finalizar:
- Contraste mínimo AA (4.5:1 texto normal, 3:1 texto grande)
- Hit targets ≥ 44x44px (mobile) / ≥ 32x32px (web)
- Foco visível em elementos interativos
- Ordem de leitura faz sentido (importante para leitores de tela)

### 6. Conecte as telas (opcional, mas útil)

Use prototyping do Figma para conectar a navegação entre estados/telas. Facilita revisão humana e validação do fluxo.

## Ferramentas do MCP a usar

Tipicamente disponíveis no Figma MCP remoto:
- `create_frame` — criar frames com auto-layout
- `create_instance` — instanciar componentes da library
- `set_variable_mode` — aplicar tokens
- `get_local_styles` / `get_published_styles` — listar estilos disponíveis
- `get_components` — listar componentes da library
- `set_properties` — preencher variant properties

Sempre **leia antes de escrever**. Sempre.

## Critérios de qualidade ("pronto")

- [ ] Todos os estados da spec foram criados
- [ ] Todo elemento usa componente da library OU está justificado como novo
- [ ] Zero valores hardcoded — tudo via token/estilo
- [ ] Auto-layout em todos os níveis, sem posicionamento absoluto
- [ ] Naming segue padrão `Screen / Feature / Estado`
- [ ] Contraste e hit targets validados
- [ ] Responsivo: se web, ao menos breakpoints desktop+mobile; se mobile, considera notch/safe area
- [ ] Pendências marcadas com comentários no Figma para o designer humano

## Output esperado

```markdown
## Tela materializada: <Feature>

**Arquivo Figma:** <link>
**Frame principal:** <link direto>

### Estados criados
- default — <link>
- loading — <link>
- empty — <link>
- error — <link>
- ...

### Componentes reutilizados da library
- Button / Primary (3 instâncias)
- Input / Default (2 instâncias)
- ...

### Componentes/estilos NOVOS criados (requerem revisão humana)
Nenhum  
OU  
- <Nome> — <link> — <justificativa>

### Tokens aplicados
- Cores: <lista>
- Tipografia: <lista>
- Spacing: <lista>

### Pendências para revisão humana
1. <Decisão tomada que vale validar> — <link no Figma>
2. ...

### Próximo passo
PARE aqui. Aguarde validação humana das telas no Figma antes de prosseguir.
Após aprovação, o frontend-engineer (web) ou mobile-engineer pode implementar 
usando a skill figma-reader para extrair os detalhes técnicos.
```

## NUNCA faça

- Criar componentes paralelos a outros já existentes ("Button2", "ButtonNew")
- Usar valores literais de cor, fonte ou spacing
- Posicionar com X/Y absolutos sem auto-layout
- Destacar/desconectar instâncias por conveniência
- Pular estados ("crio só o default, depois alguém faz o resto")
- Avançar para implementação de código — isso é trabalho do frontend-engineer
- Invocar outro agente ou skill no final — apenas reporte e pare
