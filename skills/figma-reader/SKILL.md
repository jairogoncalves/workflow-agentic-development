---
name: figma-reader
description: Use esta skill para LER arquivos do Figma Design via Figma MCP e extrair informações estruturadas que serão consumidas por código — design tokens, hierarquia de camadas, componentes, propriedades de auto-layout, estilos aplicados, assets. Acionada antes de implementar UI em React/React Native, para gerar componentes a partir de telas no Figma. Triggers: "extrair do Figma", "ler tela do Figma", "implementar essa tela", "code from Figma", "Figma to code", "tokens do Figma". NÃO use para criar ou modificar conteúdo no Figma (use figma-builder ou figjam-wireframer).
---

# Figma Reader

Esta skill ensina como ler estruturas do Figma Design via MCP para alimentar a implementação de código em React (web, shadcn/ui + Tailwind) e React Native (NativeWind).

## Pré-requisitos

- Figma MCP remoto conectado (`https://mcp.figma.com/mcp`)
- Link ou ID do frame/tela a ser implementada (selecionado pelo usuário no Figma desktop, ou passado explicitamente)
- Acesso ao arquivo Figma (você precisa ter permissão de leitura)
- Conhecimento da stack alvo: web usa shadcn/ui + Tailwind; mobile usa React Native + NativeWind

Se o link/seleção não for fornecido, PARE e peça.

## Princípios fundamentais

**Esta skill é READ-ONLY.** Nunca modifique nada no Figma — todas as operações são de leitura. Se precisar modificar, é trabalho de outra skill.

**Token-first.** Antes de extrair pixels, extraia tokens. Cores, fontes e spacing devem ser mapeados para as variáveis CSS / tema Tailwind do projeto, não copiados como valores literais.

**Componente antes de markup.** Se um node no Figma é uma instância de componente da library, no código ele deve virar uma instância do componente equivalente do design system (shadcn/ui Button, Input, Card, etc.), não markup HTML/JSX cru.

**Auto-layout vira flex.** As propriedades de auto-layout do Figma (direção, gap, padding, alinhamento, distribuição) traduzem-se diretamente para classes Tailwind: `flex`, `flex-col`, `gap-X`, `p-X`, `items-X`, `justify-X`.

## Processo

### 1. Mapeie o contexto do design system primeiro

Antes de extrair a tela específica:

1. Liste estilos publicados (cores, tipografia, effects)
2. Liste componentes da library
3. Liste variáveis e modes (tema claro/escuro)
4. Verifique se o projeto tem mapeamento Figma ↔ código documentado em `docs/ui/design-system.md`

Se houver `design-system.md`, ele é a fonte de verdade do mapeamento. Use-o.
Se não houver, **infira o mapeamento e proponha criar esse arquivo** ao final.

### 2. Leia o frame/tela alvo

Extraia, na ordem:

1. **Metadados:** nome do frame, dimensões base, breakpoints presentes
2. **Hierarquia:** árvore de nodes com tipos (FRAME, INSTANCE, TEXT, etc.)
3. **Para cada node:**
   - É instance de componente? → guarde o nome do componente da library
   - Tem auto-layout? → extraia direção, gap, padding, align, justify
   - Tem estilos aplicados? → guarde o nome do estilo (não o valor RGB)
   - É texto? → extraia conteúdo + estilo de tipografia
   - Tem assets (imagem, vetor, ícone)? → guarde referência para exportação

### 3. Mapeie para a stack alvo

#### Web (shadcn/ui + Tailwind)

Para cada instance de componente do Figma, mapeie para o componente shadcn equivalente:

| Figma | shadcn/ui |
|---|---|
| `Component / Button / Primary` | `<Button>` |
| `Component / Button / Secondary` | `<Button variant="secondary">` |
| `Component / Input / Default` | `<Input>` |
| `Component / Card` | `<Card>` + `<CardHeader>` + `<CardContent>` |
| `Component / Dialog` | `<Dialog>` + `<DialogContent>` |
| ... | ... |

(Ajuste essa tabela conforme a library específica do projeto. Se incerto, leia `docs/ui/design-system.md` ou pergunte.)

Para auto-layout → Tailwind:
- Direction horizontal + gap 16 → `flex gap-4`
- Direction vertical + gap 8 → `flex flex-col gap-2`
- Padding 16 + 24 → `px-6 py-4`
- Align center + Justify space-between → `items-center justify-between`

Para estilos:
- Cor publicada `color/primary/default` → classe Tailwind `bg-primary` ou `text-primary`
- Tipografia publicada `text/heading/lg` → classe Tailwind correspondente do tema

#### Mobile (React Native + NativeWind)

Mesma lógica do web, mas:
- `<View>` no lugar de `<div>`
- `<Text>` obrigatório para qualquer texto
- `<Pressable>` ou `<TouchableOpacity>` para áreas clicáveis
- `<Image>` para imagens, sempre com `source` e dimensões
- Cuidado com safe area: use `<SafeAreaView>` no nível da tela
- Componentes shadcn não existem aqui — use o equivalente RN do design system (provavelmente em `mobile/src/components/ui/`)

### 4. Extraia assets

Para imagens, vetores e ícones que não sejam componentes:
- Identifique o node
- Exporte via MCP em formato adequado (SVG para vetores/ícones, PNG/WebP para fotos)
- Salve em:
  - Web: `web/public/assets/` ou `web/src/assets/`
  - Mobile: `mobile/src/assets/`
- Para ícones, prefira lucide-react (web) ou lucide-react-native (mobile) se o ícone existir na biblioteca — não exporte de novo o que já vem na lib.

### 5. Identifique estados ausentes

Se a spec em `docs/ui/` menciona estados (loading, empty, error) mas o Figma não tem variantes para eles, **avise no output**. Não invente UI: peça que o figma-builder complete antes, ou que um humano valide a interpretação.

### 6. Gere o relatório de extração

Esta skill **não escreve o código diretamente** — ela produz uma extração estruturada que o agente que a chamou (frontend-engineer ou mobile-engineer) usa para escrever o código com qualidade.

## Output esperado

```markdown
## Extração do Figma: <Feature>

**Arquivo:** <link>
**Frame:** <link direto> (<largura> x <altura>)
**Estados encontrados:** default, loading, error
**Estados na spec mas ausentes no Figma:** empty ⚠️

### Mapeamento de componentes (Figma → shadcn/ui)

| Node no Figma | Componente alvo | Props |
|---|---|---|
| `Button / Primary` "Enviar" | `<Button>` | `variant="default"`, `size="default"` |
| `Input / Default` "Email" | `<Input>` | `type="email"`, `placeholder="seu@email.com"` |
| ... | ... | ... |

### Estrutura sugerida do componente

\`\`\`tsx
// Esqueleto sugerido — o frontend-engineer refina
<form className="flex flex-col gap-4 p-6">
  <h1 className="text-2xl font-semibold">Recuperar senha</h1>
  <Input type="email" placeholder="seu@email.com" />
  <Button>Enviar link</Button>
</form>
\`\`\`

### Tokens identificados
- Cores: `bg-background`, `text-foreground`, `bg-primary`, `text-primary-foreground`
- Tipografia: `text-2xl font-semibold` (heading), `text-sm` (label)
- Spacing: `gap-4`, `p-6`

### Assets a exportar
- Nenhum  
OU  
- `<nome>.svg` → exportar para `web/src/assets/`

### Pendências
- Estado `empty` ausente no Figma → pedir ao figma-builder antes de implementar
- ...

### Próximo passo
Use esta extração como base para implementar o componente. 
NÃO copie pixels — use os componentes shadcn/NativeWind mapeados acima.
```

## Critérios de qualidade ("pronto")

- [ ] Todos os nodes relevantes foram extraídos
- [ ] Mapeamento Figma → componente está feito para cada instance
- [ ] Tokens estão expressos como classes/variáveis, não como valores RGB
- [ ] Auto-layout traduzido para classes flex/gap/padding
- [ ] Estados ausentes foram sinalizados
- [ ] Assets identificados (mesmo que não exportados ainda)

## NUNCA faça

- Modificar qualquer coisa no Figma (esta skill é read-only)
- Copiar valores literais (`#3B82F6`, `16px`) — sempre mapeie para token
- Sugerir markup `<div>` cru quando há componente da library para usar
- Inventar UI para estados que não existem no Figma — sinalize a ausência
- Escrever o código final completo — esta skill só extrai; o agente chamador implementa
