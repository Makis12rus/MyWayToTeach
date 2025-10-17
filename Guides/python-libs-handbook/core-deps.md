# 🧱 Основные зависимости проекта: изоляция, фиксация версий и спокойная жизнь (полная версия)

> Гайд для тех, кто устал объяснять чужому ноутбуку, почему «у меня дома работало».

---

## 📝 Примечания

* Все инструменты устанавливайте и запускайте **только внутри виртуального окружения проекта** (папка `.venv`). Так меньше «магии», больше предсказуемости.
* В примерах встречаются версии вроде `3.12` и `2.32.3`. У вас они могут отличаться — берите актуальные для своего проекта.
* Один проект — **один** менеджер зависимостей: `pip/pip-tools`, `uv`, `Poetry`, `PDM`, `Rye` или `Pipenv`. Смешивание инструментов повышает шанс конфликтов.

---

## 🧭 Содержание

* [Введение: зачем всё это](#введение-зачем-всё-это)
* [venv](#venv) — стандартный модуль для создания виртуальных окружений
* [pip](#pip) — базовый установщик пакетов
* [uv](#uv) — быстрый менеджер пакетов и lock
* [pip-tools](#pip-tools) — генерация фиксированных requirements и sync
* [virtualenv](#virtualenv) — гибкое и быстрое создание окружений
* [Poetry](#poetry) — управление в `pyproject.toml` и `poetry.lock`
* [PDM](#pdm) — современный менеджер на актуальных PEP
* [Rye](#rye) — версии Python + зависимости + окружение
* [Pipenv](#pipenv) — `Pipfile` и автогенерация окружения
* [Итоговое резюме и рекомендуемые стеки](#итоговое-резюме-и-рекомендуемые-стеки)
* [Рекомендации и полезные привычки](#рекомендации-и-полезные-привычки)

---

## Введение: зачем всё это

Представьте проект как небольшой аквариум. Вода — версии пакетов. Если каждый раз «наливать из‑под крана», рыбкам быстро надоест. Мы создаём **свою воду** (виртуальное окружение), **фиксируем состав** (lock/`requirements.txt`) и получаем **одинаковый результат** на любой машине.

---

## venv

**venv** — стандартный модуль Python для создания изолированных виртуальных окружений внутри проекта. Такое окружение хранит свои собственные пакеты и не трогает системный Python. Это помогает запускать код предсказуемо на любой машине.

---

### 🚀 Шаг 1: Создаём виртуальное окружение

```bash
python -m venv .venv
```

Запускается модуль `venv`, который создаёт новую папку `.venv` со своим интерпретатором Python и директорией для пакетов. Точка в имени делает папку скрытой в Unix‑системах и помогает держать порядок в корне проекта.

**Ожидаемый результат:** в проекте появится папка `.venv/` со служебными файлами и каталогами (`Scripts` или `bin`, `Lib` и т. п.).

---

### 🔑 Шаг 2: Активируем окружение

**Windows (PowerShell):**

```powershell
.\.venv\Scripts\Activate.ps1
```

**Windows (cmd.exe):**

```bat
.\.venv\Scripts\activate.bat
```

**macOS/Linux (bash/zsh):**

```bash
source .venv/bin/activate
```

Скрипт активации меняет переменные среды текущего процесса. В приглашении появится префикс окружения.

**Ожидаемый результат:**

```text
(.venv) yourname@host project-dir $
```

Чтобы выйти из окружения, используйте:

```bash
deactivate
```

---

### 🛠️ Шаг 3: Обновляем `pip` внутри окружения

```bash
python -m pip install --upgrade pip
```

Обновляет менеджер пакетов в текущем окружении до свежей версии, чтобы меньше ловить ошибки на установке.

---

### 📦 Шаг 4: Устанавливаем пакеты только внутрь активированного окружения

```bash
pip install requests
```

Устанавливает пакет **внутрь** активированного окружения `.venv`.

Проверка:

```bash
python -c "import requests, sys; print('OK', requests.__version__, sys.executable)"
```

Ожидаем «OK», версию и путь к интерпретатору из `.venv`.

---

## pip

**pip** — штатный установщик пакетов для Python. Работает из активированного интерпретатора и ставит зависимости **в текущее виртуальное окружение** проекта.

---

### 🚀 Шаг 1: Проверяем и обновляем `pip`

```bash
python -m pip --version
python -m pip install --upgrade pip
```

Первая команда покажет версию и путь до исполняемого файла (убедитесь, что это `.venv`). Вторая обновит `pip`.

**Ожидаемый результат (пример):**

```text
pip 24.x from /path/to/project/.venv/... (python 3.x)
Successfully installed pip-24.x
```

---

### 📦 Шаг 2: Устанавливаем пакеты

```bash
python -m pip install requests
python -m pip install "requests==2.32.3"
python -m pip install "urllib3>=2.2"
python -m pip install -r requirements.txt
```

Соответственно: базовая установка, установка точной версии, установка с нижней границей и установка всего из файла требований.

---

### 🔍 Шаг 3: Проверяем место установки

```bash
python -c "import requests, sys; print('OK', requests.__version__, sys.executable)"
```

Должен быть путь к `.venv`.

---

### 🧾 Шаг 4: Фиксируем версии

```bash
python -m pip freeze > requirements.txt
python -m pip install --require-hashes -r requirements.txt
```

Первое — снимок версий, второе — строгая установка по списку и хешам.

---

### 🧹 Шаг 5: Обновление, удаление и диагностика

```bash
python -m pip install -U requests
python -m pip uninstall requests
python -m pip list
python -m pip show requests
python -m pip check
python -m pip cache purge
```

Обычные операции: обновить, удалить, посмотреть список, детали, проверить конфликты и очистить кэш.

---

## uv

**uv** — сверхбыстрый менеджер пакетов и проектов. Умеет создавать и использовать виртуальные окружения, фиксировать версии в lock‑файле, ставить зависимости молниеносно и управлять версиями Python.

---

### 🚀 Шаг 1: Проверяем `uv`

```bash
uv --version
```

Показывает версию `uv`.

---

### 🔑 Шаг 2: Активируем существующее окружение или создаём новое

```bash
uv venv .venv
```

Создаёт окружение рядом с проектом. Активируйте его стандартно:

**Windows (PowerShell):**

```powershell
.\.venv\Scripts\Activate.ps1
```

**macOS/Linux:**

```bash
source .venv/bin/activate
```

---

### 📦 Шаг 3: Быстрая установка пакетов (совместимый режим `pip`)

```bash
uv pip install requests
uv pip install "requests==2.32.3"
uv pip install -r requirements.txt
```

Устанавливает пакет(ы) в активированное окружение.

Проверка:

```bash
python -c "import requests, sys; print('OK', requests.__version__, sys.executable)"
```

---

### 🧾 Шаг 4: Фиксации через requirements

```bash
uv pip compile requirements.in -o requirements.txt
uv pip sync requirements.txt
```

Разрешает версии и синхронизирует окружение строго под файл.

---

### 📄 Шаг 5: Работа через `pyproject.toml` и lock‑файл

```bash
uv init
uv add requests==2.32.3
uv sync --frozen
uv tree
```

Создание проекта, добавление зависимости, строгая синхронизация по lock и просмотр дерева зависимостей.

> Пример ограничения версии Python в `pyproject.toml`:

```toml
[project]
name = "my_project"
version = "0.1.0"
requires-python = ">=3.12,<3.13"
```

---

### 🐍 Шаг 6: Версии Python и запуск команд

```bash
uv python install 3.12
uv run python --version
uv run -m pytest
```

Установка нужного Python и запуск команд в контексте проекта.

---

## pip-tools

**pip-tools** — набор утилит для детерминируемых установок: `pip-compile` генерирует **зафиксированный** `requirements.txt`, а `pip-sync` приводит окружение ровно к этим версиям.

---

### 🚀 Шаг 1: Установка

```bash
python -m pip install pip-tools
```

Устанавливает `pip-compile` и `pip-sync` в активированное окружение.

---

### 📄 Шаг 2: Декларативный список зависимостей

```text
# requirements.in — верхнеуровневые зависимости
requests==2.32.3
uvicorn>=0.30
```

---

### 🧮 Шаг 3: Компиляция фиксированных версий

```bash
pip-compile requirements.in -o requirements.txt
pip-compile --upgrade -o requirements.txt requirements.in
pip-compile --upgrade-package requests -o requirements.txt requirements.in
pip-compile --generate-hashes -o requirements.txt requirements.in
```

Генерация точных версий, массовое обновление, точечное обновление и добавление хешей.

**Ожидаемый фрагмент:**

```text
# This file is autogenerated by pip-compile
requests==2.32.3 \
    --hash=sha256:<hash1> \
    --hash=sha256:<hash2>
```

---

### 🔁 Шаг 4: Синхронизация окружения

```bash
pip-sync requirements.txt
pip-sync --dry-run requirements.txt
```

Приводит окружение строго к содержимому файла, есть сухой прогон.

---

### 🧪 Шаг 5: Проверка

```bash
python -m pip check
```

Покажет конфликты зависимостей, если они есть.

---

## virtualenv

**virtualenv** — продвинутая утилита для создания виртуальных окружений. Гибче и обычно быстрее `venv`, особенно если нужна **нестандартная версия Python** или тонкая настройка.

---

### 🚦 Шаг 1: Проверяем доступность

```bash
virtualenv --version
```

---

### 🏗️ Шаг 2: Создаём окружение

```bash
virtualenv .venv
virtualenv .venv --upgrade-deps
virtualenv .venv --prompt "proj"
```

Быстрое создание, создание с обновлением инструментов и с кастомным префиксом приглашения.

---

### 🔢 Шаг 3: Конкретная версия Python

```bash
virtualenv -p /usr/bin/python3.12 .venv312
virtualenv -p python3.12 .venv312
py -3.11 -m virtualenv .venv311   # Windows
```

Создаёт окружения, жёстко привязанные к нужной версии интерпретатора.

---

### 🔑 Шаг 4: Активация

**Windows (PowerShell):**

```powershell
.\.venv\Scripts\Activate.ps1
```

**Windows (cmd.exe):**

```bat
.\.venv\Scripts\activate.bat
```

**macOS/Linux:**

```bash
source .venv/bin/activate
```

---

### 🧰 Шаг 5: Устанавливаем и проверяем пакеты

```bash
python -m pip install --upgrade pip
pip install requests
python -c "import requests, sys; print('OK', requests.__version__, sys.executable)"
```

---

### 🔁 Шаг 6: Переносимость

```bash
python -m pip freeze > requirements.txt
virtualenv .venv && source .venv/bin/activate
python -m pip install -r requirements.txt
```

---

### 🔧 Шаг 7: Полезные параметры

```bash
virtualenv .venv --copies
rm -rf .venv && virtualenv .venv --upgrade-deps
```

---

## Poetry

**Poetry** — менеджер зависимостей и виртуальных окружений. Хранит список пакетов в `pyproject.toml`, фиксирует версии в `poetry.lock`, автоматизирует установку, обновления и запуск команд.

> Рекомендуем хранить `.venv` **внутри проекта**.

---

### 🚀 Шаг 1: Установка и проверка

```bash
python -m pip install --upgrade pip
python -m pip install poetry
poetry --version
```

---

### 🧭 Шаг 2: Храним окружение в проекте

```bash
poetry config virtualenvs.create true
poetry config virtualenvs.in-project true
poetry config virtualenvs.prefer-active-python true
```

---

### 🏗️ Шаг 3: Инициализация проекта

```bash
poetry init
# или без вопросов:
poetry init -n
```

Пример ограничения Python:

```toml
[tool.poetry]
name = "my_project"
version = "0.1.0"

[tool.poetry.dependencies]
python = ">=3.12,<3.13"
```

---

### 🐍 Шаг 4: Выбор интерпретатора

```bash
poetry env use 3.12
poetry env info
```

---

### 📦 Шаг 5: Добавляем зависимости

```bash
poetry add requests==2.32.3
poetry add --group dev pytest
poetry add "urllib3>=2.2"
```

---

### 🔒 Шаг 6: Lock и синхронизация

```bash
poetry lock
poetry install --sync
poetry update
```

---

### 🧪 Шаг 7: Запуск команд

```bash
poetry run python --version
poetry run -m pytest -q
poetry shell
```

---

### 🧾 Шаг 8: Экспорт в `requirements.txt`

```bash
poetry export -f requirements.txt -o requirements.txt --with-hashes
poetry export -f requirements.txt -o requirements.txt --without-hashes --without dev
```

---

### 🔁 Шаг 9: Воспроизведение на другой машине

```bash
git clone <repo-url> my_project
cd my_project
python -m pip install poetry
poetry install --sync --no-root
poetry run python -c "import requests, sys; print('OK', requests.__version__, sys.executable)"
```

---

### 🧹 Шаг 10: Обслуживание

```bash
poetry remove requests
poetry update requests
```

---

## PDM

**PDM** — современный менеджер зависимостей, следующий актуальным PEP. Хранит список пакетов в `pyproject.toml`, фиксирует версии в `pdm.lock`, автоматически создаёт и использует изолированное окружение для проекта.

---

### 🚀 Шаг 1: Установка и проверка

```bash
python -m pip install --upgrade pip
python -m pip install pdm
pdm --version
```

---

### 🏗️ Шаг 2: Инициализация

```bash
pdm init
# или
pdm init -n
```

Пример `pyproject.toml`:

```toml
[project]
name = "my_project"
version = "0.1.0"
requires-python = ">=3.12,<3.13"
```

---

### 🐍 Шаг 3: Интерпретатор

```bash
pdm use -f 3.12
pdm use -f /usr/bin/python3.12
pdm info
```

---

### 📦 Шаг 4: Добавление зависимостей

```bash
pdm add requests==2.32.3
pdm add --group dev pytest
pdm add "urllib3>=2.2"
```

---

### 🔒 Шаг 5: Lock и установка

```bash
pdm lock
pdm install
pdm update
```

---

### 🧪 Шаг 6: Запуск команд

```bash
pdm run python --version
pdm run -m pytest -q
pdm run python -c "import requests, sys; print('OK', requests.__version__, sys.executable)"
```

---

### 🧾 Шаг 7: Экспорт в `requirements.txt`

```bash
pdm export -f requirements -o requirements.txt
pdm export -f requirements -o requirements.txt --prod
```

---

### 🔁 Шаг 8: Воспроизведение

```bash
git clone <repo-url> my_project
cd my_project
python -m pip install pdm
pdm install
```

---

### 🧹 Шаг 9: Обслуживание

```bash
pdm remove requests
pdm update requests
pdm list
```

---

## Rye

**Rye** — единый инструмент для Python: управляет версиями интерпретатора, зависимостями и виртуальными окружениями. Хранит зависимости в `pyproject.toml`, фиксирует их в lock‑файле и синхронизирует окружение.

---

### 🚀 Шаг 1: Установка и проверка

```bash
python -m pip install --upgrade pip
python -m pip install rye
rye --version
```

---

### 🏗️ Шаг 2: Инициализация проекта

```bash
rye init
# или
rye init --no-interaction
```

---

### 🐍 Шаг 3: Версия Python

```bash
rye pin 3.12
rye toolchain list
rye toolchain install 3.12
```

Закрепляем версию Python, смотрим доступные и при необходимости устанавливаем нужную.

---

### 🔑 Шаг 4: Создание и синхронизация окружения

```bash
rye sync
rye sync --frozen
```

Создаёт `.venv/` и устанавливает зависимости по lock‑файлу. В строгом режиме не позволяет расходиться с lock.

---

### 📦 Шаг 5: Зависимости

```bash
rye add requests==2.32.3
rye add --dev pytest
rye lock
```

Добавление пакетов, dev‑зависимостей и пересборка lock.

---

### 🧪 Шаг 6: Запуск команд

```bash
rye run python --version
rye run -m pytest -q
rye run python -c "import requests, sys; print('OK', requests.__version__, sys.executable)"
```

---

### 🔁 Шаг 7: Воспроизведение

```bash
git clone <repo-url> my_project
cd my_project
python -m pip install rye
rye sync --frozen
```

---

### 🧹 Шаг 8: Обслуживание

```bash
rye remove requests
rye lock --update
rye sync
```

---

## Pipenv

**Pipenv** — менеджер зависимостей с автоматическим созданием виртуального окружения и lock‑файлом `Pipfile.lock`. Список пакетов хранится в `Pipfile`.

> Хранить окружение удобнее прямо в проекте — включим режим `.venv` в корне.

---

### 🚀 Шаг 1: Установка и проверка

```bash
python -m pip install --upgrade pip
python -m pip install pipenv
pipenv --version
```

---

### 📍 Шаг 2: Храним окружение в проекте

**macOS/Linux:**

```bash
export PIPENV_VENV_IN_PROJECT=1
```

**Windows (PowerShell):**

```powershell
$env:PIPENV_VENV_IN_PROJECT = 1
```

**Windows (cmd.exe):**

```bat
set PIPENV_VENV_IN_PROJECT=1
```

---

### 🏗️ Шаг 3: Инициализация и выбор Python

```bash
pipenv --python 3.12
```

Создаёт `Pipfile` и виртуальное окружение `.venv`.

---

### 📦 Шаг 4: Добавление зависимостей

```bash
pipenv add requests==2.32.3
pipenv add --dev pytest
pipenv add "urllib3>=2.2"
```

---

### 🔒 Шаг 5: Lock и синхронизация

```bash
pipenv lock
pipenv sync
pipenv sync --categories main
```

Пересчёт lock‑файла, синхронизация и продакшен‑вариант без dev.

---

### 🧪 Шаг 6: Запуск команд

```bash
pipenv run python --version
pipenv run -m pytest -q
pipenv shell
pipenv run python -c "import requests, sys; print('OK', requests.__version__, sys.executable)"
```

---

### 🧾 Шаг 7: Экспорт в `requirements.txt`

```bash
pipenv lock -r > requirements.txt
pipenv lock -r --dev > requirements-dev.txt
```

---

### 🔁 Шаг 8: Воспроизведение

```bash
git clone <repo-url> my_project
cd my_project
python -m pip install pipenv
export PIPENV_VENV_IN_PROJECT=1   # или эквивалент для Windows
pipenv sync --dev --deploy
```

---

### 🧹 Шаг 9: Обслуживание

```bash
pipenv uninstall requests
pipenv update
pipenv graph
pipenv check
```

---

## Итоговое резюме и рекомендуемые стеки

* **Базовый надёжный стек (по умолчанию):** `venv` + `pip` + `pip-tools`
  Простой, прозрачный, переносимый. `pip-compile`/`pip-sync` обеспечивают бетонные фиксации.

* **Современный быстрый стек:** `uv`
  Скорость, lock и sync из коробки, единый инструмент для проекта.

* **Проектный менеджмент:** `Poetry` **или** `PDM`
  Зависимости и метаданные в `pyproject.toml`, lock‑файлы и запуск команд.

* **Всё‑в‑одном:** `Rye`
  Фиксирует Python, зависимости и окружение, удобно для командной разработки.

* **Совместимость/наследие:** `Pipenv`
  Используйте там, где он исторически принят или требуется инфраструктурой.

* **Тонкая настройка окружений:** `virtualenv`
  Нужен специфический интерпретатор или параметры создания — берите его.

**Золотое правило:** один проект — один менеджер. Меньше хаоса, больше счастья.

---

## Рекомендации и полезные привычки

* Храните окружение **внутри проекта** (`.venv`) и добавьте его в `.gitignore`.
* Всегда фиксируйте версии: `pip-compile`/`uv lock`/`poetry lock`/`pdm lock`/`Pipfile.lock`/`rye lock`.
* Для CI включайте строгие режимы: `--require-hashes`, `--frozen`, `--sync`, `--deploy`.
* Документируйте быстрый старт в `README`: 3–5 команд «клонировал и побежал».
* Разделяйте боевые и dev‑зависимости (группы/категории).

---

*Теперь у вас есть инструкция, которая не ругается, когда видит другой ноутбук. Создавайте окружение, фиксируйте версии и живите спокойно.*
