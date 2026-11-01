# Quality gates

Everything is dependency-free Node (22 or later).

## What runs

| Check | Command | Covers |
|---|---|---|
| Secret scan | `node tools/quality/scan-secrets.mjs` | Credential formats in tracked files |
| Hygiene | `node tools/quality/check-hygiene.mjs` | Tracked secrets and oversized files, `.only`, fake test scripts, workflows that can falsely pass |
| Tool tests | `node --test "tools/quality/*.test.mjs"` | The checks above and below, including that each rule fails on a real defect |
| Site checks | `node tools/quality/check-site.mjs` | Doctype, lang, charset, viewport, title, description, one h1, tag balance, duplicate ids, alt text, labels, button and link names, internal links and fragments, required files, CNAME, what deployment would publish |
| Smoke | `node tools/quality/smoke-site.mjs` | Serves tracked files and requests the page; inline scripts must parse; key content must be present |

CI: `.github/workflows/quality.yml`. Settings: `tools/quality/site.config.json`.

## Deployment has no gate

GitHub Pages deploys this repository from the branch ("pages build and deployment"), not from a workflow, so **no job ordering is possible: a red quality run does not stop a push to `main` going live.** To make the gate binding, either switch Pages to a workflow deployment with `needs: quality`, or protect `main` with this check as required.

Because Pages builds the branch with Jekyll, `_config.yml` excludes `tools` and `docs` so this tooling stays off the public site. Jekyll excludes dotfiles by default.

## Run on Windows

```powershell
node tools/quality/scan-secrets.mjs
node tools/quality/check-hygiene.mjs
node --test "tools/quality/*.test.mjs"
node tools/quality/check-site.mjs
node tools/quality/smoke-site.mjs
```

## Install hooks

```powershell
git config core.hooksPath .githooks
```

`pre-commit` runs the secret scan on staged content, hygiene and site checks. `pre-push` runs everything.

## Not covered

- No real browser: inline script is parsed, not executed. The smooth-scroll handlers, the contact mail link and layout are not exercised.
- No colour contrast, keyboard or screen-reader testing. The accessibility checks are structural.
- External links, including the font CDN and the mail address, are not requested.
- The Jekyll exclusion cannot be run locally. After merging, confirm that https://mindmachine.io/tools/quality/scan-secrets.mjs returns 404.
- Em dashes and the word "LLM" in the existing copy are not gated.
