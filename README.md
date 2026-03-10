# AIpresentation on GitHub Pages

Репозиторий подготовлен для публикации через GitHub Pages.

## Что уже настроено

- `/.github/workflows/pages.yml` — автодеплой на GitHub Pages при пуше в `main` или `master`.
- `/index.html` — основная (новая) версия презентации.
- `/index_v2.html` — копия новой версии по отдельному URL.
- `/index_legacy.html` — предыдущая версия презентации.
- `/text/index.html` — отдельный URL с текстовой версией (рендер из `part1_foundations.md`).
- `/part1_foundations.md` — прямой URL к исходному markdown-файлу.

## Какие будут URL после публикации

Замените `<USERNAME>` и `<REPO>`:

- Презентация: `https://<USERNAME>.github.io/<REPO>/`
- Презентация (v2 URL): `https://<USERNAME>.github.io/<REPO>/index_v2.html`
- Презентация (legacy): `https://<USERNAME>.github.io/<REPO>/index_legacy.html`
- Текстовая страница: `https://<USERNAME>.github.io/<REPO>/text/`
- Исходный markdown: `https://<USERNAME>.github.io/<REPO>/part1_foundations.md`

## Как включить Pages в GitHub

1. Запушить репозиторий в GitHub.
2. Открыть `Settings -> Pages`.
3. В `Build and deployment` выбрать `Source: GitHub Actions`.
4. Дождаться завершения workflow `Deploy to GitHub Pages`.
