# Sybilion Dojo Extension

Extensão oficial da **Sybilion** para o [Spec-Kit](https://github.com/github/spec-kit).

Este repositório **é a extensão** — não é uma aplicação. É consumido por outros repositórios através da CLI do Spec-Kit (`specify`), que aplica os comandos, templates e hooks aqui definidos ao projecto destino.

## O que define

- **Constitution** corporativa Sybilion (princípios, engineering standards e workflow discipline) — language-agnostic, aplica-se a qualquer projecto.
- Comando `speckit.sybilion-dojo.init` — copia a constitution template para `.specify/memory/constitution.md` antes de a constitution ser gerada.
- Comando `speckit.sybilion-dojo.frame` — clarificação Socrática do intent antes de `/speckit.specify` redigir a spec (defere para o skill `brainstorming` do Superpowers quando presente).
- Comando `speckit.sybilion-dojo.contract` — gera contratos blackbox Dojo a partir do `spec.md` e `plan.md`, incluindo `tests/blackbox/dojo.yaml`, `COVERAGE.md` e fixtures.
- Comando `speckit.sybilion-dojo.validate` — corre o formatter, linter, type checker, unit tests e testes Dojo até passarem (Execution Loop completa).
- **Hooks** que ligam estes comandos ao ciclo de vida do Spec-Kit:
  - `before_constitution` → init
  - `before_specify` → frame (optional)
  - `after_plan` → contract
  - `after_implement` → validate

## Estrutura

```
.
├── extension.yml                          # Manifesto da extensão
├── COMPOSITION.md                         # Regras de composição com plug-ins companheiros (Superpowers)
├── commands/
│   ├── init.md                            # /speckit.sybilion-dojo.init
│   ├── frame.md                           # /speckit.sybilion-dojo.frame
│   ├── contract.md                        # /speckit.sybilion-dojo.contract
│   └── validate.md                        # /speckit.sybilion-dojo.validate
└── templates/
    └── constitution-template.md           # Constitution Sybilion (estratégia: replace)
```

## Requisitos

- `specify` CLI do Spec-Kit (versão >= 0.8.0).
- `dojo` disponível no `PATH` (`go install github.com/elmacnifico/dojo@latest`) — necessário em runtime para o comando de validação.
- O projecto destino traz o seu próprio toolchain (formatter, linter, type checker, test runner) declarado no respectivo manifesto. A extensão é language-agnostic.

## Compatibilidade

Compõe com o plug-in [Superpowers](https://github.com/obra/superpowers) para Claude Code (`/plugin install superpowers@claude-plugins-official`). Quando instalado:

- O skill `brainstorming` auto-activa-se durante `/speckit.sybilion-dojo.frame` (hook `before_specify`), reforçando o questionamento Socrático.
- O skill `test-driven-development` aplica-se ao loop unit-test em paralelo com o contrato Dojo blackbox (que continua a ser da exclusiva responsabilidade do `/speckit.sybilion-dojo.contract`).
- O skill `systematic-debugging` auto-activa-se quando `/speckit.sybilion-dojo.validate` encontra um sintoma fora da tabela "Failure Modes".

Sem o plug-in, as mesmas disciplinas continuam a ser aplicadas pelos hooks e comandos desta extensão — a composição é optimização, não dependência.

Detalhes técnicos da composição (regras, defer, limitações conhecidas) em [COMPOSITION.md](COMPOSITION.md).

## Instalação no repositório destino

A partir do repositório onde se quer aplicar a extensão:

```bash
# Via referência ao Git remoto (recomendado para a equipa)
specify extension add <git-url-deste-repo>

# Ou apontando para um clone local (modo dev, para iterar na extensão)
specify extension add --dev /caminho/para/spec-kit-dojo
```

Verificar:

```bash
specify extension list   # deve listar 'sybilion-dojo'
```

A partir daqui, os comandos `speckit.sybilion-dojo.init`, `speckit.sybilion-dojo.frame`, `speckit.sybilion-dojo.contract` e `speckit.sybilion-dojo.validate` ficam disponíveis no projecto destino, e os hooks disparam automaticamente nos pontos do ciclo do Spec-Kit definidos em [extension.yml](extension.yml).

## Desenvolver a extensão

Quando se itera neste repo, usar `--dev` para apontar a uma cópia local evita ter de publicar a cada alteração:

```bash
specify extension add --dev .
```
