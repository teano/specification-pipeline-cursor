# specification-pipeline-cursor

Cursor-пакет для подготовки agent-ready технических спецификаций: создание,
продолжение, review и нормализация Markdown-спецификаций до состояния,
пригодного для реализации разработчиком или coding agent.

## Что внутри

Репозиторий повторяет структуру глобального пользовательского пространства Cursor:

```text
README.md
commands/
  command-specification.md
  command-specification-help.md
skills/
  skill-specification-pipeline/
    SKILL.md
    router/
    shared/
    review-profiles/
    modes/
    policies/
```

Entrypoint после установки:

```text
/command-specification
```

Skill entrypoint:

```text
@skill-specification-pipeline
```

## Возможности

- создание новой спецификации с канонической структурой;
- продолжение существующего `.md` файла;
- grounded capture новых requirements, assumptions, risks и open questions;
- light/full review без изменения файла до явной команды применить правки;
- нормализация до implementation-ready вида с `REQ-*`, `AC-*`, registry,
  traceability matrix и readiness verdict;
- жесткая граница записи: пакет работает только с выбранным Markdown-файлом
  спецификации и не меняет код, ассеты, конфиги, тесты или project metadata.

## Ручная установка в глобальное пространство Cursor

1. Скачайте или распакуйте этот репозиторий в любую временную папку.
2. Скопируйте `commands/` и `skills/skill-specification-pipeline/` в глобальную
   пользовательскую папку Cursor:

```powershell
$cursorHome = Join-Path $env:USERPROFILE ".cursor"

New-Item -ItemType Directory -Force -Path `
  (Join-Path $cursorHome "commands"), `
  (Join-Path $cursorHome "skills")

Copy-Item -Path ".\commands\command-specification.md" `
  -Destination (Join-Path $cursorHome "commands\command-specification.md") `
  -Force

Copy-Item -Path ".\commands\command-specification-help.md" `
  -Destination (Join-Path $cursorHome "commands\command-specification-help.md") `
  -Force

Copy-Item -Path ".\skills\skill-specification-pipeline" `
  -Destination (Join-Path $cursorHome "skills") `
  -Recurse -Force
```

После установки должны существовать файлы:

```text
%USERPROFILE%\.cursor\commands\command-specification.md
%USERPROFILE%\.cursor\commands\command-specification-help.md
%USERPROFILE%\.cursor\skills\skill-specification-pipeline\SKILL.md
```

Не копируйте пакет в каждый проект. В проекте нужна только папка или файл
спецификаций, например `.cursor/specs/...` или `.cursor/tasks/...`.

## Установка через git в глобальное пространство Cursor

Репозиторий уже повторяет структуру глобальной папки Cursor (`commands/` и
`skills/`), поэтому самый прямой вариант такой:

```powershell
git clone https://github.com/teano/specification-pipeline-cursor.git "$env:USERPROFILE\.cursor"
```

Этот способ подходит, если `%USERPROFILE%\.cursor` еще не существует или пустая.
Если в глобальной папке Cursor уже есть ваши команды, rules или другие skills,
`git clone` в эту папку не подойдет: Git не клонирует в непустой каталог, а
делать всю `.cursor` checkout-ом одного пакета обычно неудобно.

В таком случае используйте отдельный checkout и скопируйте файлы в глобальное
пространство Cursor:

```powershell
git clone https://github.com/teano/specification-pipeline-cursor.git "$env:TEMP\specification-pipeline-cursor"

Copy-Item "$env:TEMP\specification-pipeline-cursor\commands\command-specification.md" "$env:USERPROFILE\.cursor\commands\command-specification.md" -Force
Copy-Item "$env:TEMP\specification-pipeline-cursor\commands\command-specification-help.md" "$env:USERPROFILE\.cursor\commands\command-specification-help.md" -Force
Copy-Item "$env:TEMP\specification-pipeline-cursor\skills\skill-specification-pipeline" "$env:USERPROFILE\.cursor\skills" -Recurse -Force
```

Обновление при прямом clone в `.cursor`:

```powershell
git -C "$env:USERPROFILE\.cursor" pull
```

При установке через временный checkout повторите копирование после `git pull`.

После установки или обновления перезапустите Cursor или откройте новый chat,
чтобы Cursor перечитал глобальные commands/skills.

## Использование

Создать новую спецификацию:

```text
/command-specification new "Daily rewards" .cursor/specs -- generate complete technical spec from the GDD below: ...
```

Продолжить существующую спецификацию:

```text
/command-specification continue .cursor/specs/daily-rewards.md -- add fragment: daily reward can be claimed once per calendar day
```

Быстрое review:

```text
/command-specification continue .cursor/specs/daily-rewards.md -- quick review
```

Полное review:

```text
/command-specification continue .cursor/specs/daily-rewards.md -- full pre-implementation review
```

Нормализация:

```text
/command-specification continue .cursor/specs/daily-rewards.md -- normalize into implementation-ready markdown
```

## Runtime inputs

`/command-specification` не запускает skill, пока не определены обязательные
значения:

| Binding | Описание |
| --- | --- |
| `SPECIFICATION_PATH` | Путь к Markdown-файлу спецификации. Для `new` файл создается до маршрутизации. |
| `SPECIFICATION_DIR` | Родительская папка `SPECIFICATION_PATH`; может быть создана только при `new`. |
| `SPECIFICATION_TITLE` | Заголовок из первого `#` в документе или title из `new`. |
| `USER_LANGUAGE` | Язык пользовательского запроса и тела спецификации, например `Russian` или `English`. |
| `USER_REQUEST` | Рабочий запрос: что нужно сделать со спецификацией. |

## Ключевые файлы

| Файл | Назначение |
| --- | --- |
| `commands/command-specification.md` | Cursor slash-command entrypoint. |
| `commands/command-specification-help.md` | Help для неверного или неполного запуска. |
| `skills/skill-specification-pipeline/SKILL.md` | Главный контракт skill и runtime flow. |
| `skills/skill-specification-pipeline/router/router-map.md` | Маршрутизация пользовательских запросов по режимам. |
| `skills/skill-specification-pipeline/shared/specification-document-regulation.md` | Каноническая форма спецификации и правила документа. |
| `skills/skill-specification-pipeline/shared/pass-loading-policy.md` | Какие `PASS-*` проверки запускать для каждого сценария. |
| `skills/skill-specification-pipeline/review-profiles/review-light.md` | Набор проверок для быстрого review. |
| `skills/skill-specification-pipeline/review-profiles/review-full.md` | Набор проверок для полного review. |
| `skills/skill-specification-pipeline/modes/*/SKILL.md` | Контракты конкретных режимов. |
