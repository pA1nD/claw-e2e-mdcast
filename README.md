# mdcast

> Cast markdown to HTML.
> A small, strict, deterministic Markdown-to-HTML CLI.

---

`mdcast` converts Markdown to HTML. That's it.

No templating. No baked-in CSS. No GitHub corners, no favicons, no opinionated page wrappers. Fragment output by default — full document behind a flag. Byte-deterministic output across runs. Installs as a single npm package with zero transitive opinion.

Built for the common case: you have `.md`, you need `.html`, you want to pipe it somewhere and forget about it.

---

## Why this exists

The Markdown-to-HTML space has three kinds of tools, and none of them fit the common case cleanly.

### The universal converter — Pandoc

[Pandoc](https://pandoc.org) converts ~30 document formats to ~30 other document formats. It is the correct answer if you're converting a LaTeX thesis to EPUB, or DocBook to MediaWiki. For "README.md → README.html" it is a ~100 MB Haskell install, a non-trivial invocation surface, and output that varies subtly between pandoc versions. Overkill, and a dependency footprint no static-site CI wants.

### The library, not the tool — markdown-it, marked, commonmark.js, remark

[markdown-it](https://github.com/markdown-it/markdown-it), [marked](https://github.com/markedjs/marked), [commonmark.js](https://github.com/commonmark/commonmark.js), and [remark](https://github.com/remarkjs/remark) are JavaScript parser libraries. They are fast, well-maintained, and the engines that power most of the web's markdown rendering. But they are libraries — not CLIs. To use them from the shell you write a wrapper: read stdin, parse, render, write stdout. Every project reinvents that wrapper. The wrapper is five lines; the packaging, argument parsing, error handling, and stdin/file-mode logic that surround it is a hundred.

### The opinionated CLI — markdown-to-html-cli and friends

A handful of CLIs do wrap those libraries: [jaywcjlove/markdown-to-html-cli](https://github.com/jaywcjlove/markdown-to-html-cli), [BaseMax/markdown-to-html-converter](https://github.com/BaseMax/markdown-to-html-converter), and several web-based converters (DevPlaybook, Dillinger, StackEdit). Most conflate conversion with page generation — they wrap the output in a full `<html>` document with their own CSS, inject GitHub-corner widgets, add favicons, embed social-media meta tags. If you want a clean `<h1>Hello</h1>` fragment to paste into a template you already own, you're fighting the tool.

## The gap

A converter that:

- Does exactly one thing: Markdown → HTML.
- Emits HTML fragments by default — just the content, no document shell.
- Emits byte-identical output for byte-identical input, every time. CI diffs are meaningful. Static-site caches invalidate correctly.
- Is CommonMark-strict, with opt-in GFM extensions (tables, task lists, strikethrough, autolinks).
- Ships as one small npm package with no transitive dependencies that leak styling, templating, or network calls.
- Has a CLI that respects Unix conventions: stdin/stdout by default, files with flags, exit codes that mean something.

That's `mdcast`.

---

## Installation

```bash
npm install -g mdcast
```

Or run without installing:

```bash
npx mdcast README.md
```

Node.js 20+. No other runtime requirements.

---

## Usage

### Pipe from stdin, write to stdout

```bash
cat README.md | mdcast > README.html
```

### Read a file, write a file

```bash
mdcast --input README.md --output README.html
```

### Emit a full HTML document instead of a fragment

```bash
mdcast --input README.md --doc --output README.html
```

`--doc` wraps the fragment in a minimal `<!doctype html>` shell with a `<title>` derived from the first `<h1>`. No CSS, no scripts. If you want styling, pipe the output through a template of your own.

### Enable GitHub-flavoured extensions

```bash
mdcast --input README.md --gfm
```

Adds tables, task lists (`- [ ]`), strikethrough (`~~text~~`), and autolinks.

### Verify determinism

Run twice, diff the output. It should be empty:

```bash
mdcast --input README.md > a.html
mdcast --input README.md > b.html
diff a.html b.html
```

If it isn't, that's a bug — please file an issue.

---

## What mdcast doesn't do

- No syntax highlighting. Use a downstream highlighter (Prism, Shiki, highlight.js) on the emitted `<pre><code>` blocks.
- No HTML sanitization. The input is trusted by default. If you're rendering user-supplied Markdown, sanitize the output with DOMPurify or a server-side equivalent.
- No templating, includes, or front-matter parsing. Feed it Markdown, get HTML. Anything more is your pipeline's job.
- No inline CSS or JavaScript injection. Output is structure only.
- No network calls. No analytics. No telemetry. mdcast runs offline forever.

These aren't planned features. They're deliberate omissions.

---

## Exit codes

| Code | Meaning |
|---|---|
| 0 | Conversion succeeded. |
| 1 | Conversion failed — stdin/file unreadable or Markdown too malformed to parse. |
| 2 | Invalid arguments — unknown flag, missing required value, or conflicting options. |

---

## Development

```bash
git clone https://github.com/pA1nD/claw-e2e-mdcast.git
cd claw-e2e-mdcast
npm install
npm test
npm run build
```

Tests live in `tests/`, one file per module. The suite is the source of truth — behaviour changes that break existing tests need a migration note in the relevant issue.

---

## Project status

See [ROADMAP.md](./ROADMAP.md). `v0.1` is Foundation — the core parser, CLI, and fragment/document output modes. Later milestones add ergonomics, extensibility, performance work, and strict spec compliance.

Contributions welcome on open issues. New features should land as GitHub issues first so the scope and acceptance criteria are agreed before code is written.

---

## License

MIT.
