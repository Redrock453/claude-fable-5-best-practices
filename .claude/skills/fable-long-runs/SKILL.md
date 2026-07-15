---
name: fable-long-runs
description: Применяй при проектировании долгих автономных запусков на claude-fable-5 (Claude Code, Managed Agents, собственные оркестраторы) — суб-агенты, самопроверка, память, task budgets, компакция, видимость прогресса.
---

# Долгие агентные запуски на Fable 5

Fable 5 в агентной обвязке работает часами и днями: планирует по этапам, делегирует, самопроверяется. Проектируй систему под **асинхронные check-in**, а не блокирующие вызовы (одиночный запрос — многие минуты).

## Постановка задачи

- Полная спецификация — в первом сообщении (`/goal` в Claude Code, Outcome с rubric в Managed Agents): контекст, ограничения, критерий «готово». Это максимизирует автономность и минимизирует токены на user-turn'ах.
- Перед стартом сложной работы: Plan mode + попроси модель расспросить о проекте, затем effort `high`/`xhigh`.
- Явно опиши, как должны выглядеть interim-апдейты — иначе модель либо молчит (thinking `omitted` = пауза в стриминге), либо перерассказывает.

## Суб-агенты — делегируй смело, но асинхронно

> Delegate independent subtasks to sub-agents and keep working while they run. Intervene if a sub-agent goes off track or is missing relevant context.

- Асинхронная коммуникация выигрывает у spawn-and-block: долгоживущие агенты сохраняют контекст (экономия cache-read), оркестратор не ждёт самого медленного.
- Суб-агентам ставь `effort: low/medium` — для Fable 5 это всё ещё очень сильный уровень.

## Самопроверка и честные статусы

- Явная self-verification в промпте: «Establish a method for checking your own work as you build; run it every [интервал], verifying against the specification with sub-agents». Свежеконтекстные агенты-верификаторы работают лучше самокритики.
- Аудит прогресса (почти убирает выдуманные статус-отчёты):
  > Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for.

## Память и доставка результатов

- Дай модели поверхность памяти — хотя бы `.md`-файлы:
  > Store one lesson per file with a one-line summary at the top. Record corrections and confirmed approaches alike. Don't save what the repo already records; update existing notes rather than duplicating; delete notes that turn out wrong.
- Для дословной доставки контента посреди запуска — `send_to_user`-инструмент: tool-inputs не суммаризуются, цифры/deliverable доходят без искажений.

## Бюджет и контекст

- Task Budgets (beta `task-budgets-2026-03-13`, `output_config.task_budget`, минимум `total: 20000`) — модель укладывается в бюджет грациозно вместо обрыва по `max_tokens`.
- Server-side compaction (beta `compact-2026-01-12`) — возвращай в историю **полный `response.content`**, компакшн-блоки должны сохраняться.
- Не показывай модели счётчик остатка контекста — провоцирует «context anxiety».

Детали и источники: `docs/04-agents-long-running.md`.
