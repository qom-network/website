---
title: "How to Contribute"
description: "Step-by-step guide for contributing to the QOM Movement website."
slug: "how-to-contribute"
toc: true
---
{{< typeit
  tag=h2
  speed=80
  lifeLike=true
 >}}
How to Contribute
{{< /typeit >}}
{{< lead >}}
Welcome! This guide explains how to edit and improve the QOM Movement website step-by-step.  
We assume you already have **Hugo (extended)** installed and can run a site with the **Blowfish** theme. For detail, see [Blowfish Official Documentation](https://blowfish.page/docs/)The working source for this site lives in the **`main`** branch of the repository.
{{< /lead >}}
---

## Fast Path

1. **Fork** the repo to your GitHub account.  
2. **Create a feature branch** from `dev`.  
3. **Run locally**: `hugo server -D`.  
4. **Edit content** under `content/…` (and put images in `assets/img/…`).  
5. **Commit & push**, then **open a Pull Request (PR)** back to `dev`.  
6. A maintainer will review, request changes if needed, and merge.

---

## 1) Repository Setup

### A. Fork and clone
```bash
# Replace <your-username> with your GitHub handle
# Clone the Repository(HTTPS)
git clone https://github.com/<your-username>/website.git
cd qom-website
# Clone the Repository(SSH)
git clone git@github.com:<your-username>/website.git
cd qom-website

# Add the upstream (original) repository for syncing
git remote add upstream https://github.com/qom-network/website.git

# Make sure you're on the dev branch
git fetch --all
git checkout dev
git pull upstream dev

# Create a working branch
# branch naming: feature/<topic>  or  docs/<topic>  or  fix/<topic>
git checkout -b docs/new-document
```

## 2) Run the Site Locally

```bash
# Development server (drafts enabled, better live reload)
hugo server -D --disableFastRender
```

Open the URL shown in your terminal (usually http://localhost:1313/).


## 3) Where to Edit (Content Layout)

- **Pages (Markdown):** `content/`
    
    - Home: `content/_index.md`
        
    - About: `content/about/_index.md`
        
    - Get Involved: `content/get-involved/_index.md`
        
    - Docs: `content/docs/_index.md`
        
    - This page: `content/get-involved/how-to-contribute.md`
        
- **Images** `assets/`
    
    - Put images in `assets/img/...`
        
    - Refer to them as `/img/<file>` in Markdown.
        
- **Navigation (if needed):** `config/_default/menus.toml`
    
- **Site params / theme tweaks:** `config/_default/params.toml`
    

> Blowfish automatically serves files from `assets/` at the site root.  
> Example image path in Markdown:
```md
![Alt text for accessibility](/img/example.png)
```

## 4) Writing & Style Guidelines

**Content style**

- Keep paragraphs short and readable.
    
- Prefer plain English; define acronyms on first use.
    
- Use **sentence-case** headings (capitalize the first word only).
    

**Headings**

- One `#` per page for the title (Front Matter `title` will become H1 in some layouts).
    
- Then `##` (H2) and `###` (H3) for sections.
    

**Front Matter (example)**
```yaml
---
title: "My New Page"
description: "Short, searchable summary (≤ 160 chars)."
draft: false
---
```

**File naming**

- Use kebab-case for filenames and directories, (e.g., `run-a-node.md.`)

**Internal links**

- Use absolute paths from the site root:

```md
See the [About page](/about/) and the [Docs](/docs/).
```


**Images**

-　Use descriptive alt text.

-　Optimize large images (prefer ≤ 200 KB where possible).

-　PNG or WebP recommended for UI assets; JPEG or WebP for photos.

## 5) Add or Edit Content
### A. Create a new page
```bash
# Example: create a new doc page
mkdir -p content/docs/run-a-node
$EDITOR content/docs/run-a-node/_index.md

```

**Starter template:**
```markdown
---
title: "Run a Node"
description: "Set up a QOM/QL1 node and join the network."
draft: false
---

## Overview
Brief summary…

## Prerequisites
- Docker …
- Disk space …

## Steps
1. …
2. …
```

### B. Update navigation (optional)

If your new page should appear in the top menu or a section menu:

```toml
# config/_default/menus.toml
[[main]]
  name = "Docs"
  pageRef = "/docs/"
  weight = 4
```

> Many lists/sidebars are generated automatically by Blowfish; explicit menu entries are optional.

## 6) Internationalization (i18n)
We keep English as the source of truth and add translations as separate content trees:

```bash
content/
  en/...
  ja/...
  es/...
```


**Example:**

- Source page: content/en/get-involved/how-to-contribute.md

- Japanese translation: content/ja/get-involved/how-to-contribute.md

Front Matter should contain the same `slug` so Hugo links the translations.
Keep structure and headings aligned between languages.

## 7) Commit, Push, and Open a PR

```bash
# Stage and commit your changes
git add .
git commit -m "docs: add how-to-contribute guide (Get Involved)"

# Push to your fork
git push origin docs/how-to-contribute
```

Now go to your fork on GitHub and click “Compare & pull request”.
Target branch: `dev` (not `main`).

**PR checklist**

- Clear title and description of changes.

- Screenshots for visual updates.

- Mention related issues (e.g., “Closes #123”).

- Keep the PR focused and small if possible.

**Reviews**

- Expect feedback; maintainers may request minor edits.

- After approval, a maintainer will merge your PR into `dev`.

## 8) Common Tasks & Shortcodes (Blowfish)


Exact shortcode names and availability depend on the current Blowfish configuration in this repository. If a shortcode errors, check layouts/shortcodes/ or theme docs and remove/adjust accordingly.


## 9) Good First Issues

Not sure where to start? Look for:

- Typos, clarity edits, broken links.

- Add alt text to images.

- Translate existing pages (EN → JA/ES).

- Small docs like “Run a Node” stubs, glossary entries, FAQs.


## 10) Next Steps

- Propose a new doc page under content/docs/

- Improve the About or Get Involved pages

- Add translations to reach more contributors

{{< lead >}}
Thank you for contributing. Every paragraph, screenshot, and code snippet helps build a sovereign, community-owned internet—together.
{{< /lead >}}