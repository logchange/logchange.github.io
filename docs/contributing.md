# Contributing to logchange Docs

Thank you for helping improve our documentation!

## Edit a page

- Click the pencil icon in the top-right of any page to edit directly on GitHub.
- Propose changes via a Pull Request.

## Run the site locally

Prerequisites:
- Python 3.8+

Install and serve:

```bash
pip install -r requirements.txt
mkdocs serve
```

Open http://127.0.0.1:8000 in your browser. Changes reload automatically.

## Content guidelines

- Keep Getting Started concise with prerequisites, installation, and a quick start example.
- Put detailed commands and options under Reference.
- Add examples and troubleshooting notes under Usage and FAQ.

## Build and deploy

- On merge to `main`, GitHub Actions builds and deploys the site to GitHub Pages automatically.
- The site is published at: https://logchange.github.io/ 