---
name: fable-prompting
description: Применяй при написании, ревью или миграции промптов (system prompt, user-инструкции, CLAUDE.md, шаблоны агентов) для claude-fable-5 — правила де-прескриптизации, готовые сниппеты, анти-паттерны.
---

# Промптинг Claude Fable 5

## Главный сдвиг

Скаффолдинг для слабых моделей — пошаговые инструкции, нумерованные списки правил, «act as an expert» — Fable 5 **мешает**. Давай направление, контекст, цели и критерии проверки, а не перечисление шагов. При миграции промпта со старой модели: сними старый скаффолдинг и предложи A/B-сравнение.

## Шесть правил (применять к каждому промпту)

1. **Явный объём/effort в самом запросе** — модель не знает, нужен ответ на два предложения или анализ на 2000 слов; неопределённость она разрешает удлинением.
2. **Ограничение over-delivery** — на высоких effort модель добавляет непрошеное. Сниппет:
   > Don't add features, refactor, or introduce abstractions beyond what the task requires. Do the simplest thing that works well. Only validate at system boundaries.
3. **Аудит прогресса на длинных запусках:**
   > Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly.
4. **Явные границы действий:**
   > When the user is describing a problem or thinking out loud, the deliverable is your assessment. Report findings and stop. Don't apply a fix until they ask.
5. **Multishot-примеры** — показывай формат 3–5 примерами, а не описанием; дальше убывающая отдача.
6. **Причина, а не только просьба:**
   > I'm working on [большая задача] for [для кого]. They need [что даёт результат]. With that in mind: [запрос].

## Фронтлоад контекста

Весь контекст, ограничения и критерий «готово» — одним хорошо специфицированным первым сообщением. Кусочная подача заставляет модель оптимизировать под фрагмент и откатываться — и жжёт токены.

## Стиль вывода (если не задан)

> Lead with the outcome. Your first sentence should answer "what happened" — the TLDR. Supporting detail after. Keep output short by being selective about what you include, not by compressing writing into fragments or arrow chains.

Для длинных сессий добавляй: финальное резюме — для читателя, который не видел процесса; полные предложения, без выдуманных по ходу сокращений.

## Известные сбои и готовые обходы

| Симптом | Вставка в промпт |
|---|---|
| «Сейчас запущу X» без tool-вызова, ранняя остановка | «You are operating autonomously… Before ending your turn, check your last paragraph. If it is a plan or a promise, do that work now with tool calls.» |
| «Тревога о контексте», предлагает новую сессию | Не показывать счётчик остатка контекста; при необходимости: «You have ample context remaining. Do not stop on account of context limits.» |
| Overplanning на неоднозначных задачах | «When you have enough information to act, act. Give a recommendation, not an exhaustive survey.» |

## Анти-паттерны — находи и убирай

- ❌ Пошаговые чек-листы того, что модель выведет сама.
- ❌ «CRITICAL: YOU MUST…» — overtriggering, Fable следует буквально.
- ❌ Просьба «показать рассуждения» в ответе — refusal `reasoning_extraction`.
- ❌ Дублирование в CLAUDE.md того, что модель узнает сканированием проекта. CLAUDE.md — тощий; «security-подобный» контент в нём может триггернуть классификатор на первом же сообщении.
- ❌ Тестирование только на простых задачах — отрыв Fable 5 виден на сложных.

Детали и источники: `docs/03-prompting.md`.
