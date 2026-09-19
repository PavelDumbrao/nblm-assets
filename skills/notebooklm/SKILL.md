---
name: notebooklm-operator
description: Use when working with Google NotebookLM through notebooklm-py CLI or its optional MCP server: authenticate, create notebooks, add and verify sources, ask grounded questions with citations, run research, generate and download Studio artifacts, build content pipelines, or troubleshoot sessions and imports.
---

# NotebookLM Operator
## 0. Актуализация для переносимой установки

Этот skill должен оставаться version-aware.

На дату подготовки Hardcoding PRO Practical, 19.09.2026, latest stable `notebooklm-py` в upstream: `0.8.2`.
Но агент НЕ должен слепо фиксировать эту версию навсегда.

Перед установкой или обновлением:

```bash
python --version
notebooklm --version
notebooklm --help
```

Если `notebooklm` ещё не установлен, сначала определи ОС, shell, доступный Python и менеджер окружений.

### Базовая стратегия по ОС

**macOS / Linux**

Предпочтительный CLI-вариант:

```bash
uv tool install "notebooklm-py[browser]"
```

Fallback:

```bash
pipx install "notebooklm-py[browser]"
```

Если используется проектный virtualenv, допустим:

```bash
python -m pip install "notebooklm-py[browser]"
```

Не используй `--break-system-packages`.

**Windows 10/11**

Сначала проверь `python`, `py`, `uv`, `pipx`.

Допустимы:

```powershell
uv tool install "notebooklm-py[browser]"
```

или из активного virtualenv:

```powershell
py -m pip install "notebooklm-py[browser]"
```

После установки открой новый shell, если `notebooklm` ещё не появился в PATH.

**WSL**

Считай WSL отдельной Linux-средой.
Браузер для login может открыться на Windows-хосте, а профиль NotebookLM хранится в WSL.

### Skill install для AI-агента

После установки CLI сначала проверь:

```bash
notebooklm skill --help
```

Если установленная версия поддерживает штатную установку skill:

```bash
notebooklm skill install
```

Upstream умеет регистрировать skill для совместимых агентных каталогов, включая `~/.claude/skills/` и `~/.agents/skills/`.

Если в проекте уже лежит этот файл как `SKILL.md`, не создавай конкурирующую копию вслепую.
Сначала проверь, какой skill-path реально читает текущий агент.

### MCP

MCP не является обязательным для прохождения practical.

Сначала добейся рабочего CLI:

```bash
notebooklm auth check --test --json
notebooklm list --limit 1 --json
```

Только потом, если клиент поддерживает MCP, проверь документацию установленной версии:

```bash
notebooklm mcp --help
```

Для поддерживаемых клиентов upstream может уметь:

```bash
notebooklm mcp install claude-code
notebooklm mcp install cursor
notebooklm mcp install windsurf
notebooklm mcp install claude-desktop
```

Не выдумывай имя MCP-клиента.
Если текущий агент не поддерживается штатным installer, продолжай через CLI + skill.

### Авторизация

Никогда не проси пользователя вставлять Google cookies, `storage_state.json`, master token или другие auth-данные в чат.

Обычный путь:

```bash
notebooklm login
notebooklm auth check --test --json
```

Успех подтверждается сетевой проверкой, а не существованием auth-файла.

### Обязательный smoke test

После установки:

```bash
notebooklm --version
notebooklm auth check --test --json
notebooklm list --limit 1 --json
```

Затем создай отдельный безопасный тестовый notebook, добавь короткий source с уникальным фактом и проверь grounded-answer с цитатой.
Не удаляй чужие notebooks и sources.
Destructive actions выполняй только после точной проверки ID и явного подтверждения.


Универсальный переносимый skill для работы с Google NotebookLM через community-пакет `notebooklm-py`.

Этот файл можно:

- читать как практическую инструкцию;
- переименовать в `SKILL.md` и положить в каталог skill;
- использовать в Codex, Claude Code или другом агентном окружении;
- адаптировать под локальную команду-обёртку, например `nblm`.

## 1. Назначение

Используй этот skill, когда нужно:

- создать или наполнить NotebookLM notebook;
- загрузить сайты, YouTube, документы, презентации или текст;
- получить grounded-ответ по источникам с цитатами;
- провести Deep Research;
- сделать подкаст, видео, слайды, квиз, карточки, инфографику, mind map или таблицу;
- превратить YouTube, встречу, курс или исследование в контент-пакет;
- организовать итерационный research loop;
- подключить экспериментальный NotebookLM MCP;
- диагностировать авторизацию, источники или генерацию артефактов.

## 2. Когда не использовать

Не используй NotebookLM как единственный источник истины, если задача требует:

- подтверждения текущего production-состояния;
- проверки реально работающего кода;
- актуальных цен, законов, тарифов или API-контрактов;
- контроля доступа к персональным данным;
- хранения транзакционного состояния;
- работы с секретами, `.env`, токенами или приватными ключами.

NotebookLM помогает анализировать источники, но не заменяет проверку исходных документов, кода, API, базы данных и официальной документации.

## 3. Важное ограничение

`notebooklm-py` является community-проектом, а не официальным Google CLI или официальным публичным API NotebookLM.

Перед работой всегда проверяй установленную версию и доступные команды:

```bash
notebooklm --version
notebooklm --help
notebooklm source --help
notebooklm generate --help
```

Некоторые команды и аргументы зависят от версии. Не предполагай наличие функции только по этому документу. Источник истины для установленной версии:

```bash
notebooklm <command> --help
```

Основной репозиторий и документация:

- https://github.com/teng-lin/notebooklm-py
- https://github.com/teng-lin/notebooklm-py/blob/main/docs/cli-reference.md
- https://github.com/teng-lin/notebooklm-py/blob/main/docs/mcp-guide.md

## 4. CLI, MCP и локальная обёртка

### CLI

Основная переносимая команда:

```bash
notebooklm
```

CLI подходит для:

- ручной диагностики;
- shell-скриптов;
- cron;
- JSON-вывода;
- массовой загрузки источников;
- понятного аудита команд;
- разделения запуска, ожидания и скачивания.

### Локальная обёртка

В некоторых окружениях может существовать команда:

```bash
nblm
```

Обычно это обёртка над `notebooklm` с заранее выбранным профилем или сетевыми настройками. Не предполагай, что `nblm` существует на новом компьютере. Сначала проверь:

```bash
command -v nblm
```

В переносимых инструкциях используй `notebooklm`.

### MCP

MCP превращает операции NotebookLM в структурированные agent tools. Вместо запуска shell-команды агент вызывает инструмент с именованными аргументами.

MCP удобен, когда агент должен самостоятельно:

- найти notebook;
- добавить source;
- дождаться обработки;
- задать grounded-вопрос;
- запустить Studio artifact;
- проверить статус;
- скачать результат.

В `notebooklm-py` MCP является опциональным и экспериментальным слоем. Его tool schemas могут меняться между версиями.

## 5. Выбор транспорта

Используй CLI по умолчанию, если:

- окружение уже работает на `0.7.x`;
- нужна максимальная прозрачность;
- задача запускается через скрипт или cron;
- необходимо сохранить сырой JSON;
- MCP ещё не прошёл отдельный smoke test.

Используй MCP, если:

- установлен совместимый `notebooklm-py[mcp]`;
- клиент поддерживает MCP;
- нужна частая агентная работа без shell parsing;
- сервер доступен только локально или надёжно защищён;
- tool inventory и авторизация проверены.

Не обновляй рабочий `0.7.x` контур прямо поверх него ради MCP. Сначала создай отдельное тестовое окружение и проверь миграцию.

## 6. Установка CLI

Рекомендуемый изолированный вариант:

```bash
python3 -m venv .venv-notebooklm
source .venv-notebooklm/bin/activate
python -m pip install --upgrade pip
python -m pip install notebooklm-py
notebooklm --version
```

Если нужны дополнительные browser-cookie возможности, сверяй extras с документацией установленной версии.

Не устанавливай новый пакет поверх боевого окружения без проверки release notes и возможности отката.

## 7. Авторизация

### Интерактивный вход

```bash
notebooklm login
```

CLI открывает браузер, пользователь входит в Google, после чего сохраняется browser session.

### Проверка

```bash
notebooklm auth check --test --json
```

Дополнительная диагностика:

```bash
notebooklm status --paths
notebooklm doctor
notebooklm profile list
```

### Несколько профилей

```bash
notebooklm profile list
notebooklm profile create work
notebooklm --profile work login
notebooklm --profile work auth check --test --json
```

### Правила безопасности авторизации

- Никогда не показывай содержимое `storage_state.json`.
- Не отправляй cookies через чат, Telegram или внешнему агенту.
- Не коммить auth-профили в Git.
- Наличие cookie-файла не доказывает работоспособность сессии.
- Источник истины: успешный `auth check --test --json`.
- Не запускай одновременно несколько keepalive-процессов для одного профиля без проверки поведения версии.
- Headless master token является чувствительным credential уровня Google-аккаунта. Используй отдельный аккаунт и только если это действительно необходимо.

## 8. Базовый рабочий цикл

```bash
notebooklm list --limit 20 --no-truncate
notebooklm create "Исследование: тема" --use --json
notebooklm source add "https://example.com/article" --json
notebooklm source list --json
notebooklm ask "Сделай краткий вывод только по источникам, с цитатами." --json
```

Для автоматизации всегда предпочитай явный notebook ID:

```bash
notebooklm metadata -n <NOTEBOOK_ID> --json
notebooklm source list -n <NOTEBOOK_ID> --json
notebooklm ask "Вопрос" -n <NOTEBOOK_ID> --json
```

Не полагайся на текущий notebook, если скрипт может работать параллельно или после другой команды `use`.

## 9. Управление notebooks

Основные операции:

```bash
notebooklm list --limit 20 --no-truncate
notebooklm create "Название" --use --json
notebooklm use <NOTEBOOK_ID>
notebooklm rename "Новое название" -n <NOTEBOOK_ID>
notebooklm metadata -n <NOTEBOOK_ID> --json
notebooklm summary -n <NOTEBOOK_ID>
```

Удаление является destructive-операцией. Перед ним проверь точный ID, название и необходимость удаления:

```bash
notebooklm delete -n <NOTEBOOK_ID> -y
```

## 10. Добавление источников

### URL

```bash
notebooklm source add "https://example.com/page" \
  -n <NOTEBOOK_ID> \
  --json
```

### YouTube

```bash
notebooklm source add "https://youtube.com/watch?v=<VIDEO_ID>" \
  -n <NOTEBOOK_ID> \
  --json
```

Обычно NotebookLM получает текст субтитров, а не визуальное содержание видео. Видео без доступных captions может не импортироваться.

### Локальный файл

```bash
notebooklm source add "/absolute/path/report.pdf" \
  -n <NOTEBOOK_ID> \
  --json
```

### Inline-текст

```bash
notebooklm source add "Текст материала" \
  --type text \
  --title "Исходные требования" \
  -n <NOTEBOOK_ID> \
  --json
```

### Google Drive

```bash
notebooklm source add-drive <FILE_ID> "Название" \
  --mime-type google-doc \
  -n <NOTEBOOK_ID>
```

Допустимые `mime-type` зависят от версии. Проверяй `source add-drive --help`.

### Проверка готовности

После добавления источник может обрабатываться асинхронно:

```bash
notebooklm source list -n <NOTEBOOK_ID> --json
notebooklm source wait <SOURCE_ID> -n <NOTEBOOK_ID>
notebooklm source get <SOURCE_ID> -n <NOTEBOOK_ID>
```

Не начинай важный grounded-анализ, пока ключевые sources не имеют готового статуса.

## 11. Форматы источников

Практически полезные форматы:

| Формат | Применение |
|---|---|
| `.md` | Лучший переносимый формат для транскриптов, переписок и структурированного текста |
| `.txt` | Простой текст без структуры |
| `.csv` | Табличные выгрузки, если корректно сохранены многострочные поля |
| `.pdf` | Отчёты, книги, презентационные документы |
| `.docx` | Word-документы |
| `.pptx` | Презентации |
| `.png`, `.jpg`, `.webp` | OCR и visual analysis, если поддерживается версией |
| `.epub` | Книги, если поддерживается версией |

Проблемные случаи:

- аудио лучше сначала транскрибировать в `.md` или `.txt`;
- `.xlsx` конвертировать в CSV или подключать как Google Sheets;
- `.jsonl` конвертировать в структурированный Markdown;
- URL, заканчивающийся на `.md`, может импортироваться хуже обычной HTML-страницы;
- локальный файл должен существовать на той машине, где выполняется CLI.

## 12. Grounded chat и цитаты

Базовый запрос:

```bash
notebooklm ask \
  "Назови пять основных выводов. Используй только источники и дай цитаты." \
  -n <NOTEBOOK_ID> \
  --json
```

Ограничение одним source:

```bash
notebooklm ask \
  "Сделай подробный конспект только этого документа." \
  -n <NOTEBOOK_ID> \
  -s <SOURCE_ID> \
  --json
```

Продолжение разговора:

```bash
notebooklm ask \
  "Теперь отсортируй выводы по влиянию." \
  -n <NOTEBOOK_ID> \
  -c <CONVERSATION_ID> \
  --json
```

Сохранение ответа как note, если команда поддерживается версией:

```bash
notebooklm ask \
  "Составь итоговый briefing." \
  -n <NOTEBOOK_ID> \
  --save-as-note \
  --note-title "Итоговый briefing" \
  --json
```

### Хорошая формула запроса

```text
РОЛЬ + ЗАДАЧА + ОГРАНИЧЕНИЕ ИСТОЧНИКОВ + ФОРМАТ + КРИТЕРИЙ ПРОВЕРКИ
```

Пример:

```text
Ты продуктовый аудитор. Найди 8 повторяющихся проблем пользователей.
Используй только загруженные источники. Для каждой проблемы укажи:
1. Суть.
2. Подтверждающую цитату.
3. Частоту или количество примеров, если это видно.
4. Реализовано ли решение или только обещано.
Не додумывай отсутствующие факты.
```

## 13. Персоны и команда аналитиков

Один notebook можно анализировать несколькими ролями:

| Роль | Задача |
|---|---|
| Critic | Найти дыры, противоречия и риски |
| Strategist | Приоритизировать действия по ROI и усилиям |
| Teacher | Сделать понятный учебный материал |
| Copywriter | Упаковать подтверждённые инсайты в контент |
| Engineer | Разделить реализованный код, планы и маркетинговые обещания |

Пример:

```bash
notebooklm configure \
  --persona "Ты строгий продуктовый аудитор. Отделяй факты от интерпретаций. Каждый значимый вывод подтверждай источником." \
  --response-length longer \
  -n <NOTEBOOK_ID>
```

Проверка применения роли:

```bash
notebooklm ask \
  "Какая роль тебе задана? Кратко опиши правила ответа." \
  -n <NOTEBOOK_ID> \
  --json
```

В некоторых версиях `--mode` и `--persona` могут перезаписывать друг друга. Для кастомной роли не смешивай их без проверки.

Смена persona влияет на последующие вопросы и может влиять на Studio artifacts. На общем notebook сначала сохрани текущую конфигурацию, после задачи восстанови её.

## 14. Deep Research

### Один deep pass

```bash
notebooklm source add-research \
  "Рынок AI-ассистентов для малого бизнеса, конкуренты, цены, ограничения" \
  --mode deep \
  --from web \
  --cited-only \
  -n <NOTEBOOK_ID>
```

### Набор узких fast-запросов

Для большого исследования часто полезнее несколько узких запросов:

```bash
notebooklm source add-research \
  "AI assistant onboarding failures case studies" \
  --mode fast \
  --from web \
  --import-all \
  -n <NOTEBOOK_ID>
```

Повтори для разных углов:

- рынок и категории;
- конкретные конкуренты;
- pricing;
- onboarding;
- retention;
- security;
- implementation patterns;
- failure cases.

### Проверка импорта

Не доверяй только сообщению `Imported N`. Проверяй фактический список и состояние sources:

```bash
notebooklm metadata -n <NOTEBOOK_ID> --json
notebooklm source list -n <NOTEBOOK_ID> --json
```

Классифицируй найденные источники:

- A: официальные документы, primary repos, API docs;
- B: стандарты, vendor engineering docs, whitepapers;
- C: community-примеры, статьи, GitHub issues;
- D: дубли, SEO, зеркала, нерелевантные материалы.

Не удаляй сомнительный source автоматически. Сначала зафиксируй причину и проверь его влияние.

## 15. Массовый набор источников

Рекомендуемая последовательность:

1. Разделить тему на независимые поисковые углы.
2. Выполнить несколько узких `fast` запросов.
3. Проверить реальный source count.
4. Дождаться обработки.
5. Найти дубли, ошибки и явно нерелевантные источники.
6. Удалять только по точному source ID.
7. Добрать отсутствующие углы.
8. Снова проверить status и count.
9. Если notebook достиг лимита, создать следующий том.

Команды очистки зависят от версии:

```bash
notebooklm source clean --dry-run -n <NOTEBOOK_ID>
notebooklm source clean -y -n <NOTEBOOK_ID>
```

Сначала всегда `--dry-run`, если он поддерживается.

## 16. Генерация Studio artifacts

Проверяй доступные типы:

```bash
notebooklm generate --help
```

Часто доступны:

- audio;
- video;
- slide-deck;
- quiz;
- flashcards;
- infographic;
- data-table;
- mind-map;
- report;
- revise-slide.

### Audio overview

```bash
notebooklm generate audio \
  "Краткий аудио-бриф для занятого предпринимателя" \
  --format brief \
  --language ru \
  --no-wait \
  -n <NOTEBOOK_ID>
```

### Video overview

```bash
notebooklm generate video \
  "Объясни проблему, доказательства и следующие действия" \
  --format explainer \
  --language ru \
  --no-wait \
  -n <NOTEBOOK_ID>
```

### Slide deck

```bash
notebooklm generate slide-deck \
  "Сделай презентацию: проблема, факты, варианты, решение, риски" \
  --format detailed \
  --no-wait \
  -n <NOTEBOOK_ID>
```

### Quiz

```bash
notebooklm generate quiz \
  "Создай тест на русском языке только по выбранному уроку" \
  --difficulty hard \
  --quantity standard \
  --no-wait \
  -n <NOTEBOOK_ID> \
  -s <SOURCE_ID>
```

У некоторых версий quiz не принимает `--language`. В таком случае укажи язык в prompt.

### Flashcards

```bash
notebooklm generate flashcards \
  "Сделай карточки: термин, определение, пример" \
  --difficulty medium \
  --no-wait \
  -n <NOTEBOOK_ID>
```

### Infographic

```bash
notebooklm generate infographic \
  "Покажи путь клиента и точки потери" \
  --orientation portrait \
  --detail detailed \
  --no-wait \
  -n <NOTEBOOK_ID>
```

### Mind map

```bash
notebooklm generate mind-map \
  --kind note-backed \
  --instructions "Разложи тему на проблемы, причины, решения и риски" \
  -n <NOTEBOOK_ID>
```

Не добавляй `--wait`, если установленная версия для `mind-map` его не поддерживает.

### Параллельный запуск

Если версия и квоты позволяют, запускай независимые artifacts через `--no-wait`, затем проверяй список:

```bash
notebooklm artifact list -n <NOTEBOOK_ID> --json
```

Не держи длинную SSH-сессию только ради `artifact wait`, если генерация занимает много минут. Запусти задачу, сохрани artifact ID и проверяй статус отдельными вызовами.

## 17. Скачивание artifacts

```bash
notebooklm download audio "/absolute/path/overview.mp3" \
  --latest \
  -n <NOTEBOOK_ID>
```

```bash
notebooklm download video "/absolute/path/overview.mp4" \
  --latest \
  -n <NOTEBOOK_ID>
```

```bash
notebooklm download slide-deck "/absolute/path/presentation.pptx" \
  --format pptx \
  --latest \
  -n <NOTEBOOK_ID>
```

```bash
notebooklm download quiz "/absolute/path/quiz.md" \
  --format markdown \
  --latest \
  -n <NOTEBOOK_ID>
```

Для надёжной автоматизации лучше указывать artifact ID:

```bash
notebooklm download infographic "/absolute/path/infographic.png" \
  -a <ARTIFACT_ID> \
  -n <NOTEBOOK_ID>
```

После скачивания проверь:

- файл существует;
- размер больше нуля;
- MIME или расширение соответствует типу;
- медиа открывается;
- содержимое соответствует prompt;
- выбран правильный artifact, а не случайный `--latest`.

## 18. Заметки, язык и sharing

### Notes

```bash
notebooklm note list -n <NOTEBOOK_ID>
notebooklm note create "Текст заметки" -t "Название" -n <NOTEBOOK_ID>
notebooklm note get <NOTE_ID> -n <NOTEBOOK_ID>
```

### Язык

```bash
notebooklm language list
notebooklm language get
notebooklm language set ru
```

### Sharing

Сначала проверить текущий статус:

```bash
notebooklm share status -n <NOTEBOOK_ID>
```

Изменение sharing является внешним изменением доступа. Выполняй только по явному запросу пользователя:

```bash
notebooklm share add user@example.com \
  --permission viewer \
  -n <NOTEBOOK_ID>
```

```bash
notebooklm share public --enable -n <NOTEBOOK_ID>
```

Публичная ссылка делает материалы доступными другим людям. Не включай её автоматически.

## 19. MCP setup

### Установка

В отдельном тестовом окружении:

```bash
python -m pip install "notebooklm-py[mcp]"
notebooklm-mcp --help
```

Или через `uvx`:

```bash
uvx --from "notebooklm-py[mcp]" notebooklm-mcp --help
```

MCP использует сохранённую CLI-авторизацию. Сначала выполни и проверь:

```bash
notebooklm login
notebooklm auth check --test --json
```

### STDIO

```bash
notebooklm-mcp
```

STDIO подходит для desktop-клиента, который запускает MCP как дочерний процесс.

Пример server block:

```json
{
  "mcpServers": {
    "notebooklm": {
      "command": "uvx",
      "args": [
        "--from",
        "notebooklm-py[mcp]",
        "notebooklm-mcp"
      ]
    }
  }
}
```

Путь и формат MCP-конфигурации зависят от клиента. Не записывай этот block в неизвестный config без предварительного чтения существующего файла.

### Loopback HTTP

```bash
notebooklm-mcp \
  --transport http \
  --host 127.0.0.1 \
  --port 9420
```

Правила безопасности:

- по умолчанию оставляй bind на `127.0.0.1`;
- не открывай порт в интернет;
- для non-loopback используй штатную аутентификацию версии;
- не передавай bearer token через аргументы командной строки;
- используй отдельный Google-аккаунт для remote deployment;
- считай MCP сервер полномочным интерфейсом ко всему NotebookLM-профилю.

### MCP smoke test

После подключения проверь:

1. MCP server появился в клиенте.
2. `tools/list` возвращает NotebookLM tools.
3. Read-only list notebooks работает.
4. Можно прочитать metadata тестового notebook.
5. Можно добавить безопасный тестовый source.
6. Source достигает ready.
7. Chat возвращает grounded-ответ.
8. Ошибка авторизации показывается явно.
9. Destructive tools требуют точных ID и подтверждения.
10. После перезапуска профиль выбирается однозначно.

## 20. Прикладные сценарии

### YouTube в Telegram-контент

```text
YouTube transcript
→ summary
→ 10 тезисов
→ Telegram-пост
→ hooks
→ audio briefing
→ доставка после отдельной проверки
```

Prompt:

```text
Разбери transcript как редактор. Дай краткое summary, 10 практических
тезисов, факты и инструменты, три варианта Telegram-поста, пять hooks,
идею лид-магнита и список утверждений, которые требуют внешней проверки.
```

### Встреча в задачи

```text
Transcript встречи
→ решения
→ задачи
→ ответственные
→ сроки
→ открытые вопросы
→ follow-up
```

Prompt:

```text
Извлеки только явно зафиксированные решения и задачи. Для каждой задачи
укажи ответственного, срок, подтверждающую цитату и уровень уверенности.
Не назначай ответственных и сроки, если их нет в источнике.
```

### Конкурентный радар

```text
Сайты + YouTube + презентации конкурентов
→ позиционирование
→ ICP
→ боли
→ офферы
→ proof
→ pricing
→ gaps
→ еженедельный brief
```

### Мини-курс

```text
Playlist или набор уроков
→ программа
→ конспекты
→ quiz
→ flashcards
→ slide deck
→ mind map
```

### Контент-фабрика

```text
Один экспертный источник
→ longread
→ 10 постов
→ short video scripts
→ FAQ
→ objections
→ audio overview
→ lead magnet
```

### Research concierge

```text
Тема
→ Deep Research
→ source trust map
→ briefing
→ risks
→ decision memo
→ next questions
```

## 21. Итерационный Deep Research Artifact Loop

Используй этот loop для сложных проектов, где один notebook или один research pass недостаточен.

### Шаг 1. Создать папку итерации

Пример:

```text
notebooklm_exports/
└── project-iteration-02-YYYYMMDD/
```

### Шаг 2. Собрать compound source

Включи:

- цель продукта;
- подтверждённую текущую архитектуру;
- implementation plan;
- redacted code export;
- прошлые NotebookLM answers;
- сохранённые artifacts;
- таблицу доверия к источникам;
- нерешённые вопросы.

### Шаг 3. Проверить секреты

Перед загрузкой ищи как минимум:

- Telegram bot tokens;
- строки `sk-`;
- `ghp_` и `github_pat_`;
- `AIza` и `ya29.`;
- JWT-like строки;
- private key blocks;
- `.env` значения;
- cookies и session storage.

Найденные совпадения проверяй вручную. Не загружай source-pack, пока живые секреты не удалены.

### Шаг 4. Измерить размер

Для кириллицы считай символы средствами, которые корректно работают с Unicode:

```bash
python3 -c "print(len(open('/absolute/path/source.md', encoding='utf-8').read()))"
```

Не режь CSV или экспорт по строкам, если поля содержат переносы. Используй CSV parser и собирай chunks по полным записям.

### Шаг 5. Создать новый notebook итерации

```bash
notebooklm create "Project Research Iteration 02" --use --json
notebooklm source add "/absolute/path/compound-source.md" --json
```

### Шаг 6. Добавить primary sources

Официальные docs, primary repos и API references добавляй вручную до web research.

### Шаг 7. Research gap map

Prompt:

```text
Сравни текущий проектный пакет с целевой задачей. Раздели вывод на:
1. Подтверждено источниками.
2. Не подтверждено.
3. Противоречия.
4. Критические пробелы.
5. Три research-запроса, которые реально разблокируют решение.
Не смешивай current implementation и target architecture.
```

### Шаг 8. Добрать только нужные источники

Запусти research по 1-3 темам, которые разблокируют решение. Не импортируй сотни sources ради объёма.

### Шаг 9. Architecture delta

Prompt:

```text
Составь architecture delta. Для каждого решения укажи:
current state, target state, evidence, alternatives, trade-offs, risk,
MVP или Next или Later, acceptance check.
```

### Шаг 10. Сохранить результаты

Сохраняй рядом:

- raw JSON;
- readable Markdown;
- source manifest;
- trust map;
- artifact IDs;
- prompts;
- список неподтверждённых выводов.

### Шаг 11. Следующая итерация

В новый compound source добавляй подтверждённые результаты предыдущего цикла. Слабые и устаревшие sources помечай, а не переносись с ними автоматически.

## 22. Проверка доверия

NotebookLM может смешивать:

- факты;
- интерпретации;
- маркетинговые обещания;
- будущие планы;
- внутренние знания модели.

Применяй цепочку:

```text
Ответ NotebookLM
→ цитата
→ исходный фрагмент
→ проверка primary source
→ проверка кода или live-системы
→ итоговый вывод
```

Особенно перепроверяй:

- технические возможности;
- security claims;
- цены и квоты;
- production status;
- юридические и финансовые выводы;
- права доступа;
- доставку материалов пользователю.

## 23. Troubleshooting

### Auth profile существует, но команды не работают

Симптом:

```text
token_fetch=false
auth error
unauthorized
```

Действия:

1. Не считать cookies рабочими.
2. Выполнить интерактивный `notebooklm login`.
3. Выполнить `auth check --test --json`.
4. Только после успешной проверки переносить профиль или запускать automation.

### Stream обрывается

Симптом:

```text
incomplete chunked read
peer closed connection
NETWORK_ERROR
```

Действия:

1. Сократить prompt и scope.
2. Ограничить sources через `-s`.
3. Использовать `--json` для машинной обработки.
4. Запустить `ask` локально без нестабильного proxy.
5. Не повторять paid generation вслепую.

### Artifact долго генерируется

1. Использовать `--no-wait`.
2. Сохранить artifact ID.
3. Проверять `artifact list --json` отдельными вызовами.
4. Не считать `pending` ошибкой.
5. Не запускать дубликат без проверки существующей задачи.

### Research сообщает ошибку импорта

1. Зафиксировать source count до операции.
2. Повторно получить `metadata` и `source list`.
3. Проверить, появились ли sources фактически.
4. Не доверять только строке stderr или `Imported N`.

### Notebook переполнен

1. Посмотреть рабочие, error и duplicate sources.
2. Выполнить dry-run очистки.
3. Удалить подтверждённый мусор по ID.
4. Создать второй notebook-том.
5. Перенести persona и source manifest.

### Команда отсутствует

Причина обычно в версии CLI.

```bash
notebooklm --version
notebooklm <group> --help
```

Не обновляй production автоматически. Сначала сверяй release notes и тестируй отдельное окружение.

## 24. Red flags

- Агент утверждает, что source загружен, не проверив его статус.
- Ответ NotebookLM выдается за live production proof.
- В notebook загружены секреты или приватные данные без согласования.
- Deep Research imports используются без source trust audit.
- Массовое удаление выполняется по title, а не по точным IDs.
- Публичный sharing включается автоматически.
- Один и тот же paid artifact запускается повторно без status check.
- `--latest` используется там, где критически важен точный artifact.
- Персона изменена на общем notebook и не восстановлена.
- MCP открыт на внешнем интерфейсе без строгой аутентификации.
- Рабочий CLI обновлён ради MCP без тестовой миграции.
- NotebookLM смешал current state и target architecture, а вывод не перепроверили.

## 25. Smoke tests

### CLI smoke

```bash
notebooklm --version
notebooklm auth check --test --json
notebooklm list --limit 1 --json
```

Ожидается:

- команда найдена;
- auth test успешен;
- notebook list возвращает валидный JSON.

### Source smoke

1. Создать безопасный тестовый notebook.
2. Добавить короткий текстовый source с уникальным фактом.
3. Дождаться ready.
4. Задать вопрос по уникальному факту.
5. Убедиться, что ответ содержит правильную цитату.

### Artifact smoke

1. Запустить недорогой quiz или mind map.
2. Проверить artifact ID и статус.
3. Скачать результат.
4. Проверить размер, тип и содержимое файла.

### MCP smoke

1. Проверить `tools/list`.
2. Вызвать read-only notebook list.
3. Получить metadata тестового notebook.
4. Добавить безопасный source.
5. Получить grounded answer.
6. Проверить понятную ошибку при неверной авторизации.

## 26. Output contract для агента

После выполнения задачи сообщи:

- где выполнено: локальный CLI, VPS-обёртка или MCP;
- версия `notebooklm-py`;
- notebook ID и название;
- source IDs и их финальные статусы;
- какие вопросы были заданы;
- какие artifacts созданы;
- artifact IDs и статусы;
- абсолютные пути скачанных файлов;
- какие проверки выполнены;
- какие утверждения остаются неподтверждёнными;
- менялся ли sharing;
- были ли внешние отправки или публикации.

Не сообщай "готово", если source остался pending, artifact не скачан, файл не проверен или доставка не подтверждена.

## 27. Быстрые prompt-шаблоны

### Research brief

```text
Подготовь decision brief только по источникам:
1. Краткий вывод.
2. Подтверждённые факты.
3. Противоречия.
4. Риски.
5. Возможности.
6. Что требует внешней проверки.
7. Следующие три действия.
Каждый значимый факт сопровождай цитатой.
```

### Конкурентный анализ

```text
Для каждого конкурента укажи: ICP, позиционирование, боли, оффер,
pricing, proof, CTA, сильные стороны, слабые стороны и незакрытые gaps.
Отделяй прямые факты от интерпретаций. Не заполняй отсутствующие данные.
```

### Технический аудит

```text
Разделяй четыре категории:
1. Реализовано в коде.
2. Описано в документации.
3. Запланировано.
4. Маркетинговое обещание.
Для каждого вывода укажи источник и уровень уверенности.
```

### Telegram-пост

```text
Напиши Telegram-пост до 3000 символов: сильный hook, 5-7 тезисов,
практический вывод и CTA. Используй только подтверждённые факты.
Отдельно перечисли утверждения, которые нельзя публиковать без проверки.
```

### Audio briefing

```text
Сделай сценарий аудио-брифа на 5-8 минут для занятого предпринимателя:
почему тема важна, пять инсайтов, риски, одно следующее действие и takeaway.
Не преувеличивай выводы источников.
```

## Продвинутый серверный контур Hardcoding PRO

Если задача требует NotebookLM 24/7 на VPS, REST API для n8n/ботов или MCP-сервиса для нескольких AI-агентов, сначала прочитай отдельную публичную заметку:

- GitHub: https://github.com/PavelDumbrao/nblm-assets/blob/main/guides/notebooklm-vps-rest-mcp-android.md
- Raw Markdown: https://raw.githubusercontent.com/PavelDumbrao/nblm-assets/main/guides/notebooklm-vps-rest-mcp-android.md

Не переходи на серверный контур автоматически. Для обычной личной работы оставайся на локальном CLI + SKILL.md. На VPS сначала делай live-аудит, используй отдельное Python-окружение, не публикуй REST/MCP наружу по умолчанию и проверяй работу реальным business-call, а не только health endpoint.

