# Rules vs Skills: como dar memória e habilidades ao seu agente de IA

No dia a dia, precisamos que o agente lembre de algumas coisas entre uma sessão e outra. Informações como "neste projeto usamos pnpm, não npm" ou "antes de finalizar qualquer tarefa, rode o lint" precisam estar sempre disponíveis. Existem também processos que só são usados em momentos específicos, como "gerar um PRD" ou "escrever release notes", e que não fazem sentido carregar o tempo todo. Para resolver essas duas necessidades, surgem dois conceitos importantes no trabalho com agentes de IA: as Rules e as Skills.


Antes de seguir, vale lembrar: a janela de contexto da LLM (large language model, o modelo de linguagem por trás de ferramentas como o Claude, o ChatGPT ou o Gemini) funciona como uma short-term memory (memória de curto prazo) tudo que está nela se perde quando você fecha a sessão ou limpa o contexto.

Neste post, vamos entender o que é cada uma, quando usar uma ou a outra e como combinar as duas para manter o agente útil sem sobrecarregar a janela de contexto.

## TL;DR

- **Rules**: sempre carregadas no contexto. Use para o que vale em qualquer tarefa (stack, convenções, comandos obrigatórios).
- **Skills**: carregadas sob demanda. Use para workflows específicos (gerar PRD, criar componente, code review).
- **Teste rápido**: se a instrução vale "mesmo sem ninguém pensar nela", é Rule. Se vale "só quando estou fazendo X", é Skill.

| | Rules | Skills |
|---|---|---|
| Quando carrega | Sempre | Quando acionada |
| Onde mora | Arquivo de rules (ex.: `CLAUDE.md`) | Pasta própria (ex.: `.claude/skills/<nome>/`) |
| Custo de contexto | Constante | Só ao usar |
| Bom para | Convenções, comandos obrigatórios | Workflows pontuais |

## Regras (Rules): a memória de longo prazo

As Rules são instruções que ficam injetadas no system prompt (o prompt base que o agente recebe antes de cada conversa). Por estarem sempre carregadas, elas funcionam como long-term memory (memória de longo prazo): persistem entre sessões, entre tarefas, entre dias diferentes de trabalho.

Cada ferramenta tem o seu nome de arquivo para guardar essas regras:

- Claude Code: `CLAUDE.md`
- Cursor: `.cursor/rules`
- Windsurf: `.windsurf/rules`
- GitHub Copilot: `.github/copilot-instructions.md`
- Factory: `.factory/rules`
- Padrão aberto adotado por várias ferramentas (Codex, Aider e outros): `AGENTS.md`
- Gemini: `GEMINI.md`

A função das Rules é guardar o que é verdade no seu projeto: padrões de arquitetura, convenções de nome, comandos obrigatórios antes de subir código, ferramentas oficiais do time. São instruções que precisam valer em qualquer tarefa, independente do que está sendo feito naquele momento.

### Caminhos para criar Rules (podem ser combinados)

1. **Do zero**: o time se reúne, discute o que faz sentido virar regra e escreve em conjunto. Esse caminho leva mais tempo, mas costuma gerar Rules mais alinhadas com a realidade do projeto.
2. **Copiar da internet**: existem repositórios públicos com Rules prontas para diversas stacks. É um bom ponto de partida, desde que você leia com atenção e remova tudo que não se aplica ao seu caso. Regras que não correspondem ao código do projeto acabam virando ruído na janela de contexto.
3. **Pedir para a LLM gerar a partir do código**: no Claude Code, o comando `/init` faz exatamente isso ele varre o projeto e gera um `CLAUDE.md` inicial. Funciona bem em projetos coesos. Em projetos que mudaram muito de stack ao longo do tempo, a LLM pode reproduzir contradições do próprio histórico, então a revisão humana continua sendo importante.

Uma sugestão prática: escreva as Rules primeiro no idioma do time, discuta com as pessoas envolvidas, refine e só depois (se fizer sentido) peça para o Claude traduzir para inglês e completar com exemplos. Vale prestar atenção no tamanho do arquivo. Rules longas demais ocupam espaço na janela de contexto que poderia estar sendo usado para o problema que você está resolvendo.

### Cuidado com excesso de Rules

Pense no seguinte cenário: você está fazendo uma tarefa de front-end e o agente carrega junto uma Rule extensa sobre padrões de SQL. Ou o contrário, trabalhando no banco de dados e recebendo regras de React. Nos dois casos, são tokens consumidos sem necessidade. Ter Rules é importante, mas em excesso elas acabam reduzindo o espaço disponível para o que a tarefa realmente exige.

### Estrutura de uma Rule

Diferente das Skills, as Rules não exigem um formato fixo. O agente lê o arquivo inteiro como parte do system prompt, então o que importa é a clareza do conteúdo: títulos curtos, listas objetivas e instruções diretas.

Um exemplo simples de `CLAUDE.md` para um projeto Node.js:

```
# Projeto: API de Pedidos

API REST para gerenciar os pedidos de uma loja online.
Stack: Node.js, Fastify e PostgreSQL.

## Convenções de código

- Sempre use TypeScript, nunca JavaScript puro.
- Nomes de arquivos em kebab-case (ex.: `order-service.ts`).
- Evite o tipo `any`. Quando precisar de algo flexível, prefira `unknown`.

## Antes de finalizar uma tarefa

1. Rode `pnpm lint` e corrija o que for apontado.
2. Rode `pnpm test` e garanta que todos os testes passam.
3. Não faça commit de arquivos `.env`.

## Skills disponíveis

- Para criar um novo endpoint REST, carregue a skill `novo-endpoint-rest`.
- Para gerar ou atualizar um README, carregue a skill `gerador-de-readme`.
```

Esse exemplo mostra três blocos comuns em uma Rule: o contexto do projeto (o que é, para que serve, qual a stack), as convenções que precisam valer sempre e os comandos obrigatórios em situações específicas. O último bloco aplica o padrão de roteamento para Skills, que será detalhado mais adiante: a Rule fica curta e aponta para a Skill correta quando o conhecimento é específico de um workflow.

## Habilidades (Skills): conhecimento sob demanda

As Skills foram popularizadas pela Anthropic no Claude Code. A diferença em relação às Rules é direta: uma Skill não fica sempre carregada na janela de contexto.

Cada Skill mora em uma pasta específica chamada `skills`, com tudo o que ela precisa para ser executada: um arquivo `SKILL.md`, pastas de templates, documentação de referência e exemplos. Quando o agente inicia uma sessão, apenas o cabeçalho da Skill (o YAML do topo do `SKILL.md`, que descreve o nome e o objetivo) é carregado. O conteúdo completo só entra na janela de contexto quando a Skill é efetivamente acionada.

Essa diferença muda o cálculo de forma significativa. É possível ter dezenas de Skills disponíveis em um projeto sem pesar no contexto, porque o que fica sempre presente é só a descrição curta de cada uma. Tentar fazer o mesmo com Rules consumiria muito mais tokens.

### Onde encontrar Skills prontas

Existem dois marketplaces (lojas) bastante usados para baixar Skills já feitas pela comunidade:

- `https://www.skills.sh/`
- `https://skillsmp.com/`

Lá você encontra desde Skills genéricas (gerar README, criar imagens, design front-end) até Skills mais específicas, como a de criação de PRD (Product Requirements Document, o documento de requisitos do produto).

Neste próprio repositório a Skill `prd-development` foi instalada como exemplo, em `.agents/skills/prd-development/`. Abrindo essa pasta, dá para ver a estrutura organizada: pasta `templates/`, pasta `references/` e o `SKILL.md` na raiz. Para usá-la dentro do Claude Code ou do Cursor, basta pedir de forma explícita: "Crie um PRD usando a skill `prd-development`, no formato que eu vou descrever a seguir".

## Estrutura de uma Skill

Toda Skill precisa de um arquivo `SKILL.md` com YAML frontmatter (cabeçalho em formato YAML, delimitado por três traços no topo do arquivo). Os campos obrigatórios são `name` e `description`:

```
---
name: gerador-de-readme
description: Cria README de projetos seguindo o padrão do time. Usar quando o usuário pedir para gerar, atualizar ou refinar um README.
---

# Gerador de README

## Instruções
1. Pergunte qual o nome e o propósito do projeto.
2. Pergunte qual a stack principal.
3. Gere o README seguindo o template em `templates/readme.md`.

## Exemplo
Veja `examples/api-de-pedidos.md` para um README pronto.
```

### Skills pré-construídas do Claude Code

O Claude Code já vem com algumas Skills nativas, prontas para uso sem instalação:

- **PowerPoint (pptx)**: criar e editar slides, analisar conteúdo de apresentações.
- **Excel (xlsx)**: criar planilhas, analisar dados e gerar relatórios com gráficos.
- **Word (docx)**: criar e editar documentos, formatar texto.
- **PDF (pdf)**: gerar PDFs formatados e relatórios.

## Rules x Skills: como decidir onde colocar cada coisa

Skills não substituem Rules. As Rules continuam sendo as instruções inegociáveis do projeto. Ter Skills disponíveis muda a forma de escrever as Rules, porque permite que elas fiquem mais enxutas.

Uma divisão que costuma funcionar bem:

Alguns exemplos para fixar:

- *"Nunca faça commit de arquivos .env"* (Rule)
- *"Sempre depois de alterar o código, rode o lint e os testes"* (Rule)
- *"Para escrever release notes, siga este formato e este checklist"* (Skill)
- *"Roteiro de code review com checklist de segurança, performance e testes"* (Skill)

Um padrão que funciona bem na prática é fazer as Rules atuarem quase como um roteador para as Skills. Dentro do `CLAUDE.md` do projeto, por exemplo:

```
- "Quando for mexer em componentes de UI, carregue a skill `ui-change`."
- "Quando for criar services, carregue a skill `create-service`."
- "Quando for criar specs, carregue a skill `create-spec`."
```

## Conclusão

As Rules e as Skills cumprem papéis complementares no agente. As Rules carregam o que é sempre verdade no projeto, o que precisa valer em qualquer tarefa. As Skills carregam conhecimento que entra em cena apenas quando é necessário, sem ocupar espaço na janela de contexto enquanto isso. Quando usadas em conjunto, elas mantêm a janela de contexto enxuta e o agente focado no problema que está sendo resolvido.

Na próxima aula, vamos avançar para outra parte importante da engenharia de contexto. Antes disso, uma sugestão prática: abra o `CLAUDE.md` ou o `.cursor/rules` do seu projeto e aplique o teste apresentado neste post. Tudo o que está nesse arquivo precisa estar sempre carregado? Existe algo ali que faria mais sentido como Skill?

## Referências

- https://skillsmp.com/
- https://www.builder.io/blog/agent-skills-rules-commands
- https://www.branas.io/formacoes/inteligencia-artificial
- https://platform.claude.com/docs/pt-BR/agents-and-tools/agent-skills/overview
- https://anthropic.skilljar.com/claude-code-101
