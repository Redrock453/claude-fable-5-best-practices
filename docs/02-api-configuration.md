# 2. Настройка API: параметры, breaking changes, примеры

Основной источник: официальная документация Anthropic ([Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5), [Introducing Claude Fable 5 and Claude Mythos 5](https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5), [Models overview](https://platform.claude.com/docs/en/about-claude/models/overview)) и bundled-справка Claude API skill.

## 2.1 Breaking changes — что вернёт 400

| Что отправлено | Результат | Как правильно |
|---|---|---|
| `thinking: {type: "disabled"}` | 400 | Опустить параметр `thinking` целиком |
| `thinking: {type: "enabled", budget_tokens: N}` | 400 | `{type: "adaptive"}` или опустить; глубина — через `output_config.effort` |
| `temperature` ≠ 1.0 / `top_k` / `top_p` < 0.99 | 400 | Убрать; вариативность — через промпт |
| Assistant-prefill (последний ход assistant) | 400 | `output_config.format` (structured outputs) или инструкция в system |
| Запрос из организации с ZDR / retention < 30 дней | 400 на **каждый** запрос | Проверить конфигурацию retention организации до отладки payload |

## 2.2 Thinking

- **Всегда включён** (adaptive). Модель сама решает, когда и сколько думать, в т.ч. между tool-вызовами.
- **Сырая цепочка рассуждений никогда не возвращается.** Управление видимостью — `display`:
  - `"omitted"` (дефолт) — thinking-блоки приходят с пустым текстом (в стриминге выглядит как долгая пауза перед выводом);
  - `"summarized"` — читаемое резюме рассуждений.
- `display` влияет только на видимость: думает и биллится модель одинаково при любом значении.
- В multi-turn передавайте thinking-блоки назад **без изменений** (включая пустые). На других моделях блоки Fable молча выбрасываются из промпта (и не биллятся) — ничего вырезать не нужно.
- Запрос «покажи своё внутреннее рассуждение в ответе» может быть отклонён с `stop_details.category: "reasoning_extraction"` — читайте summarized thinking-блоки, а не просите модель пересказать reasoning.

```python
thinking={"type": "adaptive", "display": "summarized"}  # если рассуждения показываются пользователю
```

## 2.3 Effort — главный регулятор

`output_config: {effort: ...}` — основной контроль интеллект/латентность/стоимость ([Effort docs](https://platform.claude.com/docs/en/build-with-claude/effort)):

| Уровень | Когда |
|---|---|
| `max` | Корректность важнее цены; возможен overthinking и убывающая отдача |
| `xhigh` | Самые capability-чувствительные задачи: сложный кодинг, агенты |
| `high` | **Дефолт**, большинство задач |
| `medium` | Рутина, экономия — часто «золотая середина» |
| `low` | Суб-агенты, короткие/латентно-чувствительные задачи |

Ключевые наблюдения из доков и сообщества:

- **Низкие уровни Fable 5 часто превосходят `xhigh`/`max` прошлых моделей** — не бойтесь `medium`/`low` для рутины.
- Vague-промпт + высокий effort = модель «разрешает неопределённость длиной»: собирает лишний контекст, наводит порядок, которого не просили ([Learn With Me AI](https://www.learnwithmeai.com/p/how-to-prompt-fable-5): «effort задаёт, насколько широко модель смотрит, а не насколько умной становится»).
- Снижайте effort, если задача решается корректно, но занимает дольше необходимого.
- На `xhigh`/`max` ставьте большой `max_tokens` (от 64K) и используйте стриминг.

## 2.4 Refusals и fallbacks — включать по умолчанию

Fable 5 прогоняет запросы через классификаторы безопасности (offensive cyber, биология/life sciences, извлечение reasoning). Отказ приходит как **HTTP 200** с `stop_reason: "refusal"` и объектом `stop_details` ([Refusals and fallback](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback)). Частота отказов заметно выше, чем у прежних моделей; ложные срабатывания на безобидной security/bio-смежной работе — реальность (подтверждается опытом на [Habr](https://habr.com/ru/articles/1045824/)).

Практика:

1. **Всегда проверяйте `stop_reason` до чтения `content`** (может быть пустым или частичным).
2. **Server-side fallbacks** (Claude API, Claude Platform on AWS):

```python
response = client.beta.messages.create(
    model="claude-fable-5",
    max_tokens=16000,
    betas=["server-side-fallback-2026-06-01"],
    fallbacks=[{"model": "claude-opus-4-8"}],
    messages=[...],
)
for block in response.content:
    if block.type == "fallback":
        print(f"{block.from_.model} отказал; продолжил {block.to.model}")
```

3. На Bedrock/Vertex/Foundry server-side параметр недоступен — используйте клиентский `BetaRefusalFallbackMiddleware` + `BetaFallbackState` (одно состояние на диалог).
4. Fallback срабатывает **только на policy-declines**; rate limits и 5xx не фолбэчатся.
5. Sticky routing: после fallback диалог ~1 час обслуживается fallback-моделью напрямую.
6. Биллинг: отказ до вывода не биллится; mid-stream — биллится частичный вывод; rescue — по тарифам fallback-модели.
7. За отклонённые и перенаправленные запросы **тарифы Fable не списываются** — платите по тарифу модели, которая ответила.

В Claude Code reroute может сработать **на первом же сообщении** — потому что оно несёт CLAUDE.md и контекст репозитория: security-файлы или «подозрительные» имена триггерят классификатор по одному контексту. Диагностика: старт в safe-mode; отключение автопереключения — в `/config` ([MindStudio](https://www.mindstudio.ai/blog/how-to-prompt-claude-fable-5-anthropic-engineer-rules)).

## 2.5 Что работает без изменений

Совместимо с Opus-tier: Messages API и tool use, `output_config.effort`, Task Budgets (beta `task-budgets-2026-03-13`), compaction (beta `compact-2026-01-12`), memory tool, context editing, high-res vision (2576px), prompt caching, Batches, Files API.

## 2.6 Токены и тайминги

- Токенизатор = Opus 4.8: при миграции с Opus 4.7/4.8 счётчики почти не меняются; с Opus 4.6/Sonnet/Haiku — пересчитайте `count_tokens` (×1–1.35).
- **Одиночный запрос на сложной задаче может идти многие минуты** (15-минутный запрос — норма). Планируйте: стриминг, таймауты, прогресс-UX, асинхронные check-in вместо блокировки в одном запросе.
- Минимальный кэшируемый префикс Fable 5 — 2048 токенов.
