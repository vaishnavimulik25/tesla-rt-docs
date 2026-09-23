# Tesla RT (`rt`) documentation

FreeRTOS-style documentation for the open-source **RTNG / `rt`** kernel
([https://git.rtng.org/rt/rt](https://git.rtng.org/rt/rt), crates.io `rt`, Apache-2.0).

These pages cover kernel features, public APIs, examples, supported devices from
[https://git.rtng.org/rt](https://git.rtng.org/rt) port repos, and what benchmarks exist in-tree.

**Scope:** open-source kernel and public BSP repositories only. Partner materials may call this “Tesla’s RTOS”; this repo does **not** claim closed vehicle production software.

## Browse the HTML site

Prebuilt static site is in [`site/`](site/). After cloning:

```bash
cd site
python3 -m http.server 8000
# open http://127.0.0.1:8000/
```

Or open `site/index.html` directly in a browser (some browsers restrict `file://` search).

## Edit markdown / rebuild

Sources live under [`docs/`](docs/). Rebuild with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/):

```bash
python3 -m venv .venv
.venv/bin/pip install mkdocs-material
.venv/bin/mkdocs build
# output in site/
```

## Upstream

- Kernel: https://git.rtng.org/rt/rt
- Port / board repos: https://git.rtng.org/rt
