# Канал обновлений Quatrojet

Ветка `Releases` содержит только то, что скачивает само приложение:

| Файл | Назначение |
|---|---|
| `update.json` | описание последней версии |
| `qjpy-<версия>-arm64-v8a-release.apk` | подписанный release-APK этой версии |

Ветка не связана с историей кода (`dev2`/`main`): APK — большой бинарный файл,
и держать его в основной ветке незачем.

## Формат `update.json`

```json
{
  "version": "0.4.8",
  "apk": "qjpy-0.4.8-arm64-v8a-release.apk",
  "sha256": "<sha256 APK, 64 hex-символа>",
  "notes": "Что нового"
}
```

* `version` — версия в виде `X.Y.Z` (префикс `v` допускается). Приложение
  сравнивает её со своей (`_version.py`) и предлагает обновление только если
  опубликованная версия новее.
* `apk` — имя файла рядом с `update.json` **или** абсолютный `http(s)`-адрес
  (например, ассет GitHub Release).
* `sha256` — необязательная контрольная сумма: по ней можно проверить
  скачанный файл перед установкой.
* `notes` — необязательный текст для пользователя.

## Адреса, которые читает приложение

```
https://raw.githubusercontent.com/VanMo/quatrojet/Releases/update.json
https://raw.githubusercontent.com/VanMo/quatrojet/Releases/<apk из update.json>
```

Константы заданы в `Model/update/checker.py` (`MANIFEST_URL`, `APK_URL_TEMPLATE`);
на время отладки адрес можно переопределить ключом `update.url` в `client.yaml`.

## Как опубликовать новую версию

1. Поднять версию в `_version.py` и собрать подписанный APK:
   `./build_app_release.sh` → `bin/qjpy-<версия>-arm64-v8a-release.apk`.
2. Перейти в ветку и положить APK рядом с `update.json`:
   ```bash
   git switch Releases
   cp ../bin/qjpy-0.4.8-arm64-v8a-release.apk .
   sha256sum qjpy-0.4.8-arm64-v8a-release.apk
   ```
3. Обновить `update.json` (версия, имя файла, `sha256`, `notes`).
4. `git add . && git commit -m "Релиз 0.4.8" && git push`.
5. Проверить, что `raw`-адреса отвечают:
   ```bash
   curl -s https://raw.githubusercontent.com/VanMo/quatrojet/Releases/update.json
   ```

Порядок важен: сначала загружается APK, затем `update.json`. Иначе приложение
увидит новую версию, но не сможет скачать файл.
