# mdcast — Roadmap

> **Cast markdown to HTML.**
>
> A small, strict, deterministic Markdown-to-HTML CLI.
> Fragment output by default. CommonMark-first. Zero templating.

---

## Current milestone: v0.1 — Foundation

---

## v0.1 — Foundation
*Core conversion. One thing, done correctly.*

The first milestone gets mdcast from nothing to a usable CLI. The scope is deliberately narrow: reading input, parsing Markdown to HTML, writing output — with the ergonomics and determinism guarantees promised in the README.

### User stories
- As a developer, I pipe Markdown to mdcast on stdin and get HTML on stdout — no flags needed
- As a static-site author, I run `mdcast --input README.md --output README.html` in CI and the output is byte-identical every run
- As a template author, I get a clean HTML fragment by default — no `<html>`, no `<body>`, no injected CSS
- As someone publishing a standalone page, I pass `--doc` and get a minimal valid HTML5 document with the right `<title>`
- As a GitHub user, I pass `--gfm` and get tables, task lists, strikethrough, and autolinks

### What it does

**CLI entry point**
- `mdcast` binary installed via `npm install -g mdcast`
- Flags: `--input <path>`, `--output <path>`, `--doc`, `--gfm`, `--help`, `--version`
- Stdin when `--input` is omitted, stdout when `--output` is omitted
- Exit codes per the README spec (0 success, 1 runtime failure, 2 arg error)

**Parser**
- Wraps a CommonMark-compliant parser library
- Takes a Markdown string, returns an HTML string
- `--gfm` toggles GFM extensions (tables, task lists, strikethrough, autolinks)
- Output is byte-deterministic: identical input produces identical output bytes, always

**Document mode**
- `--doc` wraps the fragment in a minimal `<!doctype html>` shell
- `<title>` is the text content of the first `<h1>` in the document; falls back to "Document" when no `<h1>` exists
- No CSS, no meta tags beyond `<meta charset="utf-8">`, no scripts

**Error handling**
- Unreadable input (missing file, bad permissions) → exit 1 with message on stderr
- Unknown flag, missing required value → exit 2 with usage hint
- Parser exceptions → exit 1 with message on stderr, no stack trace leakage

### Tech stack

| Layer | Choice |
|---|---|
| Language | TypeScript (strict) throughout |
| Runtime | Node.js 20+ |
| Parser | `markdown-it` with GFM plugins (tables, task-lists, strikethrough, linkify) |
| CLI args | `commander` |
| Tests | `vitest` |

### Done when
`mdcast` can convert the project's own `README.md` to HTML via stdin, via file flags, in fragment mode, and in document mode — with byte-deterministic output across 100 consecutive runs.

---

## v0.2 — Ergonomics
*The parts users hit on day two.*

With the core converter in place, v0.2 addresses the friction points that appear once people actually use the tool in a pipeline.

### User stories
- As a CI author, I point mdcast at a config file and avoid repeating flags across build steps
- As a docs maintainer, I run `mdcast --watch` during authoring and see rebuilds within 100ms
- As a blogger, I batch-convert an entire directory of `.md` files with one command
- As a pipeline author, I chain mdcast with other tools via predictable stdin/stdout behaviour

### What it does
- Config file support at `.mdcastrc` or `mdcast.config.json` — flags become defaults
- `--watch <dir>` rebuilds touched files on change
- `--input <dir>` expands to "every `.md` file in the directory"; `--output <dir>` mirrors the tree with `.html` extensions
- Source map output via `--source-map` — line-level mapping from HTML nodes back to Markdown offsets, for editor integrations

### Done when
A docs directory of 50 Markdown files can be batch-converted in under one second, and `--watch` surfaces an updated HTML file within 100ms of a save.

---

## v0.3 — Extensibility
*Plugins without ceremony.*

v0.3 opens the renderer to third-party extensions, but resists the usual "plugin ecosystem" bloat. Plugins are just functions — no manifests, no discovery protocol, no registry.

### User stories
- As a syntax-highlighting user, I inject a highlighter (Shiki, Prism, Starry Night) via a single hook
- As a publisher, I rewrite link URLs at render time without forking the parser
- As a sanitization-conscious author, I wire in DOMPurify via the same hook mechanism

### What it does
- `--plugin <path>` loads a JavaScript module that exports `{ tokens, renderer }` transforms
- Plugins receive the token stream pre-render and the HTML string post-render
- Bundled plugin: `mdcast/plugins/highlight` — a thin wrapper around Shiki that adds `class="language-*"` and pre-highlighted `<span>` trees
- Plugins compose: `--plugin a.js --plugin b.js` applies them in order

### Done when
A user can install a third-party plugin via `npm install` and activate it via a single `--plugin` flag — no other configuration required.

---

## v0.4 — Performance
*Fast enough to not think about.*

v0.4 makes mdcast competitive on throughput with the fastest JS Markdown parsers, and introduces a benchmark suite that runs in CI so regressions are caught before release.

### User stories
- As a static-site builder, I convert 10,000 Markdown files in under 10 seconds
- As a contributor, I see a benchmark delta on every PR — so a 2x slowdown cannot land unnoticed
- As a memory-constrained user, I convert a 50MB Markdown file without the process heap exceeding 200MB

### What it does
- Streaming parser mode — `--stream` reads input chunks and emits HTML chunks, bounded memory
- Incremental conversion cache — when `--watch` is active, only changed blocks re-render
- `npm run bench` suite with fixtures of 10B, 1KB, 10KB, 100KB, 1MB, 10MB, 100MB — reported in ops/sec and MB/sec
- Benchmark comparison against marked, markdown-it, and commonmark.js — published as a CI artifact on every PR

### Done when
Throughput on a 1MB fixture is within 20% of raw markdown-it, and the benchmark suite runs cleanly in CI with per-PR regression detection.

---

## v0.5 — Correctness
*Strict compliance with published specs.*

The final foundation-tier milestone tightens mdcast's output to match published Markdown specifications character-for-character, verified against the official test suites.

### User stories
- As a publisher targeting CommonMark-rendering platforms, I trust that mdcast's output matches the spec byte-for-byte
- As a GFM user, I rely on GitHub's rendering behaviour being mirrored exactly — including corner cases in nested tables and task lists
- As an MDX user, I opt into a strict subset of JSX-in-Markdown without pulling in a full MDX toolchain

### What it does
- CommonMark 0.31.2 spec test suite passes 100% with `--strict`
- GFM 0.29-gfm spec test suite passes 100% with `--gfm --strict`
- Optional MDX subset: `--mdx` enables `<Component />` passthrough for JSX tags in Markdown, with all other MDX semantics disabled
- Spec-conformance report published as `COMPLIANCE.md` and updated on every release

### Done when
All three spec suites pass in CI, and the compliance report is automatically generated as part of the release process.

---

## File footprint

```
src/
  parser/      Markdown → HTML conversion
  cli/         commander entry, flag handling, stdin/stdout wiring
  document/    --doc mode wrapper
  types/       shared TypeScript types
tests/
  parser/
  cli/
  document/
```

Strict TypeScript. Every public function carries a JSDoc comment. No `any`. No `@ts-ignore`.

---

## How issues map to milestones

Every issue carries the milestone label it belongs to (`v0.1`, `v0.2`, ...). The current-milestone-runner convention lives in the label itself: run N of milestone `v0.1` uses the label `v0.1-NNN` on copied issues. The product milestone labels on this ROADMAP (`v0.1`, `v0.2`, ...) are the stable anchors; run labels come and go per implementation attempt.

---

## Contributing

Open an issue first. The issue body is the spec — PRs without a linked, accepted issue will not merge. Issues must include acceptance criteria; vague "make it better" tickets will be closed with a request for specificity.

No direct commits to `main`. Branch per issue, PR per branch, squash merge with `Closes #N` in the commit title.
