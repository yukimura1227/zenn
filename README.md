# Zenn CLI

* [📘 How to use](https://zenn.dev/zenn/articles/zenn-cli-guide)

## HOW TO

### New Article

```bash
docker compose run --rm zenn npx zenn new:article
```

```bash
TITLE="タイトルを書く"
TYPE="tech" # tech:技術記事 or idea:アイデア記事
SLUG="識別子をここに"
docker compose run --rm zenn npx zenn new:article  --title ${TITLE} --type ${TYPE} --slug ${SLUG}
```

### Preview

```bash
docker compose up
# -> then access http://localhost:8000
```