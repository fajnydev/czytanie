# Czytanie‑App

Aplikacja webowa (Vue 3) pomagająca dziecku ćwiczyć czytanie – od pojedynczych liter, przez sylaby, do prostych słów.

## Szybki start (lokalnie)

```bash
npm install
npm run serve
```

Aplikacja będzie dostępna pod `http://localhost:8080`.
Dostępna też na githubPages

## Budowanie i publikacja na GitHub Pages

1. Edytuj `vue.config.js`, ustawiając `publicPath` na `'/nazwa‑repozytorium/'`.
2. Zbuduj projekt:

```bash
npm run build
```

3. Opublikuj:

```bash
npm run deploy
```

> Komenda korzysta z pakietu **gh-pages** i wypycha zbudowany katalog _dist/_ na gałąź **gh-pages**.