# CLAUDE.md — Patch scripts

Refines the root `CLAUDE.md` for `scripts/`. These are the small Python/bash patchers used to edit the big single-file apps safely, instead of hand-editing thousands of lines.

## The patch pattern (matches the root editing workflow)

1. Read the target app file. Split JS from HTML on the **last** `<script>`: `idx = content.rfind('<script>')`, `html_part = content[:idx]`, `js = content[idx+len('<script>'):]` (then trim the trailing `</script>...</html>` tail you'll re-add).
2. **`assert` the target substring exists before every replacement.** If a target string isn't found, the script must fail loudly — silent no-op edits are how drift sneaks in.
3. HTML/CSS changes edit `html_part`; JS changes edit `js`. Recombine as:
   `html_part + '<script>\n' + js + '\n\n</body>\n</html>'`.
4. **`node --check` the JS** (or the whole recombined file) before writing output. Do not write a file that doesn't parse.
5. Bump `const VER` and **prepend** a `CHANGELOG` entry — **ASCII only** (em-dash / ellipsis / glyphs have silently truncated the Android changelog).
6. Run the ID audit: compare `id="..."` in HTML against DOM lookups in JS; on the tablet build also confirm the `t-` twins.

## Conventions

- Prefer editing to a **new versioned filename** for apps that version their filename; otherwise write in place after `node --check` passes.
- Keep scripts self-contained and dependency-free (stdlib Python / plain bash) — matches the no-npm, no-build ethos of the repo.
- One script = one clearly-described change; name it for what it does. Leave a short comment block at the top stating the target file and the edit.
