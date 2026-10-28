# CLAUDE.md

Personal portfolio website of David Tabernero Mateo (game developer / technical designer), served at **https://tabernero.dev/**.

## Git workflow rules (mandatory)

### Cloud sessions (remote containers, e.g. claude.ai/code)
- Work **only on the `claude` branch** unless explicitly told otherwise.
- **At the start of every cloud session**, merge the development branch into `claude` to stay up to date:
  1. `git fetch origin`
  2. If `origin/development` exists → `git merge origin/development`
  3. Otherwise → `git merge origin/master`
- Within the `claude` branch, committing and pushing (`git push -u origin claude`) is allowed.
- **Never** create, check out, merge into, push to, delete, or otherwise manage any other branch, and never open/manage pull requests, unless explicitly requested.

### Local sessions (the user's own machine)
- **NEVER touch git** unless explicitly asked: no merge, commit, stage/add, checkout, stash, reset, push, pull, branch changes — nothing. Only edit files on disk; the user handles version control.

## Stack

- **Hugo** static site, no theme — all templates are custom in `layouts/`.
- Plain CSS in `static/css/` (no preprocessor, no PostCSS, no bundler).
- Plain JS in `static/js/`; only third-party lib is `vanilla-tilt.min.js` (tilt effect, currently mostly commented out in templates).
- Fonts from Google Fonts: Archivo, Inter (default), JetBrains Mono.
- Build: `hugo` (output to `public/`, git-ignored). Local preview: `hugo server`. Hugo is not preinstalled in cloud containers.
- CI: `.gitlab-ci.yml` (GitLab Pages, `hugo_extended` image) — `test` job builds on non-default branches, `pages` job deploys on the default branch. `.github/workflows/pages.yml` does the same for GitHub Pages (remote `github` → `davidtabernerom.github.io`).
- `.gitattributes` is a Unity-oriented template (the user works mainly in C#/Unity); it's irrelevant to the site but leave it alone.

## Structure

```
hugo.toml                 BaseURL, Title, params.Author; taxonomies disabled
archetypes/default.md     TOML front matter template
content/
  _index.md               Home       (menu weight 1)
  about/_index.md         About Me   (menu weight 2)
  projects/_index.md      Projects   (menu weight 6)
  projects/*.md           One file per game project (YAML front matter, no body)
  contact/_index.md       Contact    (menu weight 9)
data/
  qualifications.yaml     Education/certifications list (rendered on About)
  socials.yaml            Social links (rendered on Contact, icons via CSS mask)
  stores.yaml             Store key -> {name, image}   (used by project Links)
  studios.yaml            Studio key -> {name, image, link} (developers/publishers)
  datatags/tilt.yaml      data-* attrs for vanilla-tilt, emitted via partials/datatags.html
layouts/
  _default/baseof.html    Shell: head, nav, main > header + "article" block + footer
  index.html              Home: presentation, next in-development project, 3 latest releases
  about/list.html         Selfie, bio text, qualifications
  projects/list.html      Upcoming (InDevelopment) and Released lists
  projects/single.html    Project sheet: details, studios, position, store links, cover
  contact/list.html       Social network links
  partials/               head, nav (logo, menu, CV links), header (breadcrumbs), footer, datatags
static/
  css/                    One file per area: _default (vars/reset), body, nav, main, header,
                          footer, article, article.<page>, mobile — all linked in partials/head.html
  img/                    logo, selfie, placeholders (no_cover/no_banner), qualifications/,
                          socials/, stores/, studios/
  files/                  curriculum_vitae_english.pdf, curriculum_vitae_spanish.pdf
  js/                     main.js (empty), lib/vanilla-tilt.min.js
```

## Project content schema (`content/projects/*.md`)

YAML front matter, PascalCase keys:

```yaml
Title: "Game Title"
Description: "Short description"
Date: 2021-09-24            # start date of my involvement
Draft: false
ProjectTitle: "Codename"     # internal/working title
ShortenedTitle: null         # optional short name for breadcrumbs/lists
Released: true               # -> "Released Projects" / home "Latest Releases"
InDevelopment: false         # -> "Upcoming Projects" / home "Next Project"
CoverLink: "..."             # portrait cover (Steam library_600x900 or /img/no_cover.png)
BannerLink: "..."            # wide hero (Steam library_hero or /img/no_banner.png)
ReleaseDate: 2023-11-02      # used for sorting
MainDeveloper: "Praenaris"   # key in data/studios.yaml
Developers: ["Praenaris"]    # keys in data/studios.yaml
Publishers: ["GammeraNest"]  # keys in data/studios.yaml
JobPosition: "Lead Programmer & Technical Manager"
Links:
  - store: "Steam"           # key in data/stores.yaml
    link: "https://..."
```

Adding a studio or store means adding an entry in the matching data file plus its image under `static/img/studios/` or `static/img/stores/`.

## Conventions

- Tabs for indentation in HTML/CSS/templates.
- CSS custom properties defined in `static/css/_default.css` with `--area--property` naming (e.g. `--general--accent-color`, `--content--block-padding`). Reuse them instead of hardcoding values.
- HTML ids/classes are PascalCase (`#Home`, `.Game`, `.Released`, `.Highlighted`).
- New CSS files must be linked manually in `layouts/partials/head.html`.
- Responsive rules live only in `static/css/mobile.css` (linked last): `max-width: 1100px` = tablet (sidebar kept, tighter margins), `max-width: 800px` = mobile (sidebar becomes a sticky top bar, single column). When changing desktop styles, check whether the mobile overrides need updating too.
- Site copy is in English; qualification data is in Spanish (official names).
- Keep it dependency-free: no npm, no Hugo modules/themes unless asked.

## Known loose ends (observed, not yet fixed)

- `partials/head.html` references `/img/favicon-dark.png` and `/img/favicon-light.png`, which don't exist.
- `data/stores.yaml` lists `epicgames.svg` and `gog.svg`, which are missing from `static/img/stores/`.
- `partials/header.html` builds the project breadcrumb URL from `.Title` instead of `.RelPermalink`.
- `<title>` is always the site title; per-page titles are not used.
- `content/projects/unclouded.md` still has `ProjectTitle: "Windmill"` and an itch.io link with the old name (may be intentional).
