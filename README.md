# nx-ovlloader (Ryazha edition)

**EN:** Sysmodule that chainloads Tesla overlay menus on Nintendo Switch. Fork of [ppkantorski/nx-ovlloader](https://github.com/ppkantorski/nx-ovlloader) adapted for the Ryazha ecosystem (Ryazhahand-Overlay, RCU, libryazhahand). Functionality identical to upstream — title ID and manifest preserved. License: ISC. HOS 16.0.0+.

---

## Что это

Core-загрузчик sysmodule для Tesla-совместимых overlay-меню. Стартует с консолью, маппит overlay NRO в память и chainload'ит `/switch/.overlays/ovlmenu.ovl`. Без него Tesla-overlay не запустится.

Этот репозиторий — форк ppkantorski/nx-ovlloader. Функционал идентичен upstream'у — title ID `420000000007E51A`, sysmodule manifest и DMA-layout сохранены, любая Tesla-совместимая `ovlmenu.ovl` грузится корректно.

## Отличия от upstream

- Брендирование (README/CI под Ryazha-проекты).
- Авто-sync workflow `.github/workflows/sync_upstream.yml` — ежедневный cron подтягивает изменения от ppkantorski и открывает PR. Ручной merge не нужен.
- Используется как submodule в `Ryazhahand-Overlay/vendor/nx-ovlloader/`. Релизная сборка тянет наш бинарь, а не online-fetch из upstream releases.

## Установка

Если используешь Ryazhahand-Overlay (релиз `sdout.zip`) — nx-ovlloader уже включён в bundle, ничего отдельно ставить не нужно.

Если ставишь руками:

1. Скачать релиз → распаковать в корень SD.
2. Структура:
   - `atmosphere/contents/420000000007E51A/exefs.nsp` — sysmodule.
   - `atmosphere/contents/420000000007E51A/flags/boot2.flag` — авто-старт при boot.
   - `atmosphere/contents/420000000007E51A/toolbox.json` — manifest для Tesla-launcher'ов.
3. Перезагрузить консоль.

## Сборка из исходников

```sh
git clone https://github.com/Dimanchikgshehsbshene/nx-ovlloader.git
cd nx-ovlloader
export DEVKITPRO=/opt/devkitpro
make
```

Результат — `out/nx-ovlloader.nsp`. Бинарь идентичен upstream'у при том же source-tree.

## Лицензия

ISC. См. `LICENSE.md`.

Авторы: ppkantorski (upstream), Atmosphere-NX (loader-base), switchbrew (libnx), Dimasick-git (Ryazha-rebrand + sync workflow).
