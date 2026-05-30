---
name: unit-test
description: Сгенерировать юнит-тесты для указанного файла — /unit-test <path-to-file>
user_invocable: true
arguments_hint: <path-to-file>
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash(pnpm *)
  - Bash(ls *)
  - Bash(cat *)
  - Bash(find *)
model: claude-sonnet-4-6
effort: medium
---

# Unit Test Skill
Сгенерируй юнит-тесты для файла, путь которого передан аргументом, придерживаясь стека и стиля проекта.

## Аргументы
- $0 — относительный или абсолютный путь к файлу с исходным кодом (`apps/api/src/...`, `apps/web/src/...`, `packages/shared/...`)

## Алгоритм

1. Если $0 не передан — прервать и попросить путь
2. Проверить существование файла; если нет — прервать с сообщением
3. Определить контекст:
   - `apps/api/**` → NestJS, фреймворк **Jest** + `@nestjs/testing`
   - `apps/web/**` → React/Next.js, фреймворк **Vitest** + `@testing-library/react`
   - `packages/shared/**` → чистый TS, фреймворк **Vitest**
4. Прочитать файл и понять:
   - что экспортируется (функции, классы, компоненты, хуки)
   - внешние зависимости (нужны моки)
   - публичный API и крайние случаи
5. Проверить наличие соседних `*.spec.ts` / `*.test.ts(x)` для соблюдения существующего стиля
6. Если в `package.json` соответствующего пакета нет тестового фреймворка — **не ставить** автоматически, а в конце вывести команду установки и подсказку по `scripts.test`
7. Сгенерировать файл с тестами рядом с исходным:
   - `foo.ts` → `foo.spec.ts` (api, shared)
   - `Foo.tsx` → `Foo.test.tsx` (web)
8. Покрыть:
   - happy path для каждой публичной функции/метода
   - граничные случаи (пустые значения, `null`/`undefined`, пустые массивы)
   - ошибочные ветки (исключения, отказы зависимостей)
   - для NestJS-сервисов — мокать репозитории/Prisma через `jest.fn()`
   - для React-компонентов — рендер, основные взаимодействия, состояния пропсов
9. Запустить тесты, если в проекте уже настроен `pnpm --filter <pkg> test`
10. Вывести краткий отчёт: путь созданного файла, число кейсов, результат запуска (если был)

## Шаблоны

### NestJS-сервис (Jest)
```ts
import { Test, TestingModule } from '@nestjs/testing';
import { FooService } from './foo.service';

describe('FooService', () => {
  let service: FooService;
  const depMock = { findOne: jest.fn() };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [FooService, { provide: DepToken, useValue: depMock }],
    }).compile();

    service = module.get(FooService);
  });

  afterEach(() => jest.clearAllMocks());

  it('возвращает результат при валидном вводе', async () => { /* ... */ });
  it('бросает NotFoundException, если запись не найдена', async () => { /* ... */ });
});
```

### React-компонент (Vitest + Testing Library)
```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { Foo } from './Foo';

describe('<Foo />', () => {
  it('рендерит заголовок из props', () => {
    render(<Foo title="Привет" />);
    expect(screen.getByText('Привет')).toBeInTheDocument();
  });

  it('вызывает onClick при клике', async () => {
    const onClick = vi.fn();
    render(<Foo title="x" onClick={onClick} />);
    await userEvent.click(screen.getByRole('button'));
    expect(onClick).toHaveBeenCalledOnce();
  });
});
```

### Чистая TS-функция (Vitest)
```ts
import { describe, it, expect } from 'vitest';
import { sum } from './sum';

describe('sum', () => {
  it('складывает два числа', () => expect(sum(2, 3)).toBe(5));
  it('работает с нулём', () => expect(sum(0, 0)).toBe(0));
});
```

## Правила

- Названия `it/describe` — на русском, формулируя ожидаемое поведение, а не реализацию
- Один кейс — одно утверждение по смыслу (несколько `expect` допустимо, если описывают одно поведение)
- Не тестировать сторонние библиотеки и тривиальные геттеры
- AAA-структура (Arrange / Act / Assert), без комментариев-разделителей
- Все внешние зависимости — моки; никаких реальных HTTP/БД-вызовов
- Не менять исходный файл; если код невозможно протестировать без рефакторинга — отметить это в отчёте

## Обработка ошибок

| Ситуация | Действие |
|---|---|
| Аргумент $0 не передан | Прервать, попросить путь |
| Файл не существует | Прервать, сообщить путь |
| Файл — `index.ts` без логики (реэкспорт) | Сообщить, что тестировать нечего |
| Тестовый фреймворк не установлен | Сгенерировать файл, вывести команды установки и не запускать тесты |
| Соседний тест уже существует | Спросить: дополнить, заменить или прервать |