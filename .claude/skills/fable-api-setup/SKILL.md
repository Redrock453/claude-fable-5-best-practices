---
name: fable-api-setup
description: Применяй при написании или ревью кода, вызывающего claude-fable-5 через API (Anthropic SDK, Bedrock, Vertex, Foundry) — параметры запроса, thinking, effort, fallbacks, кэширование, breaking changes, обработка refusal.
---

# Настройка API для Claude Fable 5

Проверяй и применяй эти правила ко всякому коду, который вызывает `claude-fable-5` (`anthropic.claude-fable-5` / `us.anthropic.claude-fable-5` / `global.anthropic.claude-fable-5` на Bedrock).

## Параметры, которые вернут 400 — убрать

| Нельзя | Правильно |
|---|---|
| `thinking: {type: "disabled"}` | Опустить `thinking` целиком |
| `thinking: {type: "enabled", budget_tokens: N}` | `{type: "adaptive"}` или опустить; глубина — `output_config.effort` |
| `temperature` ≠ 1.0, `top_k`, `top_p` < 0.99 | Убрать; вариативность — через промпт |
| Assistant-prefill (последний ход assistant) | `output_config.format` (structured outputs) или инструкция в system |

Если организация под ZDR или retention < 30 дней — **каждый** запрос вернёт 400. Проверяй конфигурацию retention до отладки payload.

## Обязательный каркас запроса

```python
response = client.beta.messages.create(
    model="claude-fable-5",
    max_tokens=16000,                       # на xhigh/max — от 64K + стриминг
    output_config={"effort": "high"},       # low/medium/high(дефолт)/xhigh/max
    betas=["server-side-fallback-2026-06-01"],
    fallbacks=[{"model": "claude-opus-4-8"}],
    messages=[...],
)
if response.stop_reason == "refusal":
    ...  # вся цепочка отказала — обработать; content может быть пуст/частичен
```

1. **Всегда проверяй `stop_reason` до чтения `content`.** Отказ = HTTP 200 + `stop_reason: "refusal"` + `stop_details {category, explanation}`; категории: `cyber`, `bio`, `reasoning_extraction`, `frontier_llm`, `null`. Ложные срабатывания на легитимной security/bio-смежной работе — норма, поэтому fallback обязателен даже для «чистых» воркфлоу.
2. **Server-side fallback** доступен только на Claude API / Claude Platform on AWS. На Bedrock/Vertex/Foundry — клиентский `BetaRefusalFallbackMiddleware` + `BetaFallbackState` (одно состояние на диалог).
3. Fallback срабатывает только на policy-declines; 429 и 5xx не фолбэчатся. После fallback диалог ~1 час sticky на fallback-модели.
4. Mid-stream refusal: частичный вывод биллится — отбрасывай его, не считай ответом.

## Thinking

- Всегда включён (adaptive), отключить нельзя. Сырая цепочка не возвращается никогда.
- Видимость: `display: "omitted"` (дефолт, в стриминге выглядит как пауза) или `"summarized"` (читаемое резюме). На биллинг не влияет.
- В multi-turn возвращай thinking-блоки назад **без изменений**, включая пустые.
- Не проси модель «показать рассуждения» — refusal `reasoning_extraction`; читай summarized-блоки.

## Effort — главный регулятор

- `high` — дефолт; `xhigh` — сложный кодинг/агенты; `max` — только когда корректность важнее цены (риск overthinking).
- `medium`/`low` для рутины и суб-агентов — это всё ещё уровень `max` прошлых моделей.
- Vague-промпт + высокий effort = модель «разрешает неопределённость длиной». Если задача решается корректно, но долго — снижай effort.

## Кэширование и токены

- Prompt caching обязателен при $10/M входа (−90%). Порядок: `tools → system → messages`, волатильное — после последнего breakpoint. Минимальный кэшируемый префикс — **2048 токенов**.
- Токенизатор = Opus 4.8. Миграция с Opus 4.6/Sonnet/Haiku — пересчитай `count_tokens` (×1–1.35).
- Одиночный запрос может идти 15+ минут: стриминг, таймауты, прогресс-UX, асинхронные check-in.
- Долгие агентные циклы: Task Budgets (`task-budgets-2026-03-13`, `output_config.task_budget`, минимум 20 000) вместо обрыва по `max_tokens`; compaction (`compact-2026-01-12`) — возвращай в историю полный `response.content`.

Детали и источники: `docs/02-api-configuration.md`, `docs/04-agents-long-running.md`.
