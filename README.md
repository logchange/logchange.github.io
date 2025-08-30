# logchange.github.io

https://logchange.github.io/

## 📚 Organization Docs Site

This repository now hosts the *logchange documentation* site built with MkDocs (Material theme).

- Source docs: ./docs
- Config: ./mkdocs.yml

### Local preview

```bash
pip install -r requirements.txt
mkdocs serve
```

### Contributing

- Use the pencil icon on any page to edit directly on GitHub (it links to the file under ./docs).
- See docs/Contributing: https://logchange.github.io/.github/contributing/

### Structure

- Organization overview: docs/index.md
- Tools: docs/tools/{logchange|hofund|valhalla}
  - getting-started.md
  - usage.md
  - faq.md
  - reference.md

### Deployment

Publishing is automated with GitHub Actions on push to main. The workflow builds MkDocs, uploads the artifact, and deploys to GitHub Pages.