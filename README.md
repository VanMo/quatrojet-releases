# Канал обновлений Quatrojet

Публичный репозиторий `VanMo/quatrojet-releases`, ветка `main`. Здесь лежит
только то, что скачивает само приложение:

| Файл | Назначение |
|---|---|
| `update.json` | описание последней версии |
| `qjpy-<версия>-arm64-v8a-release.apk` | подписанный release-APK этой версии |

Содержимое не связано с историей кода (`VanMo/quatrojet`): APK — большой
бинарный файл, и держать его в основной ветке незачем. Репозиторий канала
обязательно публичный, иначе анонимный `raw`-адрес отдаёт 404.

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
https://raw.githubusercontent.com/VanMo/quatrojet-releases/main/update.json
https://raw.githubusercontent.com/VanMo/quatrojet-releases/main/<apk из update.json>
```

Адрес описан в `Model/update/checker.py` (`MANIFEST_URL`), имя APK берётся из
самого `update.json`. На время отладки адрес можно переопределить ключом
`update.url` в `client.yaml`.

**Репозиторий должен быть публичным.** Анонимный запрос к `raw.githubusercontent.com`
для приватного репозитория возвращает 404, и приложение покажет «Сервер обновлений
ответил кодом 404». Токен в приложение не встраивается: любой APK можно разобрать.

## Как опубликовать новую версию

1. Поднять версию в `_version.py` и собрать подписанный APK:
   `./build_app_release.sh` → `bin/qjpy-<версия>-arm64-v8a-release.apk`.
2. Перейти в ветку `Releases` приватного репозитория (в основном дереве или в
   отдельной рабочей копии) и выполнить скрипт:
   ```bash
   .venv/bin/python scripts/make_update_manifest.py \
       --apk bin/qjpy-0.4.8-arm64-v8a-release.apk \
       --notes "Что нового" --to .
   ```
   Скрипт копирует APK, считает `sha256` и пишет `update.json`.
3. `git add . && git commit -m "Релиз 0.4.8"`.
4. Отправить содержимое в публичный канал:
   ```bash
   git push <адрес quatrojet-releases> Releases:main
   ```
5. Проверить, что `raw`-адреса отвечают **без авторизации**:
   ```bash
   curl -s -o /dev/null -w "%{http_code}\n" \
       https://raw.githubusercontent.com/VanMo/quatrojet-releases/main/update.json
   ```

Порядок важен: сначала загружается APK, затем `update.json`. Иначе приложение
увидит новую версию, но не сможет скачать файл.
