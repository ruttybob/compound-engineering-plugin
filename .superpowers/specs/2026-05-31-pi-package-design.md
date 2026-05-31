# Compound Engineering Pi Package — Design Spec

**Дата:** 2026-05-31
**Статус:** draft
**Цель:** Создать pi-package для compound-engineering, устанавливаемый через `pi install ./plugins/compound-engineering-pi/`

---

## Контекст

Compound-engineering — плагин для Claude Code с 40+ агентами, 40+ навыками, командами и MCP-серверами. В проекте уже есть pi-конвертер (`src/converters/claude-to-pi.ts`, `src/targets/pi.ts`), но пользователь хочет нативный pi-package без использования конвертера.

Pi — минималистичный coding agent harness. Не имеет встроенных субагентов (используются extensions). Поддерживает навыки (Agent Skills standard), prompt templates, extensions, themes. Пакеты распространяются через npm/git.

---

## Структура пакета

```
plugins/compound-engineering-pi/
├── package.json                  # Pi package manifest
├── AGENTS.md                     # Глобальные инструкции плагина для pi
├── skills/                       # ~37 навыков (минус 3 claude-only)
│   ├── ce-code-review/
│   │   ├── SKILL.md
│   │   └── references/
│   ├── ce-compound/
│   │   └── SKILL.md
│   ├── ce-plan/
│   │   └── SKILL.md
│   └── ...                       # Остальные навыки
├── agents/                       # ~40 агентов
│   ├── ce-repo-research-analyst.md
│   ├── ce-learnings-researcher.md
│   └── ...                       # Остальные агенты
└── prompts/                      # Команды как prompt templates
    ├── compound-refresh.md
    ├── work.md
    └── ...                       # Остальные команды
```

### package.json

```json
{
  "name": "compound-engineering-pi",
  "version": "1.0.0",
  "description": "Compound Engineering plugin adapted for pi coding agent",
  "keywords": ["pi-package"],
  "pi": {
    "skills": ["./skills"],
    "prompts": ["./prompts"]
  }
}
```

Pi автоматически обнаруживает `skills/*/SKILL.md`, `prompts/*.md`, `agents/*.md` через систему автодискавери (conventional directories).

---

## Адаптация контента навыков

### Трансформации

| Claude Code | Pi | Примечание |
|---|---|---|
| `Task X(args)` / `Agent X(args)` | `subagent with agent="x" and task="args"` | Синтаксис pi-subagents |
| `Agent({subagent_type: "ce-X"})` | `subagent with agent="ce-x"` | Pi-subagents dispatch |
| `TaskCreate/Update/List/Get/Stop/Output` | `todo create/update/list/get/stop/output` | Встроенный инструмент pi |
| `TodoWrite/TodoRead` | `todo` | Legacy маппинг |
| `AskUserQuestion` | `ask_user_question` | Инструмент pi (или pi-ask-user extension) |
| `$ARGUMENTS` | `$ARGUMENTS` | Pi поддерживает нативно |
| `${CLAUDE_PLUGIN_ROOT}` | Graceful fallback без переменной | Версия из описания или fallback |
| `${CLAUDE_SKILL_DIR}` | Относительные пути | Pi резолвит относительно навыка |
| `/command-name` | `/command-name` | Pi prompts совместимы |
| `${CLAUDE_SESSION_ID}` | Graceful fallback | Не используется pi |

### Frontmatter

Frontmatter SKILL.md сохраняется без изменений — pi поддерживает Agent Skills standard:
- `name` — обязателен, max 64 chars, lowercase + hyphens
- `description` — обязателен, max 1024 chars
- `argument-hint` — поддерживается pi

### Исключения

3 навыка помечены `ce_platforms: [claude]` и не переносятся:
- `ce-update`
- `ce-test-xcode`
- `ce-report-bug`

### References и скрипты

- Markdown-файлы в `references/` копируются с адаптацией Claude-специфичного контента
- Bash/python скрипты с `CLAUDE_*` переменными → добавляется graceful fallback
- Скрипты без Claude-специфичного кода копируются как есть

### Категории навыков по сложности адаптации

1. **Простые** (~15 шт.) — ce-commit, ce-clean-gone-branches, ce-worktree, ce-setup, ce-demo-reel, ce-release-notes, ce-product-pulse, ce-proof, ce-test-browser, ce-gemini-imagegen, ce-riffrec-feedback-analysis, ce-dogfood-beta. Минимальные правки: замена TodoWrite → todo, TaskCreate → todo create.

2. **Средние** (~15 шт.) — ce-code-review, ce-doc-review, ce-optimize, ce-debug, ce-simplify-code, ce-frontend-design, ce-resolve-pr-feedback, ce-slack-research, ce-strategy, ce-commit-push-pr, ce-polish-beta, ce-brainstorm. Нужна адаптация Task/Agent вызовов и условной логики.

3. **Сложные** (~7 шт.) — ce-compound, ce-compound-refresh, ce-plan, ce-work, ce-work-beta, ce-sessions, ce-ideate. Многоуровневые workflow с оркестрацией субагентов, требуют наиболее тщательной адаптации.

---

## Агенты

### Формат

Агенты — markdown-файлы в `agents/` каталоге. Pi-subagents extension резолвит их по имени при dispatch.

**Исходный формат** (Claude Code):
```yaml
---
name: ce-repo-research-analyst
description: "Analyses the repository to..."
capabilities: [...]
---
<body instructions>
```

**Pi формат:**
```yaml
---
name: ce-repo-research-analyst
description: "Analyses the repository to..."
---
## Capabilities
- ...

<body instructions — адаптированные>
```

### Адаптация тела

Те же трансформации, что и навыки: Task → subagent, TodoWrite → todo, и т.д.

---

## Команды → Prompt Templates

Команды compound-engineering конвертируются в pi prompt templates.

```markdown
<!-- prompts/compound-refresh.md -->
---
description: "Run compound engineering refresh cycle"
argument-hint: "[scope]"
---
<адаптированное тело команды>
```

Pi автоматически обнаруживает `.md` файлы в `prompts/` и регистрирует как `/compound-refresh` и т.д.

### Имена файлов

Имена файлов prompts формируются из имён команд: `workflows:review` → `review.md`, `plan_review` → `plan-review.md`.

---

## AGENTS.md

Глобальные инструкции плагина для pi. Содержит:

1. **Описание плагина** — что такое compound-engineering, его возможности
2. **Зависимости** — pi-subagents (обязательный), pi-ask-user (рекомендуемый)
3. **Конвенции** — ce- prefix для агентов, структура навыков
4. **Инструкция по установке** — `pi install ./plugins/compound-engineering-pi/`

Pi загружает AGENTS.md при старте как контекстный файл.

---

## Установка

```bash
# Локальная установка из репозитория
pi install ./plugins/compound-engineering-pi/

# Проверка
pi list
```

После установки pi автоматически обнаруживает навыки, промпты и AGENTS.md из пакета.

---

## Ограничения

1. **MCP-серверы** — не переносятся в pi-package. Pi использует mcporter.json для MCP-конфигурации, но это не часть pi-package. Пользователи могут настроить MCP отдельно.
2. **Hooks** — Claude Code hooks не поддерживаются pi. Поведение, завязанное на hooks, не переносится.
3. **Платформенные фильтры** — навыки с `ce_platforms: [claude]` исключены.
4. **Субагенты** — требуют установленный pi-subagents extension. Без него навыки с субагентами не работают.
5. **Синхронизация** — изменения в Claude-версии навыков не автоматически попадают в pi-package. Требуется ручная адаптация.

---

## Объём работ

| Категория | Количество | Оценка усилий |
|---|---|---|
| package.json + AGENTS.md | 2 файла | Низкое |
| Простые навыки | ~15 шт. | Низкое (механические замены) |
| Средние навыки | ~15 шт. | Среднее (адаптация Task/Agent) |
| Сложные навыки | ~7 шт. | Высокое (оркестрация workflow) |
| Агенты | ~40 шт. | Низкое-среднее (механические замены + проверка) |
| Prompts | ~команды | Низкое |
| Итого | ~80+ файлов | Значительный |
