# Tesla RT (`rt`) documentation

FreeRTOS-style documentation for the open-source **RTNG / `rt`** kernel
([https://git.rtng.org/rt/rt](https://git.rtng.org/rt/rt), crates.io `rt`, Apache-2.0).

**Browse online (GitHub Pages):** https://vaishnavimulik25.github.io/tesla-rt-docs/

**Scope:** open-source kernel and public BSP repositories only. Partner materials may call this “Tesla’s RTOS”; this repo does **not** claim closed vehicle production software.

## What’s in the repo

| Path | Contents |
|------|----------|
| [`docs/`](docs/) | Prebuilt HTML site (also published via GitHub Pages) |
| [`markdown/`](markdown/) | Editable MkDocs markdown sources |
| [`site/`](site/) | Same prebuilt HTML (local mirror of `docs/`) |
| `mkdocs.yml` | MkDocs Material config (`docs_dir: markdown`) |


HTML pages are **flat `.html` files** (e.g. `01-Introduction.html`, `07-Examples/cond.html`) so you can open a page with one click instead of entering a folder and then opening `index.html`.

## Browse locally

```bash
cd docs   # or: cd site
python3 -m http.server 8000
# open http://127.0.0.1:8000/
```

## Rebuild from markdown

```bash
python3 -m venv .venv
.venv/bin/pip install mkdocs-material
.venv/bin/mkdocs build
# then sync: cp -a site/. docs/
```

## Upstream

- Kernel: https://git.rtng.org/rt/rt
- Port / board repos: https://git.rtng.org/rt
