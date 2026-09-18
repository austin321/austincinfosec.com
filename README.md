# austincinfosec.com

Personal site and blog for Austin Chen, built with [Hugo](https://gohugo.io) and
deployed to GitHub Pages on every push to `main`.

Migrated from WordPress.com — all five original posts, their images, and their
original permalinks are preserved.

## Local development

```bash
hugo server
```

Then open <http://localhost:1313>. The server live-reloads on save.

To produce a production build into `public/`:

```bash
hugo --gc --minify
```

## Writing a new post

```bash
hugo new content blog/my-post-slug/index.md
```

That creates a **page bundle** — a folder holding the post and its images:

```
content/blog/my-post-slug/
├── index.md          the post
├── cover.png         referenced as `cover: "cover.png"` in front matter
└── screenshot1.png   referenced in the body as ![alt text](screenshot1.png)
```

Drop images straight into the folder and reference them by filename. Hugo resizes
them, converts them to WebP, and lazy-loads them automatically.

Front matter fields:

| Field        | Purpose                                                |
| ------------ | ------------------------------------------------------ |
| `title`      | Post title                                             |
| `date`       | Publication date (drives ordering)                     |
| `summary`    | Card blurb; falls back to the first ~30 words          |
| `cover`      | Filename of the cover image inside the bundle          |
| `categories` | Broad grouping, e.g. `["Lab", "Education"]`            |
| `tags`       | Finer topics, e.g. `["python", "azure"]`               |
| `draft`      | `true` hides the post from builds                      |
| `aliases`    | Old URLs that should redirect here                     |

## Editing the other pages

| Page           | File                        | Notes                                                    |
| -------------- | --------------------------- | -------------------------------------------------------- |
| Home           | `content/_index.md`         | Intro paragraph under the hero                           |
| About          | `content/about.md`          | Plain Markdown                                           |
| Projects       | `content/projects.md`       | Cards come from the `projects:` list in the front matter |
| Certifications | `content/certifications.md` | Cards come from the `certifications:` list               |
| Contact        | `content/contact.md`        | Links come from `[[params.social]]` in `hugo.toml`       |

Site-wide settings — title, tagline, nav menu, social links — live in `hugo.toml`.

## Deployment

Pushing to `main` triggers `.github/workflows/deploy.yml`, which builds the site
and publishes it to GitHub Pages. The custom domain is pinned by `static/CNAME`.

## Theme

There is no external theme — the layouts in `layouts/` and the styles in
`assets/css/` are the theme. Colours, fonts, and spacing are CSS custom
properties defined at the top of `assets/css/main.css`; the dark and light
palettes are the two token blocks directly beneath them.
