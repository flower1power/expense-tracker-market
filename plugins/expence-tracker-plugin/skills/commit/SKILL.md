---
name: commit
description: Создать коммит по соглашению проекта с описанием на русском
user_invocable: true
allowed-tools:
  - Bash(git *)
  - Read
  - Glob
  - Grep
model: claude-sonnet-4-6
effort: low
---

## Правила коммитов

Формат: `<type>(<scope>): <описание>`

Типы: `feat` | `fix` | `docs` | `style` | `refactor` | `test` | `chore`
Scope: `api` | `web` | `shared` | *(без scope — корень или несколько пакетов)*
Описание: кратко на русском, первая буква строчная, без точки в конце
Breaking changes: `!` перед двоеточием — `feat(api)!: изменён формат ответа`

## Примеры

```
feat(web): добавить страницу дашборда
fix(api): исправить валидацию JWT токена
refactor(shared): переименовать типы транзакций
docs: обновить README с инструкцией по запуску
```

## Контекст выполнения
Статус проекта !`git status`
Последние коммиты: !git log --oneline -10

## Алгоритм выполнения
1. `git diff` — проверить контент для сообщения
2. Определить `type` и `scope` по затронутым файлам
3. `git add` — добавить только нужные файлы (не `git add .`)
4. Сформировать сообщение по правилам выше
5. Создать коммит через heredoc
6. Проверить результат: git log --oneline -1 

```bash
git commit -m "$(cat <<'EOF'
<type>(<scope>): <описание>

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```

7. `git status` — проверить результат

## Запрещено

- Никогда не пушить автоматически
- Не использовать `--no-verify` или `--amend` без явной просьбы
- Не добавлять файлы без понимания их содержимого
- Не коммить файлы с секретами (`.env`, credentials)
