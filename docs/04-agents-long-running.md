# 4. Долгие агентные запуски, Claude Code, суб-агенты

Источники: [официальный prompting-гайд](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5), [Introducing Claude Fable 5](https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5), гайды [Linas](https://linas.substack.com/p/claude-fable-5-guide), [MindStudio](https://www.mindstudio.ai/blog/how-to-prompt-claude-fable-5-anthropic-engineer-rules), [awesome-claude-fable-5](https://github.com/Anil-matcha/awesome-claude-fable-5).

## 4.1 Модель поведения

В агентной обвязке (Claude Code, Managed Agents) Fable 5 может работать **днями**: планирует по этапам, делегирует суб-агентам, проверяет собственную работу. Один запрос на сложной задаче — многие минуты; автономный запуск — часы. Проектируйте систему под **асинхронные check-in**, а не блокирующие вызовы.

## 4.2 Claude Code: практики

- **CLAUDE.md — тощий.** Модель сама сканирует проект; каждая лишняя строка — шум, а «security-подобный» контент может триггернуть классификатор прямо на первом сообщении.
- **Advanced Plan mode + вопросы перед кодом.** Просите модель расспросить вас о проекте до начала работы, затем — effort high ([AI Made Simple](https://aimadesimple0.substack.com/p/how-to-use-claude-fable-5-better)).
- **Полная спецификация — в первом сообщении** (`/goal` в Claude Code, Outcome с rubric в Managed Agents). Хорошо специфицированный первый ход максимизирует автономность и минимизирует лишние токены после user-turn'ов.
- **Дозируйте видимость прогресса:** без настройки модель либо молчит (в стриминге thinking `omitted` выглядит как пауза), либо перерассказывает. Явно опишите, как должны выглядеть interim-апдейты.

## 4.3 Суб-агенты: делегируйте смело, но асинхронно

В отличие от прежних моделей (где делегирование часто подавляли guardrail'ами), параллельные суб-агенты у Fable 5 надёжны. Официальная рекомендация:

> Delegate independent subtasks to sub-agents and keep working while they run. Intervene if a sub-agent goes off track or is missing relevant context.

Асинхронная коммуникация с оркестратором выигрывает у spawn-and-block: долгоживущие агенты сохраняют контекст (экономия cache-read), оркестратор не ждёт самого медленного, контекст переживает подзадачи. Суб-агентам ставьте `effort: low/medium` — для Fable 5 это всё ещё очень сильный уровень.

## 4.4 Самопроверка и память

- **Явная self-verification:** «Establish a method for checking your own work as you build; run it every [интервал], verifying against the specification with sub-agents». Свежеконтекстные агенты-верификаторы работают лучше самокритики.
- **Поверхность памяти:** Fable 5 заметно сильнее, когда ей есть куда писать выводы — хотя бы обычный `.md`. Формат:
  > Store one lesson per file with a one-line summary at the top. Record corrections and confirmed approaches alike. Don't save what the repo already records; update existing notes rather than duplicating; delete notes that turn out wrong.
- **`send_to_user`-инструмент** для дословной доставки контента посреди длинного запуска: tool-inputs никогда не суммаризуются, поэтому цифры/deliverable доходят без искажений.

## 4.5 Task Budgets — бюджет на весь агентный цикл

Beta `task-budgets-2026-03-13`: модель видит убывающий счётчик и укладывается в бюджет грациозно, вместо обрыва по `max_tokens` (минимум `total`: 20 000):

```python
with client.beta.messages.stream(
    model="claude-fable-5", max_tokens=128000,
    output_config={"effort": "high", "task_budget": {"type": "tokens", "total": 64000}},
    betas=["task-budgets-2026-03-13"],
    messages=[...], tools=[...],
) as stream:
    response = stream.get_final_message()
```

## 4.6 Компакция и контекст

Для сессий, приближающихся к пределу контекста, — server-side compaction (beta `compact-2026-01-12`). Критично: возвращайте в историю **полный `response.content`**, а не только текст — compaction-блоки должны сохраняться. Не показывайте модели явный счётчик остатка контекста (провоцирует «context anxiety»).
