# NotebookLM на VPS 24/7: REST API, MCP, Android backend и серверная авторизация

> **Продвинутый уровень Hardcoding PRO.** Сначала пройди основной практикум NotebookLM на своём компьютере. Этот материал нужен, когда NotebookLM должен работать круглосуточно на VPS, обслуживать n8n, Telegram-ботов, внутренние приложения или несколько ИИ-агентов.
>
> Публичная версия обезличена: конкретный host-id, локальные пути пользователя и лишние детали частной инфраструктуры убраны. Секретов, cookies, master token, REST bearer и приватных ключей здесь нет.
>
> **Простыми словами:**
> - REST API — HTTP-интерфейс для n8n, ботов и приложений.
> - MCP — набор инструментов NotebookLM для ИИ-агентов.
> - Android backend — серверный режим notebooklm-py, который подходит для постоянной работы без браузера на VPS.
> - systemd — механизм Linux, который держит сервис запущенным после перезагрузки.
> - localhost / 127.0.0.1 — сервис доступен только внутри самого VPS, а не всему интернету.


Дата проверенного развёртывания: 19 сентября 2026 года.

Этот гайд описывает реальный контур, который был установлен и проверен на VPS `<YOUR_VPS>`. Он объясняет не только команды, но и причины архитектурных решений, особенности авторизации Google и ошибки, из-за которых простой на вид setup занял намного больше времени.

Секреты, cookie values, master token, REST bearer и приватные ключи в документ не включены.

## Что получилось

На Hostinger работают два localhost-сервиса над одним профилем NotebookLM:

```text
Google NotebookLM
        ↑
Android gRPC backend
        ↑
master_token.json
        ↑
┌───────────────────────────┐
│ Hostinger <YOUR_VPS>      │
│                           │
│ REST  127.0.0.1:9421      │ ← n8n, внутренние сервисы, скрипты
│ MCP   127.0.0.1:9420/mcp  │ ← Codex, Claude, другие MCP-клиенты
│ CLI   notebooklm 0.8.2    │ ← ручные операции и диагностика
└───────────────────────────┘
```

Подтверждённые проверки:

- master token создан через Comet;
- локальный bootstrap увидел реальные блокноты;
- Android CLI на VPS прочитал реальный блокнот;
- REST `/healthz` вернул `200`;
- REST `/v1/notebooks` с bearer вернул `200` и реальные блокноты;
- тот же REST route без bearer вернул `401`;
- MCP handshake прошёл;
- MCP опубликовал 38 tools;
- MCP `server_info` и `notebook_list` прошли без ошибок;
- systemd services активны и включены в автозапуск;
- оба процесса слушают только `127.0.0.1`;
- `NRestarts=0` при финальной проверке;
- все все зависимости VPS runtime совместимы.

## Зачем понадобился этот контур

### CLI

CLI остаётся самым понятным способом ручной работы:

```bash
notebooklm list
notebooklm source search "запрос"
notebooklm research discover "тема"
notebooklm generate audio
```

Он полезен для диагностики, cron и пошаговых скриптов. Но постоянный сервис не должен запускать новый CLI-процесс на каждый внутренний HTTP-запрос.

### REST API

REST нужен для систем, которые умеют HTTP, но не работают с MCP:

- n8n;
- Telegram-боты;
- внутренние backend-сервисы;
- cron и webhook pipelines;
- панели управления;
- собственные приложения.

Пример логики будущего n8n workflow:

```text
Webhook / Schedule
→ REST list/create notebook
→ add source
→ wait until ready
→ grounded chat or research
→ generate artifact
→ download or deliver
```

REST сервер держит один NotebookLM client в памяти, поэтому не создаёт новый auth-контур на каждый запрос.

### MCP

MCP нужен агентам. Вместо сборки shell-команды агент получает типизированные инструменты:

- `notebook_list`;
- `source_add`;
- `source_wait`;
- `chat_ask`;
- `research_start`;
- `studio_generate`;
- `studio_download`;
- другие инструменты, всего 38 в `0.8.2`.

Это удобнее для Codex и Claude, потому что аргументы, статусы и ошибки передаются структурированно.

### Android backend

Обычный Web backend работает через browser cookies и внутренние Web RPC. Cookies требуют обновления и могут протухать в неудобный момент.

Android backend использует native gRPC NotebookLM и короткоживущий bearer, который создаётся из долговечного master token. После одноразового bootstrap VPS работает без браузера.

Это особенно полезно для:

- постоянно работающего сервиса;
- VPS без GUI;
- systemd;
- длительных MCP-сессий;
- автоматического восстановления авторизации.

## Почему использовалась версия 0.8.2

`notebooklm-py 0.8.2` является актуальным стабильным релизом на дату установки.

Ветка `0.8.x` добавила:

- MCP;
- REST server;
- Android backend;
- headless master-token auth;
- collections;
- `source search`;
- `research discover`;
- async chat status/cancel;
- notebook copy;
- улучшенную работу с citations и metadata.

Важно: package release стабильный, но tool schemas MCP и REST routes пока считаются preview. Версию нужно фиксировать явно.

Официальные материалы:

- [Release notes](https://github.com/teng-lin/notebooklm-py/releases)
- [Installation guide](https://github.com/teng-lin/notebooklm-py/blob/main/docs/installation.md)
- [Configuration](https://github.com/teng-lin/notebooklm-py/blob/main/docs/configuration.md)
- [MCP guide](https://github.com/teng-lin/notebooklm-py/blob/main/docs/mcp-guide.md)

## Исходное состояние

На VPS уже существовал старый контур:

```text
/opt/notebooklm/venv
/usr/local/bin/nblm
notebooklm-py 0.7.2
```

Live-аудит показал:

- legacy CLI `0.7.2`;
- Web auth `token_fetch=false`;
- `nblm-refresh.timer` отключён;
- порт `8000` занят `<another-service>`;
- диск заполнен достаточно высоко, поэтому перед установкой проверили свободное место;
- памяти для небольшого Python runtime достаточно;
- старые NotebookLM reports и research artifacts должны быть сохранены.

Поэтому старое окружение не обновлялось поверх. Новая версия установлена side-by-side.

Это дало простой rollback: старый runtime и wrapper остались на месте.

## Установка на Mac

Пакет с MCP нельзя ставить в общий Python, где уже живут FastAPI, Gradio или другие приложения. `fastmcp` может потребовать версии `starlette` и `pydantic`, несовместимые с ними.

Изолированная установка через `uv tool`:

```bash
uv tool install --force \
  "notebooklm-py[mcp,browser,cookies,markdown,android,headless]==0.8.2"
```

Проверка:

```bash
notebooklm --version
notebooklm-mcp --help
notebooklm-server --help
```

Если нужен bundled Chromium для обычных browser flows:

```bash
~/.local/share/uv/tools/notebooklm-py/bin/python \
  -m playwright install chromium
```

## Обычная Web-сессия на Mac

Сначала была обновлена обычная cookie-сессия:

```bash
notebooklm login --browser chrome
notebooklm auth check --test --passive --json
```

Успешный контракт:

```json
{
  "status": "ok",
  "checks": {
    "token_fetch": true
  }
}
```

Затем были проверены новые функции:

```bash
notebooklm list --limit 5 --json

notebooklm source search \
  "ИИ-агенты и автоматизация" \
  -n <NOTEBOOK_ID> \
  --limit 3 \
  --json

notebooklm research discover \
  "NotebookLM REST API Android backend headless automation security" \
  -n <NOTEBOOK_ID> \
  --json
```

`research discover` создал research-run, но источники в блокнот не импортировал. Импорт является отдельной операцией.

Локальный MCP smoke:

```text
initialize
→ tools/list = 38
→ server_info = success
```

## Как создавался master token через Comet

### Почему обычный вход недостаточен

Android backend не использует обычный `storage_state.json` как основной credential. Ему нужен:

```text
master_token.json
```

Это долговечный Google credential уровня Android device session. Он создаёт короткоживущие OAuth bearer tokens и при необходимости может заново создать Web cookies.

Обычная команда:

```bash
notebooklm --profile hostinger login \
  --master-token \
  --account <GOOGLE_EMAIL>
```

В нашем случае пользователь явно попросил провести вход через Comet.

### Важный нюанс: «Принимаю» и вечная загрузка

Google EmbeddedSetup после подтверждения условий может остаться на экране загрузки. Это не обязательно означает ошибку.

Ровно такой случай описан в:

- [issue #2350](https://github.com/teng-lin/notebooklm-py/issues/2350#issuecomment-5585734574)
- [PR #2360](https://github.com/teng-lin/notebooklm-py/pull/2360)
- [PR #2414](https://github.com/teng-lin/notebooklm-py/pull/2414)
- [gpsoauth alternative flow](https://github.com/simon-weber/gpsoauth#alternative-flow)

Документация `gpsoauth` прямо говорит: после `I agree` страница может показывать loading forever; нужно взять cookie `oauth_token` и продолжить обмен.

В `0.8.2` browser polling зависит от JavaScript страницы:

```python
page.evaluate("Date.now()")
```

Если JavaScript страницы завис, CLI также может зависнуть и не дойти до обмена.

PR #2360 переносит таймер на host process, а PR #2414 добавляет безопасную диагностику. Эти изменения приняты после выпуска `0.8.2` и отсутствуют в установленном PyPI wheel.

### Что не являлось причиной

Была проверена selective VPN-схема:

- `notebook.google`;
- `accounts.google.com`;
- `googleapis.com`;
- Google Play;
- Android auth endpoints.

Все домены шли через одну группу. Health probes проходили через VPS и несколько VPN-выходов. Смена egress не устранила зависание consent page.

Вывод: зависший экран сам по себе не доказывает проблему VPN.

### Проверенный browser flow

1. Открыть в Comet:

   ```text
   https://accounts.google.com/EmbeddedSetup
   ```

2. Ввести email аккаунта.
3. Пройти Google identity confirmation.
4. Выбрать владельца устройства.
5. Подтвердить условия Google и Google Play.
6. Не ждать закрытия loading page.
7. Через разрешённый browser API прочитать cookies только для:

   ```text
   https://accounts.google.com/
   ```

8. Выбрать cookie строго по условию:

   ```text
   name == "oauth_token"
   ```

9. Сравнить его с token, существовавшим до нового flow. После согласия значение должно измениться.
10. Обменять новый одноразовый token сразу.

Cookie одноразовый и живёт недолго. Повторная отправка старого значения возвращает:

```text
BadAuthentication
```

### Как token передавался без печати в shell

Browser clipboard и системный `pbpaste` оказались разными буферами. Поэтому передача через:

```bash
pbpaste
```

читала посторонний текст и создавала ложный `BadAuthentication`.

Проверенный способ:

1. На Mac поднят одноразовый HTTP listener только на `127.0.0.1`.
2. Listener создавал случайный URL path.
3. Страница содержала password input.
4. POST принимался только с правильным `Origin`.
5. Request logging был выключен.
6. Размер запроса был ограничен.
7. Значение передавалось из Comet прямо в локальную форму.
8. Listener сразу вызывал:

   ```python
   notebooklm.auth.master_token_bootstrap(...)
   ```

9. В stdout возвращались только:

   ```text
   ok
   notebook_count
   master_token_exists
   ```

10. Listener был остановлен после обмена.

Результат:

```text
bootstrap_ok=true
notebook_count=84
master_token_exists=true
```

### Почему нельзя парсить DevTools текст на глаз

В DevTools было поле Filter со строкой `oauth_token`. Ранний парсер нашёл первое текстовое совпадение и принял соседнюю подпись за значение cookie.

Правильный путь:

- читать структурированный browser cookie object;
- выбирать exact cookie name;
- проверять prefix/length только в памяти;
- не выводить полный cookie table;
- не выводить token;
- не сохранять его в shell history.

## Безопасность master token

Master token:

- долговечный;
- даёт возможность выпускать short-lived Google tokens;
- может пережить смену пароля;
- имеет больший blast radius, чем cookie snapshot.

Права локального файла:

```bash
chmod 600 ~/.notebooklm/profiles/hostinger/master_token.json
```

На VPS:

```text
owner: notebooklm:notebooklm
mode: 0600
directory mode: 0700
```

Для production-развёртывания безопаснее использовать отдельный Google-аккаунт, предназначенный именно для серверной автоматизации.

Нельзя:

- печатать token;
- отправлять его в Telegram;
- хранить в Git;
- передавать через command-line arguments;
- помещать в systemd unit;
- хранить в документации;
- считать смену пароля гарантированным отзывом.

Для отзыва нужно удалить соответствующую device/session в Google Account Security.

## Установка нового runtime на Hostinger

Новый runtime создан отдельно:

```bash
uv venv /opt/notebooklm/venv-082 --python /usr/bin/python3.12

uv pip install \
  --python /opt/notebooklm/venv-082/bin/python \
  "notebooklm-py[server,mcp,android,headless,markdown]==0.8.2"
```

Проверка:

```bash
/opt/notebooklm/venv-082/bin/notebooklm --version
/opt/notebooklm/venv-082/bin/notebooklm-mcp --help
/opt/notebooklm/venv-082/bin/notebooklm-server --help

uv pip check --python /opt/notebooklm/venv-082/bin/python
```

Runtime занимает около 138 МБ.

## Отдельный Linux user и закрытые каталоги

Создан system user:

```text
notebooklm
```

Layout:

```text
/opt/notebooklm/venv-082/                  runtime
/var/lib/notebooklm/.notebooklm/           NOTEBOOKLM_HOME
/var/lib/notebooklm/.notebooklm/profiles/hostinger/
/etc/notebooklm/rest.token                 REST bearer
```

Права:

```text
/var/lib/notebooklm/.notebooklm/                     0700
/var/lib/notebooklm/.notebooklm/profiles/hostinger/  0700
master_token.json                                   0600
storage_state.json                                  0600
/etc/notebooklm/rest.token                           0600
```

REST bearer был создан на сервере и его значение не выводилось.

## Передача credential на VPS

Передавался только файл:

```text
master_token.json
```

Общий принцип:

```bash
scp -p \
  ~/.notebooklm/profiles/hostinger/master_token.json \
  root@<HOSTINGER_IP>:/var/lib/notebooklm/.notebooklm/profiles/hostinger/master_token.json
```

После передачи:

```bash
chown notebooklm:notebooklm \
  /var/lib/notebooklm/.notebooklm/profiles/hostinger/master_token.json

chmod 600 \
  /var/lib/notebooklm/.notebooklm/profiles/hostinger/master_token.json
```

Не передавалась вся browser profile директория.

## Первый запуск Android backend

Первый прямой Android list вернул:

```text
AUTH_REQUIRED
```

Причина: профиль содержал master token, но ещё не содержал `storage_state.json`. В текущей `0.8.2` CLI loader проверил storage раньше, чем открыл Android backend.

Сначала был выполнен:

```bash
runuser -u notebooklm -- env \
  NOTEBOOKLM_HOME=/var/lib/notebooklm/.notebooklm \
  /opt/notebooklm/venv-082/bin/notebooklm \
  --profile hostinger \
  auth refresh --json
```

Он создал Web storage из master token без браузера.

Ответ содержал:

```json
{
  "status": "ok",
  "verified": false
}
```

Нюанс: `verified:false` не означал, что Android backend не работает. Это лишь результат конкретного Web refresh шага.

После него Android read прошёл:

```bash
runuser -u notebooklm -- env \
  NOTEBOOKLM_HOME=/var/lib/notebooklm/.notebooklm \
  /opt/notebooklm/venv-082/bin/notebooklm \
  --profile hostinger \
  --backend android \
  list --limit 1 --json
```

Команда вернула реальный пользовательский блокнот.

## Systemd REST service

Файл:

```text
/etc/systemd/system/notebooklm-rest.service
```

Ключевые параметры:

```ini
[Unit]
Description=NotebookLM REST API 0.8.2 (Android backend)
Wants=network-online.target
After=network-online.target
ConditionPathExists=/var/lib/notebooklm/.notebooklm/profiles/hostinger/master_token.json

[Service]
Type=simple
User=notebooklm
Group=notebooklm
Environment=NOTEBOOKLM_HOME=/var/lib/notebooklm/.notebooklm
ExecStart=/opt/notebooklm/venv-082/bin/notebooklm-server --host 127.0.0.1 --port 9421 --token-file /etc/notebooklm/rest.token --profile hostinger --backend android --log-level INFO
Restart=on-failure
RestartSec=10
UMask=0077
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectControlGroups=true
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
ReadWritePaths=/var/lib/notebooklm
ReadOnlyPaths=/opt/notebooklm/venv-082 /etc/notebooklm/rest.token

[Install]
WantedBy=multi-user.target
```

Почему порт `9421`: порт `8000`, используемый REST по умолчанию, уже занят `<another-service>`.

Почему `--token-file`: bearer не попадает в argv и unit text.

## Systemd MCP service

Файл:

```text
/etc/systemd/system/notebooklm-mcp.service
```

Ключевые параметры:

```ini
[Unit]
Description=NotebookLM MCP HTTP 0.8.2 (Android backend)
Wants=network-online.target
After=network-online.target
ConditionPathExists=/var/lib/notebooklm/.notebooklm/profiles/hostinger/master_token.json

[Service]
Type=simple
User=notebooklm
Group=notebooklm
Environment=NOTEBOOKLM_HOME=/var/lib/notebooklm/.notebooklm
ExecStart=/opt/notebooklm/venv-082/bin/notebooklm-mcp --transport http --host 127.0.0.1 --port 9420 --profile hostinger --backend android --log-level INFO
Restart=on-failure
RestartSec=10
UMask=0077
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectControlGroups=true
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
ReadWritePaths=/var/lib/notebooklm
ReadOnlyPaths=/opt/notebooklm/venv-082

[Install]
WantedBy=multi-user.target
```

MCP оставлен без публичного bind. Localhost transport не требует отдельного bearer в этой конфигурации.

## Запуск сервисов

```bash
systemd-analyze verify \
  /etc/systemd/system/notebooklm-rest.service \
  /etc/systemd/system/notebooklm-mcp.service

systemctl daemon-reload

systemctl enable --now \
  notebooklm-rest.service \
  notebooklm-mcp.service
```

Проверка состояния:

```bash
systemctl show \
  notebooklm-rest.service \
  notebooklm-mcp.service \
  -p Id -p ActiveState -p SubState -p NRestarts -p MemoryCurrent

ss -ltnp | rg ':(9420|9421)\b'
```

Ожидается:

```text
127.0.0.1:9420
127.0.0.1:9421
NRestarts=0
```

## REST smoke

Bearer читается процессом из:

```text
/etc/notebooklm/rest.token
```

Проверки:

```text
GET /healthz                     → 200
GET /v1/notebooks + Bearer       → 200
GET /v1/notebooks без Bearer     → 401
```

В финальной проверке REST вернул реальные блокноты.

`/healthz` показывает только liveness процесса. Он не доказывает, что Google auth и чтение блокнотов работают. Поэтому обязательна проверка `/v1/notebooks`.

## MCP smoke

Проверялось реальное Streamable HTTP соединение:

```text
initialize
→ tools/list
→ server_info
→ notebook_list
```

Результат:

```text
tool_count=38
server_info is_error=false
notebook_list is_error=false
```

Наличие 38 tools само по себе не доказывает Google auth. Поэтому нужен хотя бы один read-only business call, например `notebook_list`.

## Доступ с Mac через SSH tunnel

Сервисы не опубликованы в интернет. Для ручной работы можно открыть локальный tunnel:

```bash
ssh -N \
  -L 19421:127.0.0.1:9421 \
  -L 19420:127.0.0.1:9420 \
  root@<HOSTINGER_IP>
```

После этого:

```text
REST: http://127.0.0.1:19421
MCP:  http://127.0.0.1:19420/mcp
```

Для постоянного использования лучше создать отдельного ограниченного SSH user без root и без Docker/sudo.

## Команда нового CLI на VPS

```bash
runuser -u notebooklm -- env \
  NOTEBOOKLM_HOME=/var/lib/notebooklm/.notebooklm \
  /opt/notebooklm/venv-082/bin/notebooklm \
  --profile hostinger \
  --backend android \
  <COMMAND>
```

Пример:

```bash
runuser -u notebooklm -- env \
  NOTEBOOKLM_HOME=/var/lib/notebooklm/.notebooklm \
  /opt/notebooklm/venv-082/bin/notebooklm \
  --profile hostinger \
  --backend android \
  list --limit 5 --json
```

Старый `/usr/local/bin/nblm` всё ещё запускает `0.7.2`. Его нельзя использовать как доказательство состояния нового backend.

## Что ещё не настроено

На момент завершения не сделаны:

- публичный DNS;
- reverse proxy;
- внешний OAuth для MCP;
- прямое подключение Codex MCP;
- n8n credentials/workflows;
- доступ из n8n Docker network к host loopback;
- миграция старого cron-дайджеста;
- переключение legacy wrapper `nblm` на `0.8.2`.

Это отдельные задачи. Особенность n8n: `127.0.0.1` внутри контейнера указывает на сам контейнер, а не на Hostinger host. Для n8n нужен отдельный ограниченный сетевой маршрут или internal proxy.

## Основные ошибки и чему они научили

### Ошибка 1: считать loading screen сетевым сбоем

Экран `#close` является известным поведением EmbeddedSetup. По нему нельзя делать вывод о VPN.

Правильно: проверить точный cookie, его изменение и результат exchange.

### Ошибка 2: бесконечно повторять login

`oauth_token` одноразовый. Старое значение даёт `BadAuthentication`.

Правильно: открыть новый EmbeddedSetup flow, завершить consent, получить изменившийся token и сразу обменять.

### Ошибка 3: парсить accessibility tree по первой строке

Текст `oauth_token` мог быть значением DevTools Filter, а не именем cookie row.

Правильно: использовать structured browser cookie API и exact name match.

### Ошибка 4: считать browser clipboard системным clipboard

Virtual browser clipboard и macOS `pbpaste` вернули разные значения.

Правильно: передавать credential внутри одного доверенного контура, например через одноразовый localhost POST.

### Ошибка 5: печатать полный cookie table при диагностике

Accessibility snapshot DevTools может содержать значения cookies. Такой вывод нельзя использовать для обычной диагностики.

Правильно: возвращать только безопасные признаки:

```text
present
changed
length
error type
```

В этой сессии значения Web cookies случайно попали в технический вывод. Рекомендуется отозвать затронутую Web-сессию Google. Master token в вывод не попадал.

### Ошибка 6: ставить MCP extra в общий Python

`fastmcp` обновил shared `starlette` и `pydantic`, вызвав конфликты с FastAPI и Gradio.

Правильно: `uv tool` на Mac и отдельный venv на VPS.

### Ошибка 7: занять дефолтный порт

REST хотел `8000`, но порт уже принадлежал Telegram API Engine.

Правильно: live audit портов, затем отдельный `9421`.

### Ошибка 8: считать `healthz` полной проверкой

Процесс может быть жив, а Google auth не работать.

Правильно: authenticated read к `/v1/notebooks` и реальный MCP `notebook_list`.

### Ошибка 9: запускать Android list до первичного refresh

Master-only профиль вернул `AUTH_REQUIRED`, потому что loader ожидал storage file.

Правильно: один раз выполнить `auth refresh`, затем проверить Android list.

### Ошибка 10: считать `verified:false` провалом

`auth refresh` создал storage, но сообщил `verified:false`. Android list после этого успешно прочитал данные.

Правильно: проверять конечный бизнес-вызов, а не один промежуточный флаг.

## Диагностика

### Android CLI

```bash
runuser -u notebooklm -- env \
  NOTEBOOKLM_HOME=/var/lib/notebooklm/.notebooklm \
  /opt/notebooklm/venv-082/bin/notebooklm \
  --profile hostinger \
  --backend android \
  list --limit 1 --json
```

### Services

```bash
systemctl status notebooklm-rest.service --no-pager
systemctl status notebooklm-mcp.service --no-pager
journalctl -u notebooklm-rest.service -n 100 --no-pager
journalctl -u notebooklm-mcp.service -n 100 --no-pager
```

Не выводить DEBUG auth logs без redaction.

### Ports

```bash
ss -ltnp | rg ':(9420|9421)\b'
```

Ожидается только `127.0.0.1`.

### Permissions

```bash
stat -c '%a %U:%G %n' \
  /var/lib/notebooklm/.notebooklm/profiles/hostinger/master_token.json \
  /var/lib/notebooklm/.notebooklm/profiles/hostinger/storage_state.json \
  /etc/notebooklm/rest.token
```

Ожидается `600 notebooklm:notebooklm`.

## Обновление версии

Не обновлять runtime вслепую поверх работающего.

Безопасная схема:

```text
venv-082
→ новый venv с новой версией
→ dependency check
→ read-only CLI smoke
→ временные порты
→ REST/MCP smoke
→ переключение units
→ короткое наблюдение
→ сохранение предыдущего venv для rollback
```

MCP schemas и REST routes могут меняться между minor releases. После обновления нужно заново проверить tool count, schemas и используемые routes.

## Откат

Остановить новые сервисы:

```bash
systemctl disable --now \
  notebooklm-rest.service \
  notebooklm-mcp.service
```

Legacy runtime не перезаписывался:

```text
/opt/notebooklm/venv
/usr/local/bin/nblm
```

Удаление master token не является обязательной частью технического rollback. Если требуется отозвать credential, нужно удалить соответствующую Google device/session. Это отдельная операция с аккаунтом.

## Публичные материалы Hardcoding PRO

- Основной NotebookLM SKILL.md: https://github.com/PavelDumbrao/nblm-assets/blob/main/skills/notebooklm/SKILL.md
- Чистый SKILL.md для ИИ-агента: https://raw.githubusercontent.com/PavelDumbrao/nblm-assets/main/skills/notebooklm/SKILL.md
- Эта заметка: https://github.com/PavelDumbrao/nblm-assets/blob/main/guides/notebooklm-vps-rest-mcp-android.md

## Итоговая рабочая модель

```text
Comet используется только для одноразового bootstrap или будущего восстановления.

Hostinger работает постоянно через master token и Android backend.

REST обслуживает HTTP-интеграции.

MCP обслуживает AI-агентов.

CLI остаётся инструментом диагностики и ручных операций.
```
