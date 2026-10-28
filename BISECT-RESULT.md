# Результат git bisect — демо 9.1

**Дата:** 2026-10-28

## Завдання

У теці демо-кіта виконати `make sandbox`, потім `make test` (тест червоний на поточному стані). Запустити `git bisect start`, позначити поточний стан як `git bisect bad`, тег `good-baseline` — як `git bisect good`, на кожному коміті-кандидаті проганяти тест і казати `good` / `bad`. Зафіксувати, на який коміт вийшов bisect.

## Результат

**Перший поганий коміт:** `b9f653d` — `refactor(api): simplify pagination cursor handling`

> Хеші комітів у `sandbox/` залежать від часу запуску `setup.sh`, тому після `make reset` вони будуть іншими. Стабільний орієнтир — subject коміту.

## Хід bisect

| Крок | Коміт | `make test` | Позначка |
|---|---|---|---|
| старт | `e196a48 chore(auth): tidy refresh logging` (HEAD) | червоний | `bad` |
| старт | `907d622 test(api): cover pagination` (`good-baseline`) | зелений | `good` |
| 1 | `cd024c3 feat(auth): add token refresh on expiry` | зелений | `good` |
| 2 | `2e93a68 docs: usage notes` | червоний | `bad` |
| 3 | `b9f653d refactor(api): simplify pagination cursor handling` | червоний | `bad` |

Git: `b9f653d… is the first bad commit`. Після цього — `git bisect reset`.

## Причина

Коміт прибрав гілку для порожнього курсора в `page()` (`src/api/pagination.py`):

```diff
-    items = query(cursor) if cursor else query_first()
+    items = query(cursor)
```

`page()` без курсора тепер викликає `query(None)`, і порівняння `row["id"] > cursor` падає:

```
TypeError: '>' not supported between instances of 'int' and 'NoneType'
```

Тобто «рефакторинг» змінив поведінку — класичний випадок для bisect.

## Примітки

- `make` на машині відсутній, тому замість `make test` запускалось його тіло: `cd sandbox/src && python -m unittest tests.test_pagination`; замість `make sandbox` — `bash setup.sh`.
- Баг не виправлявся — завдання лише знайти коміт.
