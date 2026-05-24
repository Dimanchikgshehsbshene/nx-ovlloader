# nx-ovlloader (Ryazha edition) — HOS 16.0.0+

[![platform](https://img.shields.io/badge/platform-Switch-898c8c?logo=C.svg)](https://gbatemp.net/forums/nintendo-switch.283/?prefix_id=44)
[![language](https://img.shields.io/badge/language-C-ba1632?logo=C.svg)](https://github.com/topics/c)
[![ISC License](https://img.shields.io/badge/license-ISC-189c11.svg)](https://github.com/Dimanchikgshehsbshene/nx-ovlloader/blob/main/LICENSE.md)
[![Upstream sync](https://img.shields.io/badge/upstream-ppkantorski%2Fnx--ovlloader-blue)](https://github.com/ppkantorski/nx-ovlloader)

**nx-ovlloader** — core-загрузчик sysmodule для Tesla-совместимых
overlay-меню на Nintendo Switch. Запускается в фоне при старте консоли,
маппит overlay NRO в память и chainload'ит `/switch/.overlays/ovlmenu.ovl`
— даёт мгновенный доступ к overlay-меню из любой игры или приложения.

Этот репозиторий — **форк** [ppkantorski/nx-ovlloader](https://github.com/ppkantorski/nx-ovlloader),
адаптированный под экосистему **Ryazha**:
[Ryazhahand-Overlay](https://github.com/Dimanchikgshehsbshene/Ryazhahand-Overlay),
[RCU / ryazha-clk](https://github.com/Dimanchikgshehsbshene/RCU),
[libryazhahand](https://github.com/Dimanchikgshehsbshene/libryazhahand).

Функционал идентичен upstream'у — title-id, sysmodule manifest и DMA-layout
сохранены, чтобы любая Tesla-совместимая `ovlmenu.ovl` загружалась корректно.
Отличия:

1. **Брендирование** — наш репо/README/CI под Ryazha.
2. **Авто-sync с upstream** — `.github/workflows/sync_upstream.yml`
   ежедневно подтягивает изменения от ppkantorski и открывает PR
   с новыми коммитами. Ручной merge не нужен.
3. **Submodule** — подключается в `Ryazhahand-Overlay/vendor/nx-ovlloader`.
   Release-сборка использует нашу версию вместо online-fetch из upstream
   releases.

Производный от [nx-hbloader](https://github.com/switchbrew/nx-hbloader),
расширяет оригинал dynamic heap sizing, HOS-version-aware defaults,
exit flag сигнализацией и автоматическим self-relaunch через
[nx-ovlreloader](https://github.com/ppkantorski/nx-ovlreloader).

---

## Возможности

- **Chainloading оверлеев** — автоматически загружает `/switch/.overlays/ovlmenu.ovl`
  при старте и после каждого выхода из overlay'я.
- **Динамический heap** — читает persistent конфиг `/config/nx-ovlloader/heap_size.bin`;
  если не задан — выбирает оптимальный default по версии HOS:
  - HOS 21.0.0+ → **4 MB**
  - HOS 20.0.0+ → **6 MB**
  - Старше → **8 MB**
- **Live heap change detection** — проверяет смену heap между загрузками
  оверлеев, позволяя UI Ryzhand применять новый размер без перезагрузки.
- **Exit flag** — читает `/config/nx-ovlloader/exit_flag.bin` для clean exit'а
  (вместо авто-reload). Флаг удаляется сразу после прочтения.
- **Auto-relaunch через nx-ovlreloader** — при exit'е процесса использует
  `pmshell` чтобы запустить
  [nx-ovlreloader](https://github.com/ppkantorski/nx-ovlreloader)
  (`0x420000000007E51B`), который респавнит nx-ovlloader без полного
  restart консоли.
- **Rotating NRO address strategy** — маппит overlay-бинарь в предсказуемом
  64 GB–256 GB окне с 2 MB выравниванием.
- **Atomic load guard** — атомарный flag предотвращает конкурентные
  `loadNro()` вызовы.
- **Cached SD filesystem handle** — SD FS открывается один раз, не
  пере-открывается при каждом overlay switch.
- **Минимальный footprint** — одна FS сессия, минимальный directory cache,
  no CWD support.

---

## Установка

### Рекомендуемый способ (через полный пакет Ryzhand)

Проще всего — через релиз
[Ryazhahand-Overlay](https://github.com/Dimanchikgshehsbshene/Ryazhahand-Overlay)
`sdout.zip`. Распакуй в корень SD — он содержит nx-ovlloader,
nx-ovlreloader, `ovlmenu.ovl` и нужную структуру.

### Ручная установка

1. Скачай последний релиз
   [Releases](https://github.com/Dimanchikgshehsbshene/nx-ovlloader/releases/latest)
   и распакуй в корень SD-карты.
2. Положи overlay-menu в `/switch/.overlays/ovlmenu.ovl` — например,
   [Ryazhahand-Overlay](https://github.com/Dimanchikgshehsbshene/Ryazhahand-Overlay)
   или Tesla Menu.
3. Перезагрузи консоль. nx-ovlloader запустится автоматически с
   Atmosphère и chainload'ит `ovlmenu.ovl`.

> **Требования:** [Atmosphère](https://github.com/Atmosphere-NX/Atmosphere)
> и HOS 9.0.0+.

---

## Сборка из исходников

```sh
git clone --recursive https://github.com/Dimanchikgshehsbshene/nx-ovlloader.git
cd nx-ovlloader
make
```

Артефакт: `out/atmosphere/contents/420000000007E51A/exefs.nsp` (+ flag/toolbox.json).
nx-ovlreloader собирается рекурсивно как зависимость в `external/`.

Требования сборки: devkitPro + libnx + Atmosphère sysmodule headers.

---

## Auto-sync с upstream

Каждые 24 часа GitHub Actions (`.github/workflows/sync_upstream.yml`)
делает `git fetch upstream` и сравнивает с нашим `main`. Если есть
новые коммиты — создаётся PR с branch `sync/upstream-<date>`.
Maintainer review + merge.

Это сохраняет нашу историю чистой и upstream-фиксы прилетают
автоматически без необходимости вручную следить за `ppkantorski/nx-ovlloader`.

Принудительный sync:

```sh
gh workflow run sync_upstream.yml
```

---

## Authorship & license

ISC. Авторство upstream'а сохранено в `LICENSE.md`. Все правки в этой
ветке — © Dimasick-git, тоже под ISC.

Спасибо [ppkantorski](https://github.com/ppkantorski) за nx-ovlloader,
[Atmosphere-NX](https://github.com/Atmosphere-NX) за платформу,
[switchbrew](https://github.com/switchbrew) за исходный nx-hbloader.
