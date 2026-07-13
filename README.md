# Claude Fable 5 — лучшие практики настройки и использования

Глубокое исследование по модели **Claude Fable 5** (`claude-fable-5`) от Anthropic — первой публично доступной модели класса **Mythos**, стоящей выше линейки Opus. Собрано из официальной документации Anthropic, Habr, Reddit-сообщества, независимых обзоров и технических блогов (июнь–июль 2026).

## Ключевые факты в одном абзаце

Fable 5 — модель для самых сложных, длинных и амбициозных задач: автономные запуски на часы и дни, end-to-end работа уровня «человеко-недель». Контекст **1M токенов**, вывод до **128K**. Цена **$10 / $50 за 1M** входных/выходных токенов (в 2 раза дороже Opus 4.8). Thinking всегда включён (adaptive, отключить нельзя), глубина управляется `output_config.effort` (`low`…`max`). Sampling-параметры (`temperature`/`top_k`) убраны. Есть классификаторы безопасности (кибер/био) с fallback на Opus 4.8. Обязательное хранение данных 30 дней (ZDR несовместим).

## Структура репозитория

| Файл | Содержание |
|---|---|
| [docs/01-overview.md](docs/01-overview.md) | Что такое Fable 5 и Mythos-класс, доступность, спецификации |
| [docs/02-api-configuration.md](docs/02-api-configuration.md) | Настройка API: thinking, effort, fallbacks, breaking changes, примеры кода |
| [docs/03-prompting.md](docs/03-prompting.md) | Промптинг: 6 правил, готовые сниппеты, анти-паттерны |
| [docs/04-agents-long-running.md](docs/04-agents-long-running.md) | Долгие агентные запуски, Claude Code, sub-агенты, память |
| [docs/05-community-insights.md](docs/05-community-insights.md) | Опыт сообщества: Habr, Reddit, независимые обзоры |
| [docs/06-cost-limits-safety.md](docs/06-cost-limits-safety.md) | Экономика, лимиты, классификаторы, приватность |
| [docs/07-noise-handling.md](docs/07-noise-handling.md) | Борьба с шумом: тощий CLAUDE.md, промпты, вывод — с примерами «до/после» |
| [templates/CLAUDE.md.template](templates/CLAUDE.md.template) | Готовый скелет «тощего» CLAUDE.md для копирования в свои проекты |
| [SOURCES.md](SOURCES.md) | Полный список источников |

## TL;DR — 10 главных практик

1. **Давайте цель, а не пошаговый скрипт.** Старые промпты «для слабых моделей» (нумерованные правила, act-as-expert) *снижают* качество Fable 5.
2. **Указывайте effort осознанно:** `high` — дефолт, `xhigh` — сложный кодинг/агенты, `medium/low` — рутина (и это всё ещё уровень `max` прошлых моделей).
3. **Ограничивайте over-delivery:** «Сделай простейшее работающее решение, ничего сверх задачи».
4. **Весь контекст — сразу, одним хорошо специфицированным первым сообщением.** Кусочная подача снижает эффективность.
5. **На длинных запусках требуйте аудит прогресса по tool-результатам** — почти убирает выдуманные статус-отчёты.
6. **Включайте fallback на Opus 4.8 по умолчанию** (`fallbacks` + beta `server-side-fallback-2026-06-01`) и обрабатывайте `stop_reason: "refusal"`.
7. **Не отправляйте `thinking: {type:"disabled"}` и `budget_tokens`** — это 400. Thinking всегда включён; параметр либо опускать, либо `{type:"adaptive"}`.
8. **Кэшируйте промпты** (до 90% экономии входа) — при цене $10/M это критично.
9. **CLAUDE.md — тощий.** Fable 5 сама сканирует проект; каждая лишняя строка — шум.
10. **Давайте самые сложные задачи.** Тестировать Fable 5 на простых воркфлоу — недооценивать её; на коротких задачах выгоднее Opus 4.8 / Sonnet 5.

## Быстрый старт (Python)

```python
import anthropic

client = anthropic.Anthropic()

response = client.beta.messages.create(
    model="claude-fable-5",
    max_tokens=16000,
    output_config={"effort": "high"},          # thinking всегда on; глубина — через effort
    betas=["server-side-fallback-2026-06-01"],  # fallback при срабатывании классификатора
    fallbacks=[{"model": "claude-opus-4-8"}],
    messages=[{"role": "user", "content": "..."}],
)

if response.stop_reason == "refusal":
    ...  # вся цепочка отказала — обработать
```

---

*Исследование собрано автоматически (Vika / Claude Fable 5), июль 2026. Все утверждения снабжены ссылками на источники в соответствующих разделах.*
