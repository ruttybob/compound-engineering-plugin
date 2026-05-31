# Compound Engineering Pi Package — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use subagent-driven-development (recommended) or executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Создать нативный pi-package `compound-engineering-pi` со всеми навыками, агентами и промптами, устанавливаемый через `pi install ./plugins/compound-engineering-pi/`.

**Architecture:** Отдельный каталог `plugins/compound-engineering-pi/` с pi-native структурой (skills/, agents/, prompts/, AGENTS.md, package.json). Содержимое копируется из `plugins/compound-engineering/` и адаптируется: Claude-специфичные API (Task, Agent, TodoWrite, AskUserQuestion) заменяются на pi-эквиваленты (subagent, todo, ask_user_question).

**Tech Stack:** Markdown, YAML frontmatter, pi Agent Skills standard, bash-скрипты

---

## File Structure

```
plugins/compound-engineering-pi/
├── package.json                              # Pi package manifest
├── AGENTS.md                                 # Глобальные инструкции плагина
├── skills/                                   # 38 навыков (все кроме ce-update)
│   ├── ce-code-review/
│   │   ├── SKILL.md
│   │   └── references/                       # Reference-файлы (копия + адаптация)
│   ├── ce-compound/
│   │   ├── SKILL.md
│   │   ├── references/
│   │   ├── scripts/
│   │   └── assets/
│   ├── ...                                   # Остальные навыки
│   └── lfg/
│       ├── SKILL.md
│       └── references/
├── agents/                                   # 43 агента
│   ├── ce-repo-research-analyst.md
│   ├── ce-learnings-researcher.md
│   └── ...                                   # Остальные агенты
└── prompts/                                  # ~10 prompt templates из корневых команд
    └── triage-prs.md                         # Единственная корневая команда
```

---

## Общие правила адаптации контента

Все трансформации применяются при копировании файлов из `plugins/compound-engineering/` в `plugins/compound-engineering-pi/`.

### Маппинг API

| Claude Code | Pi | Контекст |
|---|---|---|
| `Task ce-X(args)` | `subagent with agent="ce-x" and task="args"` | Запуск субагента |
| `Agent({subagent_type: "ce-X"})` | `subagent with agent="ce-x"` | Typизированный dispatch |
| `TaskCreate`, `TaskUpdate`, `TaskList`, `TaskGet`, `TaskStop`, `TaskOutput` | `todo create`, `todo update`, `todo list`, `todo get`, `todo stop`, `todo output` | Управление задачами |
| `TodoWrite`, `TodoRead` | `todo` (или `todo list` / `todo update`) | Legacy |
| `AskUserQuestion` | `ask_user_question` | Взаимодействие с пользователем |
| `${CLAUDE_PLUGIN_ROOT}` | Fallback без переменной (см. паттерн ниже) | Пути к ресурсам плагина |
| `${CLAUDE_SKILL_DIR}` | Относительные пути (`./scripts/`, `./references/`) | Пути внутри навыка |
| `${CLAUDE_SESSION_ID}` | Удалить или заменить на fallback | ID сессии |
| `Skill ce-X` | `/skill:ce-x` | Вызов навыка |
| `/workflows:X` | `/x` | Slash-команды без префикса |
| `/prompts:X` | `/x` | Slash-команды без префикса |

### Паттерн CLAUDE_PLUGIN_ROOT fallback

```markdown
**Plugin version (pre-resolved):** !`jq -r .version "${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json"`

If the line above resolved to a semantic version (e.g., `2.42.0`), use it.
Otherwise (empty, a literal command string, or an error), use the versionless fallback.
Do not attempt to resolve the version at runtime.
```

Если навык ссылается на `${CLAUDE_PLUGIN_ROOT}`, заменить на относительный путь или удалить блок с комментарием.

### Frontmatter

- `name` — оставить как есть (ce- prefix совместим с pi)
- `description` — оставить как есть
- `argument-hint` — оставить как есть
- `model`, `tools` — удалить (не поддерживаются pi Agent Skills standard)
- `ce_platforms` — удалить
- `disableModelInvocation` — удалить (pi поддерживает `disable-model-invocation` в frontmatter, но семантика другая)

---

## Task 1: Создать скелет пакета

**Files:**
- Create: `plugins/compound-engineering-pi/package.json`
- Create: `plugins/compound-engineering-pi/AGENTS.md`

- [ ] **Step 1: Создать package.json**

```json
{
  "name": "compound-engineering-pi",
  "version": "1.0.0",
  "description": "Compound Engineering plugin adapted for pi coding agent. AI-powered development tools for code review, research, design, and workflow automation.",
  "keywords": ["pi-package"],
  "pi": {
    "skills": ["./skills"],
    "prompts": ["./prompts"]
  }
}
```

- [ ] **Step 2: Создать AGENTS.md**

```markdown
# Compound Engineering (Pi)

AI-powered development tools for code review, research, design, and workflow automation.

## Dependencies

This plugin requires:

- **pi-subagents** (required) — provides the `subagent` tool used by skills that dispatch parallel agents
  ```bash
  pi install npm:pi-subagents
  ```
- **pi-ask-user** (recommended) — provides the `ask_user_question` tool; skills fall back to numbered options in chat when it is missing
  ```bash
  pi install npm:pi-ask-user
  ```

## Skills

All skills use the `ce-` prefix and are invoked via `/skill:ce-<name>`.

Key workflows:
- `/skill:ce-compound` — Full compound cycle (research + plan + work + review)
- `/skill:ce-code-review` — Structured code review with persona agents
- `/skill:ce-plan` — Implementation planning
- `/skill:ce-work` — Execute planned work
- `/skill:ce-brainstorm` — Requirements exploration

## Agents

Agents are resolved by pi-subagents when dispatched via the `subagent` tool.
All agent names use the `ce-` prefix (e.g., `ce-repo-research-analyst`).

## Conventions

- Agent names: `ce-<descriptive-name>` (e.g., `ce-security-reviewer`)
- Skill names: `ce-<workflow-name>` (e.g., `ce-code-review`)
- The `lfg` skill runs the full autonomous pipeline end-to-end
```

- [ ] **Step 3: Создать пустые каталоги**

```bash
mkdir -p plugins/compound-engineering-pi/skills
mkdir -p plugins/compound-engineering-pi/agents
mkdir -p plugins/compound-engineering-pi/prompts
```

- [ ] **Step 4: Проверить что pi видит пакет**

```bash
cd plugins/compound-engineering-pi && pi list && cd ../..
```

Ожидаемо: pi может не показать содержимое пока нет навыков, но package.json должен быть валидным.

- [ ] **Step 5: Commit**

```bash
git add plugins/compound-engineering-pi/
git commit -m "feat(pi-package): create package skeleton with package.json and AGENTS.md"
```

---

## Task 2: Скопировать и адаптировать агентов (43 файла)

**Files:**
- Create: `plugins/compound-engineering-pi/agents/ce-*.md` (43 файла)

Агенты в основном не содержат Claude-специфичного контента (0 с AskUserQuestion, 1 с TodoWrite, 1 с Task). Адаптация минимальна.

- [ ] **Step 1: Создать скрипт массовой адаптации агентов**

Скрипт копирует все агенты и применяет трансформации. Создать временный скрипт:

```bash
cat > /tmp/adapt-agents.sh << 'SCRIPT'
#!/bin/bash
set -e

SRC="plugins/compound-engineering/agents"
DST="plugins/compound-engineering-pi/agents"

mkdir -p "$DST"

for f in "$SRC"/*.md; do
  name=$(basename "$f")
  
  # Copy file
  cp "$f" "$DST/$name"
  
  # Remove Claude-specific frontmatter fields
  sed -i '' '/^model:/d' "$DST/$name"
  sed -i '' '/^tools:/d' "$DST/$name"
  
  # Transform Task calls: Task ce-X(args) -> subagent with agent="ce-x" and task="args"
  sed -i '' -E 's/Task ([a-z][a-z0-9_-]+)\(([^)]*)\)/subagent with agent="\1" and task="\2"/g' "$DST/$name"
  
  # Transform TodoWrite/TodoRead -> todo
  sed -i '' 's/TodoWrite/todo update/g' "$DST/$name"
  sed -i '' 's/TodoRead/todo list/g' "$DST/$name"
  
  # Transform TaskCreate/Update/List/Get -> todo equivalents
  sed -i '' 's/TaskCreate/todo create/g' "$DST/$name"
  sed -i '' 's/TaskUpdate/todo update/g' "$DST/$name"
  sed -i '' 's/TaskList/todo list/g' "$DST/$name"
  sed -i '' 's/TaskGet/todo get/g' "$DST/$name"
  sed -i '' 's/TaskStop/todo stop/g' "$DST/$name"
  sed -i '' 's/TaskOutput/todo output/g' "$DST/$name"
  
  echo "Adapted: $name"
done
SCRIPT
chmod +x /tmp/adapt-agents.sh
bash /tmp/adapt-agents.sh
```

- [ ] **Step 2: Визуально проверить 3-4 файла на корректность**

Проверить что frontmatter содержит только name и description, что Task/TodoWrite заменились:

```bash
head -10 plugins/compound-engineering-pi/agents/ce-repo-research-analyst.md
head -10 plugins/compound-engineering-pi/agents/ce-swift-ios-reviewer.md
head -10 plugins/compound-engineering-pi/agents/ce-session-historian.md
grep -rn "TodoWrite\|TaskCreate\|CLAUDE_\|model:\|tools:" plugins/compound-engineering-pi/agents/
```

Ожидаемо: grep ничего не находит, frontmatter чистый.

- [ ] **Step 3: Commit**

```bash
git add plugins/compound-engineering-pi/agents/
git commit -m "feat(pi-package): add 43 adapted agents"
```

---

## Task 3: Скопировать и адаптировать простые навыки (без references/scripts)

**Files:**
- Create: `plugins/compound-engineering-pi/skills/ce-*/SKILL.md` (~20 навыков без extras)

Простые навыки — те, у которых только SKILL.md без references/, scripts/ или assets/.

- [ ] **Step 1: Определить список простых навыков**

```bash
for skill in plugins/compound-engineering/skills/*/; do
  name=$(basename "$skill")
  # Skip ce-update (claude-only)
  [ "$name" = "ce-update" ] && continue
  extras=$(ls "$skill" 2>/dev/null | grep -v SKILL.md | grep -v frontmatter)
  if [ -z "$extras" ]; then
    echo "$name"
  fi
done
```

Ожидаемые: ce-agent-native-architecture (с references!), ce-agent-native-audit, ce-clean-gone-branches (с scripts!), ce-commit, ce-dhh-rails-style, ce-sessions (с scripts!), ce-worktree (с scripts!), ce-product-pulse (с references!), ce-proof (с references!), ce-strategy (с references!)

Пересчитать: «простые» = навыки где адаптация контента минимальна (мало Task/Agent/TodoWrite). Сначала скопировать все навыки механически, затем адаптировать.

- [ ] **Step 2: Создать скрипт массового копирования навыков**

```bash
cat > /tmp/copy-skills.sh << 'SCRIPT'
#!/bin/bash
set -e

SRC="plugins/compound-engineering/skills"
DST="plugins/compound-engineering-pi/skills"

# Skip ce-update (claude-only)
SKIPS="ce-update"

mkdir -p "$DST"

for skill in "$SRC"/*/; do
  name=$(basename "$skill")
  
  # Skip claude-only
  if echo "$SKIPS" | grep -qw "$name"; then
    echo "Skipped: $name"
    continue
  fi
  
  # Copy entire skill directory
  mkdir -p "$DST/$name"
  cp -R "$skill"* "$DST/$name/"
  
  echo "Copied: $name"
done
SCRIPT
chmod +x /tmp/copy-skills.sh
bash /tmp/copy-skills.sh
```

- [ ] **Step 3: Применить трансформации ко всем SKILL.md файлам**

```bash
cat > /tmp/transform-skills.sh << 'SCRIPT'
#!/bin/bash
set -e

DST="plugins/compound-engineering-pi/skills"

for skillmd in "$DST"/*/SKILL.md; do
  [ -f "$skillmd" ] || continue
  
  # Transform Task ce-X(args) -> subagent with agent="ce-x" and task="args"
  sed -i '' -E 's/Task ([a-z][a-z0-9_-]+)\(([^)]*)\)/subagent with agent="\1" and task="\2"/g' "$skillmd"
  
  # Transform Agent({subagent_type: "ce-X"}) -> subagent with agent="ce-x"
  sed -i '' -E 's/Agent\(\{subagent_type: "([^"]+)"\}\)/subagent with agent="\1"/g' "$skillmd"
  
  # Transform Claude task-tracking primitives
  sed -i '' 's/TaskCreate/todo create/g' "$skillmd"
  sed -i '' 's/TaskUpdate/todo update/g' "$skillmd"
  sed -i '' 's/TaskList/todo list/g' "$skillmd"
  sed -i '' 's/TaskGet/todo get/g' "$skillmd"
  sed -i '' 's/TaskStop/todo stop/g' "$skillmd"
  sed -i '' 's/TaskOutput/todo output/g' "$skillmd"
  
  # Transform legacy TodoWrite/TodoRead
  sed -i '' 's/TodoWrite/todo update/g' "$skillmd"
  sed -i '' 's/TodoRead/todo list/g' "$skillmd"
  
  # Transform AskUserQuestion -> ask_user_question
  sed -i '' 's/AskUserQuestion/ask_user_question/g' "$skillmd"
  
  # Remove ce_platforms frontmatter
  sed -i '' '/^ce_platforms:/d' "$skillmd"
  
  # Transform CLAUDE_PLUGIN_ROOT references
  sed -i '' 's/\${CLAUDE_PLUGIN_ROOT}/\${PI_PLUGIN_ROOT:-.}/g' "$skillmd"
  sed -i '' 's/\$CLAUDE_PLUGIN_ROOT/\${PI_PLUGIN_ROOT:-.}/g' "$skillmd"
  
  # Transform CLAUDE_SKILL_DIR -> relative path
  sed -i '' 's/\${CLAUDE_SKILL_DIR}/./g' "$skillmd"
  sed -i '' 's/\$CLAUDE_SKILL_DIR/./g' "$skillmd"
  
  # Transform CLAUDE_SESSION_ID -> remove or fallback
  sed -i '' 's/\${CLAUDE_SESSION_ID}/pi-session/g' "$skillmd"
  sed -i '' 's/\$CLAUDE_SESSION_ID/pi-session/g' "$skillmd"
  
  # Transform /workflows:X -> /x
  sed -i '' -E 's/\/workflows:([a-z][a-z0-9_-]+)/\/\1/g' "$skillmd"
  
  # Transform /prompts:X -> /x
  sed -i '' -E 's/\/prompts:([a-z][a-z0-9_-]+)/\/\1/g' "$skillmd"
  
  echo "Transformed: $(basename $(dirname "$skillmd"))"
done
SCRIPT
chmod +x /tmp/transform-skills.sh
bash /tmp/transform-skills.sh
```

- [ ] **Step 4: Проверить что не осталось Claude-специфичных паттернов**

```bash
grep -rn "TaskCreate\|TaskUpdate\|TodoWrite\|TodoRead\|CLAUDE_PLUGIN_ROOT\|CLAUDE_SKILL_DIR\|CLAUDE_SESSION_ID\|ce_platforms" plugins/compound-engineering-pi/skills/*/SKILL.md
grep -rn "AskUserQuestion" plugins/compound-engineering-pi/skills/*/SKILL.md | head -5
```

Ожидаемо: пустой вывод для первого grep. Для AskUserQuestion — может остаться в виде описательной ссылки (не как вызов API), что допустимо.

- [ ] **Step 5: Проверить что frontmatter валиден для pi**

Pi требует `name` (1-64 chars, lowercase, hyphens) и `description` (max 1024 chars):

```bash
for skillmd in plugins/compound-engineering-pi/skills/*/SKILL.md; do
  name=$(grep "^name:" "$skillmd" | head -1 | sed 's/name: *//')
  desc_len=$(grep "^description:" "$skillmd" | head -1 | sed 's/description: *//' | wc -c)
  if [ ${#name} -gt 64 ]; then
    echo "NAME TOO LONG: $name ($(echo -n "$name" | wc -c) chars)"
  fi
  if [ "$desc_len" -gt 1024 ]; then
    echo "DESC TOO LONG: $(basename $(dirname "$skillmd")) ($desc_len chars)"
  fi
done
echo "Frontmatter check complete"
```

Ожидаемо: нет ошибок.

- [ ] **Step 6: Commit**

```bash
git add plugins/compound-engineering-pi/skills/
git commit -m "feat(pi-package): add 38 adapted skills with content transformations"
```

---

## Task 4: Адаптировать reference-файлы и скрипты

**Files:**
- Modify: `plugins/compound-engineering-pi/skills/*/references/*.md` — адаптация контента
- Modify: `plugins/compound-engineering-pi/skills/*/scripts/*` — адаптация bash-скриптов

- [ ] **Step 1: Найти все reference-файлы с Claude-специфичным контентом**

```bash
grep -rl "CLAUDE_\|TodoWrite\|TaskCreate\|AskUserQuestion\|Task [a-z]" plugins/compound-engineering-pi/skills/*/references/ plugins/compound-engineering-pi/skills/*/scripts/ 2>/dev/null
```

- [ ] **Step 2: Адаптировать каждый найденный файл**

Применить те же трансформации что в Task 3 Step 3, но к reference-файлам и скриптам:

```bash
cat > /tmp/transform-extras.sh << 'SCRIPT'
#!/bin/bash
set -e

DST="plugins/compound-engineering-pi/skills"

# Transform all .md files under references/
for ref in $(find "$DST" -path "*/references/*.md"); do
  sed -i '' 's/TaskCreate/todo create/g' "$ref"
  sed -i '' 's/TaskUpdate/todo update/g' "$ref"
  sed -i '' 's/TaskList/todo list/g' "$ref"
  sed -i '' 's/TaskGet/todo get/g' "$ref"
  sed -i '' 's/TodoWrite/todo update/g' "$ref"
  sed -i '' 's/TodoRead/todo list/g' "$ref"
  sed -i '' 's/AskUserQuestion/ask_user_question/g' "$ref"
  sed -i '' -E 's/Task ([a-z][a-z0-9_-]+)\(([^)]*)\)/subagent with agent="\1" and task="\2"/g' "$ref"
  sed -i '' 's/\${CLAUDE_PLUGIN_ROOT}/\${PI_PLUGIN_ROOT:-.}/g' "$ref"
  sed -i '' 's/\${CLAUDE_SKILL_DIR}/./g' "$ref"
  echo "Adapted ref: $ref"
done

# Transform all shell scripts
for script in $(find "$DST" -path "*/scripts/*" -type f); do
  sed -i '' 's/\$CLAUDE_PLUGIN_ROOT/\${PI_PLUGIN_ROOT:-.}/g' "$script"
  sed -i '' 's/\${CLAUDE_PLUGIN_ROOT}/\${PI_PLUGIN_ROOT:-.}/g' "$script"
  echo "Adapted script: $script"
done
SCRIPT
chmod +x /tmp/transform-extras.sh
bash /tmp/transform-extras.sh
```

- [ ] **Step 3: Проверить что скрипты не сломаны**

```bash
# Check shell syntax of adapted scripts
find plugins/compound-engineering-pi/skills/ -name "*.sh" -exec bash -n {} \; 2>&1
echo "Syntax check complete"
```

Ожидаемо: нет ошибок синтаксиса.

- [ ] **Step 4: Commit**

```bash
git add plugins/compound-engineering-pi/skills/
git commit -m "feat(pi-package): adapt references and scripts for pi compatibility"
```

---

## Task 5: Адаптировать сложные навыки вручную

**Files:**
- Modify: `plugins/compound-engineering-pi/skills/ce-compound/SKILL.md`
- Modify: `plugins/compound-engineering-pi/skills/ce-compound-refresh/SKILL.md`
- Modify: `plugins/compound-engineering-pi/skills/ce-plan/SKILL.md`
- Modify: `plugins/compound-engineering-pi/skills/ce-work/SKILL.md`
- Modify: `plugins/compound-engineering-pi/skills/ce-work-beta/SKILL.md`
- Modify: `plugins/compound-engineering-pi/skills/ce-code-review/SKILL.md`
- Modify: `plugins/compound-engineering-pi/skills/lfg/SKILL.md`

Механические замены из Task 3 покрывают ~80% случаев. Оставшиеся 20% — контекстные паттерны, которые sed не ловит. Эти навыки нужно проверить вручную.

- [ ] **Step 1: Прочитать каждый сложный навык и составить список оставшихся проблем**

```bash
for skill in ce-compound ce-compound-refresh ce-plan ce-work ce-work-beta ce-code-review lfg ce-sessions ce-ideate ce-brainstorm; do
  echo "=== $skill ==="
  # Look for remaining Claude-specific patterns
  grep -n "Agent(\|Skill [a-z]\|subagent_type\|\/workflows:\|\/prompts:" "plugins/compound-engineering-pi/skills/$skill/SKILL.md" 2>/dev/null | head -10
  echo ""
done
```

- [ ] **Step 2: Исправить оставшиеся паттерны в каждом навыке**

Для каждого навыка из списка выше — прочитать файл, найти и заменить:

1. `Agent({subagent_type: "ce-X", ...})` → `subagent with agent="ce-x"`
2. `Skill ce-X` → `/skill:ce-x`
3. `subagent_type:` ссылки → `agent="ce-x"`
4. `/workflows:X` и `/prompts:X` → `/x`
5. Условные конструкции типа `if the host platform supports X` — упростить для pi

Использовать `edit` tool для точечных замен.

- [ ] **Step 3: Проверить что lfg корректно вызывает навыки**

Навык `lfg` — автономный pipeline. Проверить что все вызовы навыков используют `/skill:ce-X` формат:

```bash
grep -n "Skill\|skill:" plugins/compound-engineering-pi/skills/lfg/SKILL.md | head -20
```

Ожидаемо: все ссылки на навыки в формате `/skill:ce-X`.

- [ ] **Step 4: Commit**

```bash
git add plugins/compound-engineering-pi/skills/
git commit -m "fix(pi-package): manual adaptation of complex workflow skills"
```

---

## Task 6: Создать prompt templates

**Files:**
- Create: `plugins/compound-engineering-pi/prompts/triage-prs.md`

Команды compound-engineering были мигрированы в навыки (v2.39.0). Единственная оставшаяся команда в корне репозитория — `triage-prs.md`, которая является командой для разработки, не для пользователей плагина.

- [ ] **Step 1: Проверить необходимость prompt templates**

Команды (workflows:review, plan_review и т.д.) были мигрированы в навыки и теперь вызываются через `/skill:ce-X`. Prompt templates нужны только если есть отдельные команды, не покрытые навыками.

```bash
# Check root commands
cat .claude/commands/triage-prs.md
```

- [ ] **Step 2: Решить — включать triage-prs или нет**

`triage-prs` — это команда для development workflow этого репозитория, не для пользователей плагина. Не включать в pi-package.

Если есть другие команды, которые стоит включить — добавить их в `prompts/`.

- [ ] **Step 3: Commit (если есть изменения)**

Если prompts/ пуст — можно пропустить commit.

```bash
# If prompts are empty, add .gitkeep
touch plugins/compound-engineering-pi/prompts/.gitkeep
git add plugins/compound-engineering-pi/prompts/
git commit -m "feat(pi-package): add empty prompts directory"
```

---

## Task 7: Валидация пакета

**Files:**
- Verify: весь `plugins/compound-engineering-pi/`

- [ ] **Step 1: Проверить структуру пакета**

```bash
find plugins/compound-engineering-pi/ -type f | head -50
echo "---"
echo "Total files: $(find plugins/compound-engineering-pi/ -type f | wc -l)"
echo "Skills: $(ls plugins/compound-engineering-pi/skills/ | wc -l)"
echo "Agents: $(ls plugins/compound-engineering-pi/agents/*.md | wc -l)"
echo "Prompts: $(ls plugins/compound-engineering-pi/prompts/*.md 2>/dev/null | wc -l)"
```

Ожидаемо: ~38 навыков, 43 агента, 0-1 промптов.

- [ ] **Step 2: Проверить что package.json валидный**

```bash
cat plugins/compound-engineering-pi/package.json | python3 -m json.tool
```

- [ ] **Step 3: Проверить что все SKILL.md имеют валидный frontmatter**

```bash
for skillmd in plugins/compound-engineering-pi/skills/*/SKILL.md; do
  name=$(grep "^name:" "$skillmd" | head -1 | sed 's/name: *//')
  desc=$(grep "^description:" "$skillmd" | head -1 | sed 's/description: *//')
  if [ -z "$name" ]; then
    echo "MISSING NAME: $skillmd"
  fi
  if [ -z "$desc" ]; then
    echo "MISSING DESCRIPTION: $skillmd"
  fi
done
echo "All frontmatter validated"
```

- [ ] **Step 4: Пробная установка через pi install**

```bash
pi install ./plugins/compound-engineering-pi/
```

Ожидаемо: pi устанавливает пакет без ошибок.

- [ ] **Step 5: Проверить что pi видит установленные навыки**

```bash
pi list
```

Ожидаемо: видны навыки из compound-engineering-pi.

- [ ] **Step 6: Проверить что навык загружается**

Запустить pi и проверить `/skill:ce-code-review` — должен загрузиться навык.

- [ ] **Step 7: Commit финальный**

```bash
git add plugins/compound-engineering-pi/
git commit -m "feat(pi-package): complete compound-engineering pi package v1.0.0"
```

---

## Task 8: Очистка

**Files:**
- Remove: временные скрипты `/tmp/adapt-agents.sh`, `/tmp/copy-skills.sh`, `/tmp/transform-skills.sh`, `/tmp/transform-extras.sh`

- [ ] **Step 1: Удалить временные скрипты**

```bash
rm -f /tmp/adapt-agents.sh /tmp/copy-skills.sh /tmp/transform-skills.sh /tmp/transform-extras.sh
```

- [ ] **Step 2: Проверить что нет мусора в пакете**

```bash
find plugins/compound-engineering-pi/ -name ".DS_Store" -o -name "*.bak" -o -name "*~"
```

Ожидаемо: пусто.
