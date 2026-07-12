# 3. Промптинг Fable 5: правила и готовые сниппеты

Источники: [официальный prompting-гайд](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5), разборы правил Anthropic от [MindStudio](https://www.mindstudio.ai/blog/how-to-prompt-claude-fable-5-anthropic-engineer-rules) ([вторая часть](https://www.mindstudio.ai/blog/how-to-prompt-claude-fable-5-six-rules-anthropic)), [AI Blew My Mind](https://aiblewmymind.substack.com/p/how-to-use-claude-fable-5), [AI Made Simple](https://aimadesimple0.substack.com/p/how-to-use-claude-fable-5-better), [Learn With Me AI](https://www.learnwithmeai.com/p/how-to-prompt-fable-5).

## 3.1 Главный сдвиг: разучивайтесь микроменеджить

Всё, что вырабатывалось для слабых моделей — пошаговые инструкции, нумерованные списки правил, «act as an expert»-преамбулы — было нужно, чтобы удерживать модель на рельсах. Fable 5 это **мешает**: официальная рекомендация — давать **направление, контекст, цели и критерии проверки**, а не перечисление шагов. После миграции A/B-тестируйте промпты со снятым старым скаффолдингом — избыточная прескриптивность *снижает* качество.

## 3.2 Шесть рабочих правил

1. **Явный effort-уровень в самом запросе.** Модель не знает, нужен ответ на два предложения или анализ на 2000 слов. Неопределённость она разрешает удлинением.
2. **Ограничивайте over-delivery.** На высоких effort модель добавляет непрошеные фичи и рефакторит рабочее. Сниппет:
   > Don't add features, refactor, or introduce abstractions beyond what the task requires. Do the simplest thing that works well. Only validate at system boundaries.
3. **Аудит длинных запусков.** Требование сверять заявления о прогрессе с tool-результатами почти полностью убирает выдуманные статус-отчёты:
   > Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly.
4. **Явные границы действий.** Fable 5 иногда делает смежные непрошеные шаги (черновик письма, backup-ветки git):
   > When the user is describing a problem or thinking out loud, the deliverable is your assessment. Report findings and stop. Don't apply a fix until they ask.
5. **Multishot-примеры.** Показывайте формат, а не описывайте его: 3–5 примеров, дальше — убывающая отдача.
6. **Причина, а не только просьба.** Модель работает лучше, когда понимает намерение:
   > I'm working on [большая задача] for [для кого]. They need [что даёт результат]. With that in mind: [запрос].

## 3.3 Фронтлоад контекста

Давайте всё нужное **сразу**: полный контекст, ограничения, критерий «готово» — одним хорошо специфицированным первым сообщением. Кусочная подача заставляет модель оптимизировать под фрагмент и потом откатываться. Это же правило — ключ к экономии токенов в интерактивных сессиях.

## 3.4 Стиль вывода

Ненастроенная Fable 5 на высоких effort склонна к плотным структурам, разделам «альтернативы, которые не выбраны», arrow-chain-стенографии. Лечится короткой инструкцией:

> Lead with the outcome. Your first sentence should answer "what happened" — the TLDR. Supporting detail after. Keep output short by being selective about what you include, not by compressing writing into fragments or arrow chains.

Для длинных агентных сессий добавляйте требование «финальное резюме — для читателя, который не видел процесса»: полные предложения, без выдуманных в ходе работы сокращений.

## 3.5 Редкие сбои и их обход

| Симптом | Обход |
|---|---|
| Ранняя остановка: «Сейчас запущу X» без tool-вызова | System-reminder: «You are operating autonomously… Before ending your turn, check your last paragraph. If it is a plan or a promise, do that work now with tool calls.» |
| «Тревога о контексте» — предлагает новую сессию | Не показывать модели счётчик остатка контекста; при необходимости: «You have ample context remaining. Do not stop on account of context limits.» |
| Overplanning на неоднозначных задачах | «When you have enough information to act, act. Give a recommendation, not an exhaustive survey.» |

## 3.6 Анти-паттерны

- ❌ Пошаговые чек-листы для того, что модель выведет сама.
- ❌ «CRITICAL: YOU MUST…» — вызывает overtriggering; Fable следует инструкциям буквально.
- ❌ Тесты только на простых задачах — отрыв Fable 5 виден на сложных.
- ❌ Просить модель «показать рассуждения» в ответе — риск refusal `reasoning_extraction`.
- ❌ Дублировать в CLAUDE.md то, что модель узнает сканированием проекта.
