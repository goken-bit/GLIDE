# GLIDE — a minimalist IDE for [GLang](https://github.com/GLang/GLang)

A single-file, in-browser IDE for GLang. You type `.g` code, hit **Build**, and GitHub
Actions compiles runs it on a Linux runner. Results (compile errors, build status, console
output) come back into the editor; the compiled binary is downloadable.

No servers, no databases — the "compiler server" *is* GitHub Actions.

## How it works

1. `index.html` (CodeMirror + a small GLang syntax mode) is deployed to **GitHub Pages**
   by `pages-deploy.yml`.
2. Clicking **Build** POSTs your code to your repo with a
   [`repository_dispatch`](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event)
   event, carrying your real PAT from the browser (stored in `localStorage` only).
3. `build.yml` picks up the dispatch, compiles with the real `glangc.py` (cloned from
   your `GLang` repo), runs console programs headless, then pushes the result to the
   `glide-results` branch.
4. The IDE polls
   `raw.githubusercontent.com/<owner>/<repo>/glide-results/<build-id>/result.txt`
   and shows the output.

## Setup (one time)

1. Create a GitHub repo for GLIDE (e.g. `GLIDE`), and make sure your **GLang repo is
   public** and uses the same owner, or edit the clone URL in `build.yml`.
   (Only the *GLIDE* repo needs to be public if you want to visit the IDE without logging in.)
2. Push this folder to the repo:
   ```
   git init && git add -A && git commit -m "GLIDE" && git push -u origin main
   ```
3. In repo **Settings → Pages**: Source = *GitHub Actions*. The first `push` triggers
   the Pages deployment.
4. Create a **PAT with `repo` scope** (GitHub → Settings → Developer settings → Personal
   access tokens). Paste it into the box in the IDE — it never leaves your browser.

## Using it

- **Build** — dispatches a build, waits for the result (SDK games: binary only; console
  programs: binary + stdout).
- **Download binary** — the compiled ELF from `glide-results/<id>/main`. Needs SDL2 on
  your machine to run SDL games (on Termux: `pkg install sdl2`).
- New / Save — blank editor / download current file.

## Notes / limits

- `repository_dispatch` payloads are size-limited (tens of KB) — keep `.g` files under
  ~50 KB.
- One build at a time (concurrency group); results land in order.
- `glide-results` is a scratch branch holding every build's binary — prune it whenever.
- CodeMirror is vendored in `vendor/` — the IDE is fully self-contained (no CDN).