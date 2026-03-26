# GitHub Actions CI/CD Templates

Набор переиспользуемых GitHub Actions воркфлоу для frontend-экосистемы `shoshin-packages`.

## Описание

- Реализованы как [Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) — подключаются одной строкой
- Composite Action `npm-setup` инкапсулирует общую логику: Node.js, git, npm registry, `npm ci`
- Кешируются npm-пакеты через `actions/setup-node` (ключ по хэшу `package-lock.json`)
- npm-реестр: [GitHub Packages](https://github.com/features/packages) (`@shoshin-packages` scope)
- PR-превью документации через [pr-preview-action](https://github.com/rossjrw/pr-preview-action)

## Структура

```text
.github/
├── actions/
│   └── npm-setup/           # composite action: Node.js + git + npm registry + npm ci
│
└── workflows/
    ├── core-lint.yml        # ESLint + Stylelint + TypeScript (параллельно)
    ├── core-test.yml        # Vitest + JUnit отчёт + GitHub Check Run
    ├── library-build.yml    # сборка npm-библиотеки → артефакт dist/
    ├── library-publish.yml  # semantic-release → GitHub Packages + GitHub Release
    ├── docs-build.yml       # VitePress сборка → артефакт
    └── docs-publish.yml     # GitHub Pages (main) + PR превью
```

## Требования

### Organization Secrets (`shoshin-packages`)

| Секрет | Описание |
|--------|----------|
| *(не требуется)* | Используется встроенный `GITHUB_TOKEN` |

Для `library/publish.yml` передавать `secrets.GITHUB_TOKEN` с правами `contents: write` и `packages: write`.

## Использование

```yaml
# .github/workflows/ci.yml в вашем проекте
name: CI

on:
  push:
    branches: [main, dev]
  pull_request:

jobs:
  lint:
    uses: shoshin-packages/ci/.github/workflows/core-lint.yml@main
    secrets: inherit

  test:
    uses: shoshin-packages/ci/.github/workflows/core-test.yml@main
    secrets: inherit

  build:
    uses: shoshin-packages/ci/.github/workflows/library-build.yml@main
    secrets: inherit

  publish:
    needs: build
    uses: shoshin-packages/ci/.github/workflows/library-publish.yml@main
    with:
      build-artifact-name: ${{ needs.build.outputs.artifact-name }}
    secrets: inherit
```
