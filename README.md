# Sybilion Dojo Preset

Preset oficial da **Sybilion** para o [Spec-Kit](https://github.com/github/spec-kit).

Este repositório **é o preset** — não é uma aplicação. É consumido por outros repositórios através da CLI do Spec-Kit (`specify`), que aplica os comandos, templates e hooks aqui definidos ao projecto destino.

## O que define

- **Constitution** corporativa com os engineering standards da Sybilion (linting, tipos estritos, logger interno, versões de tooling).
- Comando `speckit.sybilion.contract` — gera contratos blackbox Dojo a partir do `plan.md`.
- Comando `speckit.sybilion.validate` — corre linters e testes Dojo até passarem.
- **Hooks** que ligam estes comandos ao ciclo de vida do Spec-Kit (`after_plan` → contract, `after_implement` → validate).

## Estrutura

```
.
├── preset.yml                          # Manifesto do preset
├── commands/
│   ├── contract.md                     # /speckit.sybilion.contract
│   └── validate.md                     # /speckit.sybilion.validate
└── templates/
    └── constitution-template.md        # Constitution Sybilion (estratégia: replace)
```

## Requisitos

- `specify` CLI do Spec-Kit (versão >= 0.8.0).
- `dojo` disponível no PATH (`go install github.com/elmacnifico/dojo@latest`) — necessário em runtime para o comando de validação.

## Instalação no repositório destino

A partir do repositório onde se quer aplicar o preset:

```bash
# Via referência ao Git remoto (recomendado para a equipa)
specify preset add <git-url-deste-repo>

# Ou apontando para um clone local (modo dev, para iterar no preset)
specify preset add --dev /caminho/para/spec-kit-dojo
```

Verificar:

```bash
specify preset list   # deve listar 'sybilion-dojo'
```

A partir daqui, os comandos `speckit.sybilion.contract` e `speckit.sybilion.validate` ficam disponíveis no projecto destino, e os hooks disparam automaticamente nos pontos do ciclo do Spec-Kit definidos em [preset.yml](preset.yml).

## Desenvolver o preset

Quando se itera neste repo, usar `--dev` para apontar a uma cópia local evita ter de publicar a cada alteração:

```bash
specify preset add --dev .
```
