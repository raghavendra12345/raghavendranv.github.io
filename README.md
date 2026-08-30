# Cryptography blog — Hugo starter

A Hugo site set up for math-heavy technical writing: KaTeX equations that
actually render, a Posts section for dated essays, and a Topics section of
standing reference pages that pull in related posts automatically.

Built and verified against **Hugo 0.152.0 extended** and **PaperMod**.

---

## 1. Setup

```bash
# From inside this folder
git init
git branch -M main

# PaperMod is not included in this download — add it as a submodule.
# The deploy workflow expects it at exactly this path.
git submodule add --depth=1 \
  https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod

# Preview locally at http://localhost:1313
hugo server -D
```

If you don't have Hugo yet, you need the **extended** build — PaperMod compiles
SCSS and the standard build will fail. On macOS `brew install hugo`; on
Windows `winget install Hugo.Hugo.Extended`; on Linux grab the
`hugo_extended_*_linux-amd64.deb` from the
[releases page](https://github.com/gohugoio/hugo/releases).

## 2. Replace the placeholders

Search the project for `USERNAME` and `Your Name` and replace them:

| File | What to change |
|---|---|
| `hugo.toml` | `baseURL`, `title`, `author`, `socialIcons` URL |
| `content/about.md` | everything — it's a stub |

**baseURL depends on which kind of Pages site you make:**

- User site — repo named `USERNAME.github.io` → `https://USERNAME.github.io/`
- Project site — repo named anything else → `https://USERNAME.github.io/reponame/`

The workflow overrides this at build time anyway, but `hugo server` uses it.

## 3. Deploy

1. Push to a GitHub repo on the `main` branch.
2. Repo → **Settings → Pages → Build and deployment → Source: GitHub Actions**.
   This step is easy to miss and nothing deploys without it.
3. Every push to `main` now rebuilds and publishes.

To link this from your Google Sites landing page, add a nav link pointing at the
Pages URL. If you'd rather use a custom domain, put a `CNAME` file containing
your domain in `static/` and configure the DNS records under Settings → Pages.

---

## 4. Writing a post

Create `content/posts/your-slug.md`:

```yaml
---
title: "Your Title"
date: 2026-09-01
math: true            # REQUIRED — without this KaTeX never loads
tags: ["lattices", "provable-security"]
description: "One or two sentences. Shows in listings and link previews."
ShowToc: true
draft: false
---
```

`math: true` is the one that catches people. Set `math = true` under `[params]`
in `hugo.toml` if you'd rather load KaTeX on every page.

### Math syntax

| Form | Syntax |
|---|---|
| Inline | `$g^{ab} \bmod p$` or `\(...\)` |
| Display | `$$ ... $$` on its own lines, or `\[...\]` |
| Numbered | `$$ E = mc^2 \tag{1} $$` |

Predefined macros (extend the list in `layouts/_partials/extend_head.html`):

`\Z` `\F` `\E` `\Adv` `\Enc` `\Dec` `\Gen` `\negl` `\given`

So you write `\Z_q` rather than `\mathbb{Z}_q`, and `\negl(\lambda)` rather
than `\mathsf{negl}(\lambda)`.

### Theorem and definition blocks

```markdown
{{< box "Definition 1" >}}
A group $(G, \cdot)$ is **cyclic** if there is some $g \in G$
with $G = \{ g^k : k \in \Z \}$.
{{< /box >}}
```

### Two traps with multi-line math

**Lines beginning with `-` become bullet points.** Markdown grabs them before
the math passthrough does. Move the minus to the end of the previous line, or
prefix with `&`:

```latex
$$
\begin{aligned}
X = \Bigl| \; &\Pr[\dots] \\
              &\;-\; \Pr[\dots] \Bigr|
\end{aligned}
$$
```

**Long equations scroll rather than wrap.** They won't break your layout, but on
a phone the reader has to swipe. For anything longer than about 60 characters,
break it yourself with `aligned`.

---

## 5. Adding a topic page

Create `content/topics/your-topic.md`:

```yaml
---
title: "Zero-Knowledge Proofs"
weight: 40          # controls card order on /topics/
math: true
tag: "zero-knowledge"   # posts with this tag auto-list on the page
description: "One line. Appears on the topic card."
ShowToc: true
---
```

The `tag` field is the link: any post tagged `zero-knowledge` appears in a
"Posts on this topic" list at the bottom of the page. Tag the post once and it
shows up — nothing to maintain by hand.

---

## 6. How the pieces fit

```
hugo.toml                              config; the passthrough block is load-bearing
content/
  posts/          *.md                 dated essays
  topics/         *.md                 standing reference pages
  about.md, search.md
layouts/
  _partials/extend_head.html           fonts + KaTeX + macros
  _shortcodes/box.html                 {{< box >}} theorem blocks
  topics/list.html                     the topic card grid
  topics/single.html                   topic page + auto post list
assets/css/extended/custom.css         the whole design layer
.github/workflows/deploy.yml           build and publish
themes/PaperMod/                       submodule — you add this
```

PaperMod auto-loads every `.css` file in `assets/css/extended/`. No import
needed; add more files there and they're picked up.

---

## 7. Design notes

The palette is oxidised brass. Verdigris (`#1c6f63` / `#4fb8a5` dark) carries
links and structural rules. Raw brass (`#9a6f24` / `#d3a757` dark) is used for
exactly one thing — the hairline beside display equations — so an equation is
always the most prominent object on the page. If you add accents, don't spend
brass on them or that stops being true.

Newsreader for prose, chosen because it sits comfortably beside KaTeX's
Computer Modern instead of clashing with it. Martian Mono for headings, used
at h1–h3 only. IBM Plex Mono for code, metadata, and tables.

All three load from Google Fonts, and KaTeX from jsDelivr. To self-host instead
(faster, no third-party requests), install `katex` and the `@fontsource/*`
packages from npm, drop them in `static/`, and repoint the URLs in
`extend_head.html`.

---

## 8. Things worth knowing

- **`layouts/_partials/`, not `layouts/partials/`.** Hugo 0.146+ renamed this.
  Older tutorials give the wrong path and it fails silently.
- **No `topics` taxonomy.** It would collide with the `/topics/` section URL.
  Tags do the grouping instead.
- **Drafts** (`draft: true`) are visible in `hugo server -D` but never
  published. Handy for parking half-finished proofs.
- **Search** is client-side Fuse.js over an index Hugo generates. It works
  offline and needs no service.
- **Subresource Integrity** is not set on the CDN tags. If you want it, pin an
  exact KaTeX version and copy the hashes from https://katex.org/docs/browser.

## 9. Next steps that pay off

- Add `static/favicon.ico` and set `params.assets.favicon`.
- Write topic pages before posts. They're the part people bookmark.
- Cross-link aggressively: a post that links three topic pages, and three topic
  pages linking back, is what makes a reference site feel navigable rather than
  like a pile of essays.
