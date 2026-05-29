# Nothing Held Back Documentation

Official NHB documentation site. Source markdown lives in [`nhb-docs/`](./nhb-docs/) and is published to GitHub Pages via [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

**Live site:** https://natalie-alphafour.github.io/nhb-documentation/

## Editing

Edit any `.md` file under `nhb-docs/` and push to `main`. The GitHub Actions workflow at `.github/workflows/deploy.yml` will rebuild and redeploy automatically.

To add a new page, drop it into the appropriate folder and add an entry under `nav:` in [`mkdocs.yml`](./mkdocs.yml).

## Local preview

```bash
pip install -r requirements.txt
mkdocs serve
```

Then open http://127.0.0.1:8000/.
