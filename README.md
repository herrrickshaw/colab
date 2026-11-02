# Colab runner template

A single Jupyter notebook (`colab_runner_template.ipynb`) that clones any public GitHub
repository into a Google Colab GPU / high-RAM runtime, installs every `requirements*.txt`
it declares, and runs one command — either as a batch job or as a long-running server exposed
through a Cloudflare quick tunnel. It is a personal harness for offloading heavy jobs
(large data crunches, GPU-only models such as Unlimited-OCR, long batch runs) from a Mac to
Colab. It produces no artefacts of its own; whatever the wrapped command writes is the output.

> Status: maintained — notebook-only drop. There is no Python package, no tests and no
> data in this repo; the notebook is the whole product.

## What it does
- Clones `REPO` at `BRANCH` (shallow, depth 1) and optionally `cd`s into `SUBDIR`.
- Installs every `requirements*.txt` found in that directory (including `requirements-ocr.txt`
  when present), so Colab mirrors the repo's declared dependencies.
- Prints `nvidia-smi -L` so you can confirm the GPU runtime took effect.
- `SERVE_PORT = 0` (default): runs `CMD` once as a batch job in the notebook.
- `SERVE_PORT > 0`: starts `CMD` in the background, polls `http://localhost:<port>` for up to
  30 minutes, then downloads `cloudflared` and prints a public `*.trycloudflare.com` URL.

## Quick start
```bash
git clone https://github.com/herrrickshaw/colab.git
cd colab
```
1. Open `colab_runner_template.ipynb` in Google Colab (File → Upload notebook, or open from GitHub).
2. Edit the `PARAMS` cell: `REPO`, `BRANCH`, `SUBDIR`, `CMD`, `SERVE_PORT`.
3. Runtime → Change runtime type → GPU, if the job needs one.
4. Runtime → Run all.

The shipped defaults clone `herrrickshaw/global-market-research-platform` on `main` and run
`python3 --version`, which is a harmless smoke test.

## How it works
| Cell | What it does |
|---|---|
| PARAMS | Five plain Python variables; the only cell you edit |
| Setup | `nvidia-smi -L`, `git clone --depth 1`, loop over `requirements*.txt` with `pip -q install` (`check=False`, so one bad requirements file does not abort the run) |
| Run | Batch: `!{CMD}`. Server: `subprocess.Popen(CMD)` → readiness poll → `cloudflared tunnel --url` → regex the public URL out of its log |

Design choices: the dependency list is deliberately not duplicated here — the notebook installs
whatever the target repo declares, so it never drifts from that repo. Cloudflare quick tunnels
need no account or token, which is why they were chosen over ngrok.

## Repository layout
| Path | Purpose |
|---|---|
| `colab_runner_template.ipynb` | The harness (4 cells, no saved outputs) |
| `HOW_TO_USE_WITH_AI.md` | Generic guide for non-programmers: hand the repo to an AI assistant |
| `CLAUDE.md` | Conventions for Claude Code sessions (source-fetching rules) |
| `.github/workflows/quality-gates.yml` | gitleaks secret scan on every push / PR |
| `graphify-out/` | Generated knowledge-graph cache; not hand-edited |

## Configuration
All configuration is the `PARAMS` cell:

| Variable | Default | Meaning |
|---|---|---|
| `REPO` | `herrrickshaw/global-market-research-platform` | Public GitHub repo, `owner/name` |
| `BRANCH` | `main` | Branch to clone |
| `SUBDIR` | `''` | Optional working directory inside the clone |
| `CMD` | `python3 --version` | Shell command to run |
| `SERVE_PORT` | `0` | `>0` turns on server mode + tunnel |

No secrets are needed to run the template. Pushing results back from Colab needs a GitHub
token; supply it inside the Colab session only — never commit it to this repo.

## Limitations and known issues
- Only public repositories can be cloned as written (`https://github.com/{REPO}.git`, no auth).
- Free Colab disconnects after a few hours; keep the tab open for servers and download batch
  outputs via the Files pane or Dropbox before the runtime dies.
- `pip install` failures are swallowed (`check=False`); read the cell log if a later import fails.
- The server readiness poll times out silently after 180 × 10 s — the tunnel is started regardless.
- `cloudflared` is fetched from its `releases/latest` URL on every run, so the binary version is
  not pinned.

## Related repositories
- [global-market-research-platform](https://github.com/herrrickshaw/global-market-research-platform) — the default `REPO` in the PARAMS cell.

## Licence
No licence file yet — all rights reserved.
