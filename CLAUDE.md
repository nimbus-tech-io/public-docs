# public-docs

A small Jekyll docs site for the Nimbus Tech team. Live at https://nimbus-tech-io.github.io/public-docs/.

GitHub Pages builds it from `main` in legacy mode: push to `main` and it deploys. There is no CI, no Gemfile and no plugins, so use only what GitHub Pages supports out of the box.

## Layout

- `*.md` at the repo root: one file per doc. `foo.md` is served at `/public-docs/foo.html`.
- `index.md`: the home page. It lists every page with `layout: doc` and has a tag filter. Don't edit it to add a doc.
- `_layouts/doc.html`: the page frame (title, subtitle, byline, tags, scripts).
- `assets/css/doc.css`: all styles.
- `_includes/badge.html`: model badges.
- `assets/images/<doc-name>/`: images for a doc.

## Writing a new doc

1. Create `<slug>.md` at the root. Use a short, descriptive kebab-case slug.
2. Start it with this front matter:

   ```yaml
   ---
   layout: doc
   title: The Page Title
   subtitle: One sentence saying what the reader gets.
   updated: YYYY-MM-DD   # today; change it on every edit
   author: Name
   tags: [topic]         # lowercase; reuse existing tags where they fit
   mermaid: true         # only if the page has a Mermaid chart
   ---
   ```

3. Don't write an `# H1`. The layout prints the title. Start sections with `##` and use `###` for sub-sections.
4. Separate the main sections with `---`.

The home page and the tag filter pick up the new page by themselves.

## Markdown building blocks

The parser is kramdown (GFM). Use plain Markdown everywhere you can. These extras give the house style:

**Callout:** use GitHub alert syntax. The line after the marker is the title. `[!TIP]` is green, `[!NOTE]` is blue and `[!WARNING]` is amber. The layout turns these into callout boxes at build time.

```markdown
> [!TIP]
> Title of the box
>
> Body text.
```

**Numbered steps:** every numbered list is styled as steps by itself. Just write `1.`, `2.`, `3.`. Don't use numbered lists for anything else.

Don't use kramdown `{: .class}` lines. Prettier moves them and breaks the styling.

**Model badges:** `{% include badge.html m="opus" %}`, and the same for `sonnet` and `haiku`.

**Tables:** plain Markdown tables. The layout makes them scroll sideways on phones.

**Images:** `![Alt text](assets/images/<doc-name>/file.png)`. Use a relative path with no leading slash.

**Charts:** use a fenced `mermaid` block and set `mermaid: true` in the front matter. `using-claude.md` has a flowchart with the house colors (`classDef`) to copy.

**Links:** external links open in a new tab by themselves. Link to other docs by file name, like `[text](choosing-a-model.html)`.

## Rules

- Never link to claude.ai artifacts. They are private. If a doc needs one, bring its content into the repo as a page and link to that page.
- When you rename a page, its old URL stops working. Check for links to it with `rg`.
- Do not commit `.claude/launch.json`. It is local preview config and is in `.gitignore`.

## Previewing

Run the site locally with Jekyll 4 (Ruby from Apple Silicon Homebrew):

```bash
bundle install
bundle exec jekyll serve --baseurl ""
```

Then open http://localhost:4000/. The `Gemfile` is for local preview only. GitHub Pages builds the live site with its own Jekyll 3.x, and the `github-pages` gem does not run on Ruby 3.2+. Our pages use only features that both versions support. Keep it that way: no Jekyll 4-only features.
