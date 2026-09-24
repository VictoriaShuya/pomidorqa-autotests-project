# PomidorQA — автотесты (портфолио)

Автотесты на [PomidorQA](https://aiqa.su/pomidorqa) — сервис коротких (25 мин) встреч для
QA/IT-специалистов. Продукт для набора — чёрный ящик: исходников приложения нет, проверяем
живое поведение на `aiqa.su/pomidorqa`.

**Что нельзя сломать:** регистрация и вход, профиль и навыки, попадание в каталог по свободному
слоту, поиск по навыку, бронирование (включая гонку за слот), отмена встречи, списки
«Мои встречи».

## Покрытие требований

Считаем от спецификации, не от числа тестов:

- [requirements.md](./requirements.md) — 50 функциональных требований MVP (R3.1–R12.3)
- [docs/coverage-matrix.md](./docs/coverage-matrix.md) — матрица «требование → статус → тесты»

**Срез:** **21 из 50** требований в статусе `automated` (**42%**). Остальное: `partial` (9),
`out of scope` (2), `not covered` (18). Подробности и обоснования — в матрице.

## Уровни и архитектура

| Уровень | Тестов | Что проверяет |
|---|---|---|
| unit | 10 | чистые функции в `src/pyramid` (пересечение слотов, пароль, формат времени) |
| api | 8 | правила брони и регистрации на локальном мок-сервере |
| e2e | 14 | сценарии в браузере на живом стенде |
| **всего** | **32** | |

- `tests/pages/` — Page Object: локаторы и действия экрана
- `tests/helpers/` — тестовые данные, регистрация (UI/API), подготовка каталога
- unit/api проверяют **учебный** код набора; продуктовые правила, которые видит пользователь —
  в E2E

## Установка

```bash
npm install
npx playwright install chromium
```

## Запуск

```bash
npm run test:unit   # Unit — без сети и без браузера
npm run test:api    # API — HTTP к локальному мок-серверу
npm run test:e2e    # E2E — браузер на aiqa.su/pomidorqa
npm test            # все три уровня
npm run report      # HTML-отчёт последнего прогона
```

Один вход для локальной и CI-логики — от дешёвой проверки к дорогой:

```bash
npm run gate        # typecheck → lint → unit → api → e2e
```

Замер локального `npm run gate` (24.09.2026): **32 passed за 31.1 с**
(typecheck + lint + 10 unit + 8 api + 14 e2e).

По умолчанию E2E бьёт в `https://aiqa.su`. Локальный стенд:

```bash
POMIDORQA_BASE_URL=http://localhost:3000 npx playwright test --project=e2e
```

## CI

GitHub Actions (`.github/workflows/playwright.yml`):

- триггеры: `pull_request` → `main`, `push` → `main`, ручной `workflow_dispatch`
- порядок: `typecheck` → `lint` → установка Chromium → `npm test`
- артефакт: HTML-отчёт Playwright

Красная статика не поднимает браузер: typecheck и lint идут до `playwright install`.

## Структура

```
src/pyramid/       — чистые функции (unit) и локальный мок-сервер (api)
tests/unit/        — пересечение слотов, формат времени, валидация пароля
tests/api/         — бронь, гонка, регистрация на моке
tests/e2e/         — логин-ошибка, профиль, поиск, бронь, отмена
tests/helpers/     — данные, регистрация, подготовка каталога
tests/pages/       — Page Object
docs/              — матрица покрытия
requirements.md    — спецификация MVP (50 требований)
```
