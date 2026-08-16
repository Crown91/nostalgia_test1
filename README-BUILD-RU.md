# Сборка Alpha Protocol (nostalgia) без composite builds

## Почему в libs/ лежат sodium, punchy и zemlya, а sha нет

Автор подключал зависимости двумя разными способами:

- `sodium` и `punchy` — локальными jar из `libs/` (поэтому они в репозитории);
- `sha` и `zemlya` — через composite build в `settings.gradle`
  (`includeBuild('../SHA')`, `includeBuild('../Zemlya')`) плюс координаты
  `net.sha:sha:1.0.7` и `net.zemlya:zemlya:1.0.0`.

То есть исходная сборка требовала, чтобы рядом с папкой проекта лежали ещё два
репозитория — `../SHA` и `../Zemlya`. На GitHub Actions их нет, и Gradle падает ещё до
компиляции. `zemlya-1.0.0.jar` в `libs/` лежит как остаток прежнего подхода, а `sha.jar`
автор там держать перестал — оттуда и асимметрия.

## Что изменено

- `settings.gradle`: убраны `includeBuild('../SHA')` и `includeBuild('../Zemlya')`.
- `build.gradle`: вместо maven-координат используются jar из `libs/`:

```gradle
compileOnly files("libs/sha.jar")
compileOnly files("libs/zemlya-1.0.0.jar")
compileOnly files("libs/sodium.jar", "libs/punchy.jar")
```

  Ремаппинг не нужен: MC 26.1.x необфусцирован и в `gradle.properties` стоит
  `fabric.loom.disableObfuscation=true`.
- Добавлена проверка, которая пишет внятную ошибку при отсутствии jar в `libs/`.
- `.github/workflows/build-jar.yml`: JDK 25, проверка входных файлов, артефакт
  `nostalgia-mod-jar`, логи `build-logs` выгружаются всегда.

## Что нужно сделать

1. Собрать `sha_test` → скачать артефакт `sha-mod-jar` → переименовать jar в `sha.jar`.
2. Загрузить его сюда как `libs/sha.jar`.
3. Actions → `build-jar` → Run workflow → артефакт `nostalgia-mod-jar`.

## Важно про установку

`sha` и `zemlya` теперь `compileOnly`, а не `include` (jar-in-jar). В `.minecraft/mods`
нужно класть три файла: `alpha_protocol-1.0.0.jar`, `sha.jar`, `zemlya-1.0.0.jar`,
иначе будет `NoClassDefFoundError: net/sha/...`.
