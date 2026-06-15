---
title: Настройка локального окружения разработки на Windows с WSL
description: Подготовка Ubuntu, Docker Desktop, Node.js, Yarn и Codex CLI с WSL 2 на Windows для локальной разработки NocoBase и рабочих процессов с ИИ агентами.
---

# Настройка локального окружения разработки на Windows с WSL

Для локальной разработки NocoBase на Windows рекомендуем сначала подготовить WSL 2. Это позволяет Node.js, Yarn, NocoBase CLI, командам Docker и ИИ агентам работать в одной оболочке Linux — с путями, правами и сборкой нативных зависимостей, ближе к типичным Linux-окружениям развёртывания.

Если вы не уверены, нужен ли вам WSL, сначала см. [Настройка локального окружения разработки](./local-development-setup.md).

## Перед началом

Перед установкой WSL проверьте версию Windows и состояние виртуализации.

### Проверка версии Windows

Нажмите `Win + R`, введите `winver` и убедитесь, что ваша система соответствует одному из следующих требований:

- Windows 11
- Windows 10 версии 2004 или новее, сборка 19041 или новее

Если версия старше, обновите Windows перед продолжением.

### Проверка виртуализации

Откройте диспетчер задач, перейдите в раздел «Производительность» / «ЦП» и убедитесь, что виртуализация отображается как «Включено».

Если виртуализация не включена, включите её в BIOS / UEFI. Название опции зависит от производителя, например Intel VT-x, Intel Virtualization Technology, AMD-V или SVM Mode.

## Шаг 1: Установка WSL 2

Откройте PowerShell от имени администратора:

1. Откройте меню «Пуск» Windows
2. Найдите `PowerShell`
3. Щёлкните правой кнопкой мыши и выберите «Запуск от имени администратора»

Выполните:

```powershell
wsl --install
```

После установки перезагрузите компьютер.

По умолчанию эта команда устанавливает Ubuntu. При первом запуске Ubuntu попросит создать имя пользователя и пароль Linux. Эти имя и пароль используются только внутри WSL и не обязаны совпадать с учётной записью Windows.

Чтобы установить конкретный дистрибутив, сначала выведите список доступных дистрибутивов:

```powershell
wsl --list --online
```

Затем установите дистрибутив, например Ubuntu:

```powershell
wsl --install -d Ubuntu
```

## Шаг 2: Подтверждение версии WSL

Выполните следующую команду в PowerShell:

```powershell
wsl --list --verbose
```

Вывод должен выглядеть примерно так:

```text
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

Убедитесь, что `VERSION` равен `2`. Если дистрибутив всё ещё использует WSL 1, преобразуйте его в WSL 2:

```powershell
wsl --set-version Ubuntu 2
```

Также рекомендуем установить WSL 2 версией по умолчанию для вновь устанавливаемых дистрибутивов:

```powershell
wsl --set-default-version 2
```

При необходимости можно один раз обновить WSL:

```powershell
wsl --update
```

## Шаг 3: Установка Docker Desktop

Если вы планируете устанавливать или запускать NocoBase через Docker, установите Docker Desktop для Windows.

Скачайте установщик из документации Docker:

- [Установка Docker Desktop в Windows](https://docs.docker.com/desktop/setup/install/windows-install/)

Во время установки обратите внимание на следующие параметры:

1. Обычно для локальной разработки на личном компьютере выбирайте `Per-user`
2. На странице конфигурации выберите `Use WSL 2 instead of Hyper-V`
3. После установки запустите Docker Desktop из меню «Пуск» Windows
4. При первом запуске прочитайте и примите Docker Desktop Subscription Service Agreement

## Шаг 4: Включение интеграции Docker с WSL

После запуска Docker Desktop сначала убедитесь, что включён бэкенд WSL 2:

1. Перейдите в Docker Desktop / Settings / General
2. Убедитесь, что включён параметр Use the WSL 2 based engine
3. Нажмите Apply

![Docker Desktop WSL 2 engine](https://static-docs.nocobase.com/2026-06-12-19-10-41.png)

Затем включите интеграцию с дистрибутивом:

1. Перейдите в Docker Desktop / Settings / Resources / WSL Integration
2. Включите Enable integration with my default WSL distro
3. Найдите нужный дистрибутив, например `Ubuntu`
4. Включите переключатель для этого дистрибутива
5. Нажмите Apply & restart или Apply

![Docker Desktop WSL integration](https://static-docs.nocobase.com/2026-06-12-19-11-09.png)

Если WSL Integration не отображается в разделе Resources, Docker Desktop обычно работает в режиме Windows containers. Щёлкните значок Docker в системном трее Windows, переключитесь на Linux containers и проверьте снова.

## Шаг 5: Проверка Docker

Сначала проверьте из PowerShell:

```powershell
wsl --list --verbose
docker version
docker compose version
docker run hello-world
```

Затем войдите в WSL:

```powershell
wsl
```

Выполните следующие команды внутри WSL:

```bash
docker version
docker compose version
docker run hello-world
```

Если контейнер `hello-world` успешно загружен и запущен, Docker Desktop и интеграция с WSL 2 работают.

## Шаг 6: Установка Node.js и Yarn в WSL

Сам WSL не является средой выполнения Node.js. Ubuntu, установленная через `wsl --install`, обычно не включает Node.js или npm, поэтому установите их внутри дистрибутива WSL.

Войдите в WSL из PowerShell:

```powershell
wsl
```

Все команды ниже выполняются в терминале WSL.

Сначала проверьте, установлен ли уже Node.js:

```bash
node -v
npm -v
```

Если появляется `command not found`, установите Node.js одним из следующих способов.

### Вариант A: Установка Node.js 22 через NodeSource

Если в этом окружении WSL нужна одна общая версия Node.js, рекомендуется NodeSource.

```bash
sudo apt update
sudo apt install -y curl
curl -fsSL https://deb.nodesource.com/setup_22.x -o nodesource_setup.sh
sudo -E bash nodesource_setup.sh
sudo apt install -y nodejs
```

Проверьте установку:

```bash
node -v
npm -v
npx -v
```

### Вариант B: Установка Node.js 22 через nvm

Если нужно переключать версии Node.js между проектами или проект использует `.nvmrc` для фиксации версии, используйте nvm.

```bash
sudo apt update
sudo apt install -y curl ca-certificates
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.5/install.sh | bash
source ~/.bashrc
nvm install 22
nvm use 22
```

Проверьте установку:

```bash
node -v
npm -v
npx -v
```

Если в корне проекта нужно зафиксировать Node.js 22, создайте `.nvmrc`:

```bash
echo "22" > .nvmrc
nvm install
nvm use
```

Позже после входа в каталог проекта выполните:

```bash
nvm use
```

Это переключит на версию, указанную проектом.

:::warning Примечание

Выберите NodeSource или nvm. Не рекомендуем смешивать оба способа управления Node.js в одном пользовательском окружении WSL.

:::

### Установка Yarn 1.x

Для локальной разработки NocoBase требуется Yarn 1.x. После установки Node.js можно включить Yarn через Corepack:

```bash
corepack enable
corepack prepare yarn@1.22.22 --activate
yarn -v
```

Если Corepack недоступен в вашем окружении, установите Yarn через npm:

```bash
npm install -g yarn@1.22.22
yarn -v
```

## Шаг 7: Установка Codex CLI

Codex CLI также можно использовать в нативной командной строке Windows. В этом руководстве он устанавливается внутри WSL, чтобы Codex и локальный инструментарий NocoBase находились в одном Linux-окружении. Когда Codex выполняет команды вроде `nb`, `yarn` или `docker`, он использует те же пути к файлам, синтаксис оболочки и среду выполнения.

Убедитесь, что вы сейчас находитесь внутри WSL:

```bash
echo $WSL_DISTRO_NAME
```

Если выводится имя дистрибутива, например `Ubuntu`, вы находитесь внутри WSL.

Выполните установщик Codex CLI внутри WSL:

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

Для неинтерактивной установки используйте:

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | CODEX_NON_INTERACTIVE=1 sh
```

После установки выполните:

```bash
codex
```

При первом запуске появится запрос на вход. Следуйте подсказкам и выполните аутентификацию с учётной записью ChatGPT или ключом OpenAI API.

Проверьте доступность команды Codex:

```bash
codex --version
```

Рекомендуем запускать Codex из каталога проекта внутри WSL:

```bash
mkdir -p ~/projects
cd ~/projects/my-app
codex
```

:::tip Примечание

Поскольку Codex установлен внутри WSL, затем запускайте `codex` из терминала WSL. Если выполнить `codex` из Windows PowerShell, будет использоваться нативное окружение командной строки Windows, а не окружение, подготовленное в этом руководстве.

:::

## Где размещать файлы проекта

Рекомендуем размещать файлы проекта внутри файловой системы WSL, например:

```bash
~/projects/my-app
```

Избегайте использования смонтированного пути Windows как расположения проекта по умолчанию, например:

```bash
/mnt/c/Users/<your-name>/projects/my-app
```

Обычно это даёт лучшую производительность файловой системы и уменьшает проблемы с символическими ссылками и правами. Разница особенно заметна для проектов с большим количеством файлов зависимостей, таких как проекты Node.js, Python, Java или Go.

Чтобы открыть файлы WSL из проводника Windows, перейдите по адресу:

```text
\\wsl$\Ubuntu\home\<your-name>
```

## Часто задаваемые вопросы

### WSL сообщает, что команда docker не найдена

Сначала убедитесь, что дистрибутив использует WSL 2:

```powershell
wsl --list --verbose
```

Если используется WSL 1, преобразуйте его в WSL 2:

```powershell
wsl --set-version Ubuntu 2
```

Затем вернитесь в Docker Desktop, перейдите в Settings / Resources / WSL Integration, включите интеграцию для соответствующего дистрибутива и примените изменения.

### Отсутствует WSL Integration

Обычно Docker Desktop в данный момент работает в режиме Windows containers.

Можно поступить так:

1. Щёлкните значок Docker в системном трее Windows
2. Выберите Switch to Linux containers
3. Дождитесь перезапуска Docker Desktop
4. Снова перейдите в Settings / Resources / WSL Integration

### Docker Desktop не запускается или WSL ведёт себя необычно

Сначала попробуйте:

```powershell
wsl --shutdown
wsl --update
```

Затем перезапустите Docker Desktop.

### Docker Engine был установлен в WSL вручную

Docker рекомендует удалить Docker Engine или Docker CLI, установленные напрямую в дистрибутив Linux WSL, перед установкой Docker Desktop. Иначе они могут конфликтовать с интеграцией Docker Desktop и WSL.

Обычно внутри дистрибутива WSL удаляют `docker-ce`, `docker-ce-cli`, `containerd.io` и связанные пакеты, а затем используют интеграцию Docker CLI, предоставляемую Docker Desktop.

### WSL сообщает, что команда codex не найдена

Сначала убедитесь, что команда выполняется внутри WSL, а не в PowerShell:

```bash
echo $WSL_DISTRO_NAME
```

Затем проверьте, есть ли Codex в `PATH`:

```bash
which codex
```

Если команда не найдена, откройте терминал WSL заново или выполните установщик снова:

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

## Официальные ссылки

- [Microsoft Learn: установка Linux в Windows с помощью WSL](https://learn.microsoft.com/en-us/windows/wsl/install)
- [Microsoft Learn: установка Node.js в подсистеме Windows для Linux](https://learn.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-wsl)
- [Docker Docs: установка Docker Desktop в Windows](https://docs.docker.com/desktop/setup/install/windows-install/)
- [Docker Docs: бэкенд Docker Desktop WSL 2 в Windows](https://docs.docker.com/desktop/features/wsl/)
- [Docker Docs: изменение настроек Docker Desktop](https://docs.docker.com/desktop/settings-and-maintenance/settings/)
- [OpenAI Developers: Codex CLI](https://developers.openai.com/codex/cli)
- [OpenAI Developers: Codex в Windows](https://developers.openai.com/codex/windows)
- [nvm: Node Version Manager](https://github.com/nvm-sh/nvm)
- [npm Docs: загрузка и установка Node.js и npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm/)
