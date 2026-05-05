# Cockpit Tools — fork с горячим перезапуском Windsurf

> Языки README: **Русский** · [中文](README.zh-CN.md) · [English](README.en.md)

Этот репозиторий — форк [`jlcodes99/cockpit-tools`](https://github.com/jlcodes99/cockpit-tools) с одним важным отличием:

**При смене аккаунта Windsurf IDE перезапускается в той же рабочей директории**, в которой была открыта до переключения. Не нужно вручную возвращаться к проекту — Windsurf сам поднимется на нужной папке с уже подменённым аккаунтом.

---

## Что нового в форке

`@/src-tauri/src/modules/windsurf_instance.rs` — добавлена функция `capture_workspace_arg_from_pid`, которая:
- читает командную строку запущенного процесса Windsurf через `/proc/<pid>/cmdline` (Linux), `ps -ww -o args=` (macOS) или `wmic process` (Windows);
- извлекает первый позиционный аргумент (путь к открытой папке/воркспейсу);
- проверяет существование пути в файловой системе.

`@/src-tauri/src/commands/windsurf_instance.rs` — команда `windsurf_start_instance` теперь:
1. Захватывает workspace-путь **до** закрытия Windsurf.
2. Закрывает IDE, инжектит новый аккаунт в `state.vscdb`.
3. Запускает Windsurf, добавляя в аргументы захваченный путь.

В логах при срабатывании появляется строка:
```
[Windsurf Restart] восстановление рабочей директории: /path/to/your/project
```

Дефолтный язык интерфейса в форке — русский.

---

## Системные требования

- Linux x86_64 (Debian/Ubuntu/Arch) — основная целевая платформа форка
- Windsurf IDE установлен в `/usr/bin/windsurf` (стандартный путь Linux-сборки)
- Графическая сессия (X11 или Wayland)
- Доступ в интернет для скачивания зависимостей сборки

Для сборки потребуются:
- **Rust 1.80+** (через `rustup`)
- **Node.js 20+** и **npm**
- Системные библиотеки Tauri (gtk, webkit и т. д.)

---

## Быстрый старт (Debian / Ubuntu)

```bash
# 1. Системные зависимости Tauri
sudo apt update
sudo apt install -y \
  build-essential curl wget file libssl-dev libgtk-3-dev libayatana-appindicator3-dev \
  librsvg2-dev libwebkit2gtk-4.1-dev libsoup-3.0-dev pkg-config \
  libxdo-dev xdotool wtype ydotool

# 2. Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"

# 3. Node.js (если ещё нет)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# 4. Клонирование и сборка
git clone https://github.com/kroch228/cockpit-tools.git
cd cockpit-tools
npm install
npm run tauri build -- --bundles deb

# 5. Установка собранного .deb
sudo dpkg -i "target/release/bundle/deb/Cockpit Tools_"*"_amd64.deb"

# 6. Запуск
cockpit-tools
```

После запуска `cockpit-tools` должен:
- появиться в трее;
- открыть главное окно Cockpit Tools;
- начать в фоне рефрешить токены добавленных аккаунтов.

## Быстрый старт (Arch / Manjaro)

```bash
# 1. Системные зависимости
sudo pacman -S --needed base-devel curl wget openssl gtk3 libayatana-appindicator \
  librsvg webkit2gtk-4.1 libsoup3 pkgconf nodejs npm xdotool wtype ydotool

# 2. Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"

# 3. Сборка
git clone https://github.com/kroch228/cockpit-tools.git
cd cockpit-tools
npm install
npm run tauri build -- --bundles deb

# 4. Запуск напрямую из target (без dpkg)
./target/release/cockpit-tools
```

---

## Установка без `dpkg`

Если не нужен .deb пакет — можно скопировать бинарник вручную:

```bash
mkdir -p ~/.local/bin
install -m 755 target/release/cockpit-tools ~/.local/bin/cockpit-tools

# Убедись, что ~/.local/bin есть в PATH:
echo $PATH | tr ':' '\n' | grep -q "$HOME/.local/bin" || \
  echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc

cockpit-tools
```

---

## Где Cockpit Tools хранит свои данные

Все аккаунты, токены и кэш — **вне репозитория**, в:

```
~/.antigravity_cockpit/
```

Эта директория **никогда не попадает в git**. При первом запуске она создаётся пустой — ты добавляешь свои аккаунты сам через UI.

---

## Проверка горячего перезапуска Windsurf

1. Открой Windsurf IDE в любой папке проекта (`windsurf ~/my-project`).
2. В Cockpit Tools → раздел **Windsurf** → **Управление инстансами** → выбери в дефолтном инстансе другой привязанный аккаунт.
3. Нажми **Запустить**.
4. Windsurf закроется и сразу же откроется заново уже под новым аккаунтом, **в той же папке `~/my-project`**.

В логах `~/.antigravity_cockpit/logs/app.log.*` появится:
```
[Windsurf Restart] восстановление рабочей директории: /home/<user>/my-project
```

Если строки нет — Cockpit Tools не смог захватить путь. Возможные причины:
- Windsurf не был запущен в момент клика «Запустить» (нечего захватывать).
- Открыт пустой стартовый экран без папки/файла.
- Бинарник Windsurf лежит не в `/usr/bin/windsurf` (поправь в настройках Cockpit Tools).

---

## Обновление

```bash
cd cockpit-tools
git pull
npm install
npm run tauri build -- --bundles deb
sudo dpkg -i "target/release/bundle/deb/Cockpit Tools_"*"_amd64.deb"
```

---

## Лицензия и оригинал

Это форк проекта [`jlcodes99/cockpit-tools`](https://github.com/jlcodes99/cockpit-tools). Лицензия и условия использования — те же, что в оригинальном репозитории. См. также подробное описание возможностей в [`README.zh-CN.md`](README.zh-CN.md) и [`README.en.md`](README.en.md).

## Issues

Если что-то не собирается / не работает — открой issue: <https://github.com/kroch228/cockpit-tools/issues>
