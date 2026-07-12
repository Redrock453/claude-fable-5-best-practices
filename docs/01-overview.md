# 1. Обзор: что такое Claude Fable 5

## Mythos-класс

Fable 5 — **не** следующая Opus под новым именем. Anthropic позиционирует её как первый публично доступный представитель нового яруса **Mythos-class**, стоящего *над* классом Opus ([Anthropic](https://www.anthropic.com/claude/fable), [Caylent](https://caylent.com/blog/claude-fable-5-anthropics-first-public-mythos-class-model)).

Пара моделей:

| Модель | ID | Доступ |
|---|---|---|
| Claude Fable 5 | `claude-fable-5` | Общедоступна (API, Bedrock, Vertex, Foundry) |
| Claude Mythos 5 | `claude-mythos-5` | Только Project Glasswing (по приглашению) |

Это **одна и та же базовая модель** — имена различают уровень защитных механизмов, а не веса ([модельный обзор](https://platform.claude.com/docs/en/about-claude/models/overview)). Fable снабжена блокирующими классификаторами; Mythos — «расширенный» вариант для одобренных клиентов.

## Спецификации

| Параметр | Значение |
|---|---|
| Контекст | 1M токенов (дефолт = максимум) |
| Максимальный вывод | 128K токенов на запрос |
| Thinking | Adaptive, **всегда включён**, отключить нельзя |
| Effort | `low` / `medium` / `high` (дефолт) / `xhigh` / `max` |
| Sampling | `temperature` = 1.0 или не задан; `top_p` ≥ 0.99; `top_k` не поддерживается |
| Токенизатор | Тот же, что у Opus 4.8 (введён с Opus 4.7) |
| Цена | $10 / $50 за 1M input/output; кэш входа −90%; US-only inference ×1.1 |
| Retention | Обязательные 30 дней (ZDR → 400 на каждый запрос) |
| GA | 9 июня 2026: Claude API, Claude Platform on AWS, Amazon Bedrock, Google Cloud, Microsoft Foundry |

Источники: [AWS Bedrock model card](https://docs.aws.amazon.com/bedrock/latest/userguide/model-card-anthropic-claude-fable-5.html), [Cloudflare AI docs](https://developers.cloudflare.com/ai/models/anthropic/claude-fable-5/), [models.dev](https://models.dev/models/anthropic/claude-fable-5/), [анонс Anthropic](https://www.anthropic.com/news/claude-fable-5-mythos-5).

На Bedrock ID модели: `anthropic.claude-fable-5`, geo-inference `us.anthropic.claude-fable-5`, global `global.anthropic.claude-fable-5`.

## Для чего она

Официальная позиция: Fable 5 особенно эффективна на **end-to-end работе, которая занимает у человека часы, дни или недели** — планирование по этапам, делегирование суб-агентам, самопроверка. В агентной обвязке (Claude Code, Managed Agents) может работать **днями** ([Introducing Claude Fable 5](https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5)).

Практическое правило выбора модели из ранних гайдов: **если задача заняла бы у человека две недели и больше — она «заслуживает» Fable 5; быстрые правки — Sonnet 5 / Opus 4.8** ([Linas's Newsletter](https://linas.substack.com/p/claude-fable-5-guide)). Разбор на Хабре формулирует то же иначе: главное улучшение — не отдельная способность, а **устойчивость на длинных сложных задачах** ([Habr: цена, лимиты, границы задач](https://habr.com/ru/articles/1046736/)).

## Чем сильнее предыдущих моделей (по документации)

- Долгие автономные запуски и first-shot реализация хорошо специфицированных систем;
- Корпоративные deliverables целиком: финанализ, таблицы, слайды, документы;
- Code review / debugging и поиск по истории репозитория;
- Vision на плотных/испорченных изображениях (обучена крутить/кропать через bash);
- Навигация в неопределённости, параллельная делегация суб-агентам и длительная коммуникация с ними.

**Важно:** не оценивайте Fable 5 только на задачах, которые уже решали старые модели — её отрыв растёт с длиной и сложностью задачи.
