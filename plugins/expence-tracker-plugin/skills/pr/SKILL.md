---
name: pr
description: Создать Pull Request на GitHub с заданым названием и веткой
model: sonnet
allowed-tools: Bash(git *), Bash(gh *), Bash(bash *validate.sh*)
user_invocable: true
argument_hint: <title>, <base-branch default main>
---

# PR Skill
Создай pull request на GitHub, соблюдая соглашение проекта.

## Аргументы
- $0 - Название pull request по Conventional Commit (например `feat(web): добавить авторизацию`)
- $1 - Целевая ветка (по умолчанию `main`)

## Подготовка
1. Проверить что ветка готова:
   !`bash .claude/skills/pr/scripts/validate.sh`
2. Получить diff от базовой ветки:
   !`git diff main..HEAD`
3. Получить список коммитов:
   !`git log main..HEAD --oneline`

## Задача
Используя данные выше заполни шаблон из @template.md
Посмотри пример хорошего PR: @examples/good.pr.md

## Создание PR
Создай PR командой:
gh pr create \
   --title "$0 или сгенерированный title" \
   --body "заполненный шаблон" \
   --base "${ARGUMENTS:-main}"

## Правила
- Заголовок по Conventional Commit
- Если ветка не запушена:
  git push --set-upstream origin HEAD