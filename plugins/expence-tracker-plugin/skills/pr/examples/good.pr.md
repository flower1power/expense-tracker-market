# Пример хорошего pull request.

## Заголовок

feat(transactions): add summary endpoint by category

## Описание

## Что сделано

- Добавлен GET /transactions/summary эндпоинт
- Агрегация сумм по categoryId за указанный месяц
- Отдельно считается INCOME и EXPENSE

## Причина изменений

Дашборд требует данные для графики расходов по категориям.
Без этого эндпоинта фронтенд делал бы N запросов.

## Как тестировать

1. Запустить `docker-compose up -d && pnpm dev:web`
2. Авторизоваться POST login
3. Запросить: GET /transactions/summary?month=1&year=2025
4. Ожидаемый ответ:
    \`\`\`json
    [{"categoryId":"uuid", "categoryName": "Продукты", "totalIncome": 0, "totalExpense":15000}]
    \`\`\`

## Затронутые модули

- Backend: transactions.service.ts, transactions.controller.ts
- База данных: нет изменений схемы

## Чеклист

- [x] Typecheck пройден
- [x] Unit тесты написаны
- [x] CLAUDE.md актуален