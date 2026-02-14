---
title: "Open Source Tech of the Day: Nushell"
pubDate: 2026-02-14
description: "A modern shell where data is structured by default—pipelines you can actually trust."
---

If you’ve ever piped some command output into `grep | awk | sed` and then spent 20 minutes wondering why your “simple one-liner” turned into a fragile origami swan… today’s pick is for you.

## Quick tour

**Nushell** (often written as **Nu**) is an open-source shell that treats command output as **structured data** instead of raw text. That means your pipeline steps can work with real tables/records, not “maybe-it’s-tab-separated” strings.

A tiny taste:

- List files as a table: `ls`
- Filter rows: `ls | where type == "file"`
- Sort by size: `ls | sort-by size`
- Export clean JSON/CSV: `ls | select name size modified | to json`

It still plays nicely with your existing command-line tools—when you call an external command, Nu can capture the output and let you keep working.

Practical next steps (a.k.a. “links you’ll actually click”):

- A friendly intro: the Nu **Book** (docs)
- The **GitHub repo** for releases/issues
- A deeper “why structured pipelines matter” explainer in the docs

(Actual URLs are in the **Links** section at the end.)

## What problem it solves

Classic shells are incredible… and also kind of wild.

Most commands print text meant for humans. Then we pipe that text into tools that try to interpret it. It works, until it doesn’t: localized dates, weird spacing, columns that shift, file names with spaces, or command output that changes subtly between versions.

Nushell aims to make the “plumbing” part of your terminal **predictable**:

- Output is typically a **table** (rows + columns), not a blob.
- You can **filter, group, join, and transform** data with a consistent set of commands.
- You can **serialize** data without regex gymnastics.

In short: fewer brittle one-liners, more reusable mini-programs.

## Why it’s cool (standout features)

### 1) Structured pipelines by default

The headline feature: pipelines pass rich values around. When you run `ls`, you get columns like `name`, `size`, and `modified`. You can reliably say “show me files larger than 50MB” without parsing.

Example:

```nu
ls | where size > 50mb | sort-by size | select name size
```

### 2) Great “data wrangling” primitives

Nu ships with a bunch of commands that feel like a tiny data toolkit living in your shell:

- `open` to read structured files (JSON, CSV, YAML, TOML)
- `get`, `select`, `rename`, `update` for shaping data
- `to json`, `to csv`, `to yaml` to export

So “inspect a JSON API response” or “massage a CSV before importing” can be a 10-second terminal task.

### 3) Ergonomic, modern shell UX

Nushell has a lot of the quality-of-life stuff you’d expect from a modern environment:

- Helpful errors (less cryptic incantation, more “here’s what went wrong”)
- Good completions and discoverability
- A consistent command style that’s easier to remember than ten different flags

And when you *do* want to drop back to traditional tools, you can—Nu isn’t trying to delete your existing toolbox.

## Who it’s for

Nushell is a particularly good fit if you:

- Spend time in terminals but feel allergic to “parse text with regex until it cries”
- Work with **JSON/CSV/logs** and want faster ad-hoc analysis
- Like the idea of shell scripting, but want it to be **more readable**

It’s also a nice “gateway shell” for folks who are productive in the terminal but don’t want to memorize every `awk` dialect on Earth.

If you live and breathe POSIX shell scripts that must run everywhere (old servers, minimal containers, etc.), you’ll still want Bash/sh in your back pocket. Nu can coexist—think of it as an extra power tool.

## Getting started (smallest possible first step)

Install Nushell, then run `nu`.

### macOS (Homebrew)

```bash
brew install nushell
nu
```

### Windows (Winget)

```powershell
winget install Nushell.Nushell
nu
```

### Linux

The easiest route varies by distro; the docs list current options. Once installed:

```bash
nu
```

Try this first inside Nu:

```nu
ls | where name =~ "\.md$" | select name size
```

That’s it—you’ve already done something that would normally be a small pile of quoting and column parsing.

## Links

- Official docs (The Nu Book): https://www.nushell.sh/book/
- GitHub repo: https://github.com/nushell/nushell
- Docs page on structured data & pipelines: https://www.nushell.sh/book/pipelines.html
