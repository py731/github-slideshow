# CLAUDE.md — AI Assistant Guide for github-slideshow

## Project Overview

**github-slideshow** is a GitHub Learning Lab educational project that teaches Git and GitHub fundamentals through an interactive web-based slideshow. It uses [Jekyll](https://jekyllrb.com/) as the static site generator and [reveal.js](https://revealjs.com/) as the presentation framework.

The GitHub Learning Lab bot communicates with learners via GitHub Issues and Pull Requests, guiding them through exercises using this repository as a sandbox.

---

## Repository Structure

```
github-slideshow/
├── _config.yml              # Jekyll + reveal.js configuration
├── _includes/
│   ├── head.html            # HTML <head> with reveal.js CSS links
│   ├── script.html          # Reveal.js initialization JS
│   └── slide.html           # Per-slide section template
├── _layouts/
│   ├── presentation.html    # Top-level reveal.js wrapper layout
│   ├── print.html           # Print/PDF export layout
│   └── slide.html           # Individual slide layout
├── _posts/
│   └── 0000-01-01-intro.md  # Sample/intro slide (add slides here)
├── index.html               # Presentation entry point
├── script/
│   ├── cibuild              # CI build + HTML validation
│   ├── server               # Local dev server
│   ├── setup                # Dev environment setup
│   └── stage                # Internal staging deployment
├── Gemfile                  # Ruby gem dependencies
├── Gemfile.lock             # Locked gem versions
├── package-lock.json        # npm lock file (reveal.js)
├── .editorconfig            # Editor formatting standards
└── .gitignore               # Excludes _site/, .sass-cache/, etc.
```

---

## Development Commands

### Environment Setup (first time)
```sh
script/setup
```
- Installs Ruby via rbenv (macOS) if needed
- Installs bundler and runs `bundle install`

### Local Development Server
```sh
script/server
# or with options:
bundle exec jekyll serve --livereload
```
Serves the site at `http://localhost:4000` with live reload.

### CI Build + HTML Validation
```sh
script/cibuild
```
- Builds the Jekyll site: `bundle exec jekyll build --baseurl "."`
- Validates HTML output: `htmlproofer _site/index.html --empty-alt-ignore`
- Output goes to `_site/` (gitignored)

### Staging Deployment (internal only)
```sh
script/stage
```
Builds with a custom baseurl and pushes compiled output to an internal GitHub Enterprise staging remote.

---

## Adding Slides

Slides are Jekyll posts in `_posts/`. Each file must follow Jekyll's post naming convention: `YYYY-MM-DD-title.md`.

**Example: `_posts/0000-01-02-my-slide.md`**
```markdown
---
layout: slide
title: "My Slide Title"
---

Slide body content goes here. Supports Markdown and HTML.
```

### Slide Front Matter Options
| Key | Purpose |
|-----|---------|
| `layout` | Must be `slide` |
| `title` | Rendered as `<h1>` at top of slide |
| `class` | Custom CSS class on the `<section>` element |
| Any `data-*` | Passed as reveal.js data attributes (e.g., `data-background`) |

Slides are rendered in **reverse post order** (newest date = first slide), so use dates to control ordering.

---

## Configuration

### Jekyll (`_config.yml`)
Key settings:
- `timezone: Europe/Berlin`
- `highlighter: rouge` / `markdown: kramdown`
- `plugins: [jemoji]` — enables emoji support via `:emoji_name:` syntax
- `future: false` — posts dated in the future are excluded

### Reveal.js (`_config.yml` under `reveal:`)
```yaml
reveal:
  controls: false
  progress: true
  history: true
  keyboard: true
  transition: linear
  backgroundTransition: slide
  width: 1000
  height: 920
```
These values are passed to `Reveal.initialize()` in `_includes/script.html`.

---

## Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Static site generator | Jekyll (via github-pages gem) | ~3.8.5 |
| Presentation framework | reveal.js | 3.9.2 |
| Syntax highlighting | Rouge (Jekyll) + Highlight.js (reveal.js) | — |
| HTML validation | html-proofer | ~3.14 |
| Markdown renderer | kramdown | — |
| Emoji support | jemoji | — |

---

## Code Conventions

### Editor Standards (`.editorconfig`)
- **Indentation**: Tabs (width 4) for most files
- **YAML, JSON, JS, CSS, SCSS, HTML**: 2-space indentation
- **Markdown**: 4-space indentation
- **Charset**: UTF-8
- **Line endings**: LF
- **Trailing whitespace**: trimmed

### File Conventions
- All slide content lives in `_posts/` as Markdown files
- Layout templates in `_layouts/` and partials in `_includes/` use Liquid templating
- Avoid committing `_site/`, `.sass-cache/`, `.jekyll-metadata`, or `.bundle`

---

## Testing

The only automated test is HTML validation via html-proofer, run in `script/cibuild`:
```sh
htmlproofer _site/index.html --empty-alt-ignore
```

There are no unit/integration tests. When making changes:
1. Run `script/server` and visually inspect the presentation
2. Run `script/cibuild` to confirm the build passes HTML validation
3. Ensure no broken links or malformed HTML is introduced

---

## Common Pitfalls

- **Slide ordering**: Posts render in reverse date order. Use the date portion of the filename (`YYYY-MM-DD`) to control slide sequence; lower dates appear later in the deck.
- **`future: false`**: Posts with future dates will not be included in the build. Use dates like `0000-01-01` for slides that should always appear.
- **baseurl**: The CI build sets `--baseurl "."` which differs from local serving. Asset paths in templates use `{{ site.baseurl }}` to stay portable.
- **`_site/` is gitignored**: Never commit the compiled output directory.
- **reveal.js is in `node_modules/`**: Referenced directly from `node_modules/reveal.js/` in HTML includes. Run `npm install` if the directory is missing (though it is typically committed).

---

## Git Workflow

- The primary branch is `master`
- Feature branches follow the pattern `claude/<description>-<session-id>`
- Commits should be clear and descriptive
- Push with: `git push -u origin <branch-name>`
