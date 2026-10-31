# AGENTS.md

Instruções gerais para agentes de IA que atuam neste repositório.

## Contexto

Este repositório reúne prompts versionados e a documentação de apoio a projetos
assistidos por agentes. Todo o conteúdo é escrito em português (pt-BR), salvo
quando o próprio artefato exigir outro idioma.

## Estrutura

```text
.
├── AGENTS.md
├── CLAUDE.md
├── .claude/
│   ├── skills/
│   │   └── <nome-da-skill>/
│   │       └── SKILL.md
│   ├── agents/
│   │   └── <nome-do-agente>.md
│   └── commands/
│       └── <nome-do-comando>.md
├── docs/
│   ├── specs/
│   ├── plans/
│   ├── features/
│   │   └── <nome-da-feature>/
│   │       └── FEATURE.md
│   └── adr/
│       └── 0001-<titulo>.md
└── prompts/
    └── <nome-do-prompt>.md
```

## O que vai em cada lugar

| Caminho | Conteúdo |
|---|---|
| `AGENTS.md` | Instruções gerais para qualquer agente. |
| `CLAUDE.md` | Ponto de entrada do Claude Code; referencia este arquivo. |
| `.claude/skills/` | Skills, uma pasta por skill com `SKILL.md`. |
| `.claude/agents/` | Subagentes, um `.md` por agente. |
| `.claude/commands/` | Slash commands e prompts reutilizáveis do Claude Code. |
| `docs/specs/` | Especificações: o quê e por quê. |
| `docs/plans/` | Planos de implementação: como e em que ordem. |
| `docs/features/` | Documentação de features, uma pasta por feature com `FEATURE.md`. |
| `docs/adr/` | Decisões de arquitetura (ADRs). |
| `prompts/` | Prompts versionados, fora de `.claude/`. |

## Fluxo de trabalho

1. Descreva o problema ou a ideia em `docs/specs/`.
2. Transforme a spec aprovada em um plano em `docs/plans/`.
3. Implemente seguindo o plano.
4. Registre o resultado em `docs/features/<nome>/FEATURE.md`.
5. Se houver uma decisão arquitetural relevante, crie um ADR em `docs/adr/`.
6. Se o trabalho gerar um prompt reutilizável, salve-o em `prompts/`.

## Convenções

- Use `kebab-case` em nomes de arquivos e pastas.
- Escreva a documentação em pt-BR, de forma objetiva.
- Numere ADRs com quatro dígitos: `0001-titulo-da-decisao.md`.
- Uma feature por pasta: `docs/features/<nome>/FEATURE.md`.
- Evite duplicar conteúdo; referencie o documento original.
- Mantenha `prompts/` apenas com prompts versionados. Slash commands ficam em
  `.claude/commands/`.

## Skills, subagentes e comandos

### Skill

Crie `.claude/skills/<nome>/SKILL.md` com front matter:

```yaml
---
name: nome-da-skill
description: Quando e por que usar esta skill.
---
```

### Subagente

Crie `.claude/agents/<nome>.md` com front matter contendo, no mínimo, `name` e
`description`. Inclua `tools` e `model` quando necessário.

### Slash command

Crie `.claude/commands/<nome>.md`. O nome do arquivo define o comando (`/nome`).

## Regras para agentes

- Leia este arquivo e os documentos relevantes antes de alterar o projeto.
- Preserve o conteúdo existente em `prompts/`; ele é versionado.
- Faça mudanças pequenas e focadas, mantendo a estrutura acima.
- Atualize a documentação junto com qualquer mudança de comportamento.
- Registre decisões relevantes em `docs/adr/`.
- Nunca inclua segredos, tokens ou dados sensíveis no repositório.
