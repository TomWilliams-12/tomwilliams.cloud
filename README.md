# tomwilliams.cloud

Personal portfolio and blog — built with [Hugo](https://gohugo.io/) and deployed to GitHub Pages.

## Quick Start

### Prerequisites

- [Hugo Extended](https://gohugo.io/installation/) v0.139.0+
- Git

### Install Hugo (macOS/Linux)

```bash
# macOS
brew install hugo

# Linux (Debian/Ubuntu)
wget https://github.com/gohugoio/hugo/releases/download/v0.139.0/hugo_extended_0.139.0_linux-amd64.deb
sudo dpkg -i hugo_extended_0.139.0_linux-amd64.deb

# Windows (via Chocolatey)
choco install hugo-extended
```

### Run locally

```bash
git clone https://github.com/YOUR-USERNAME/tomwilliams.cloud.git
cd tomwilliams.cloud
hugo server -D
```

Open `http://localhost:1313` to preview. The `-D` flag includes draft posts.

## Deployment

This site deploys automatically via GitHub Actions on every push to `main`.

### First-time setup

1. **Create the GitHub repo**: `tomwilliams.cloud` (or any name you like)

2. **Push this project**:
   ```bash
   cd tomwilliams.cloud
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/YOUR-USERNAME/tomwilliams.cloud.git
   git push -u origin main
   ```

3. **Enable GitHub Pages**:
   - Go to repo → Settings → Pages
   - Source: **GitHub Actions**
   - That's it — the workflow in `.github/workflows/hugo.yml` handles the rest

4. **Add custom domain**:
   - In repo → Settings → Pages → Custom domain, enter: `tomwilliams.cloud`
   - Tick "Enforce HTTPS"
   - Add these DNS records at your domain registrar:

   | Type  | Name | Value                        |
   |-------|------|------------------------------|
   | A     | @    | 185.199.108.153              |
   | A     | @    | 185.199.109.153              |
   | A     | @    | 185.199.110.153              |
   | A     | @    | 185.199.111.153              |
   | CNAME | www  | YOUR-USERNAME.github.io      |

5. **Set up subdomain redirects** (optional, if using Cloudflare for DNS):
   - Add CNAME records or redirect rules for `linkedin.tomwilliams.cloud`, `github.tomwilliams.cloud`, etc.

## Configuration

Edit `hugo.toml` to update:

```toml
[params]
  github = "https://github.com/YOUR-USERNAME"
  linkedin = "https://linkedin.com/in/YOUR-PROFILE"
  email = "your@email.com"

[params.contact]
  formspree = "https://formspree.io/f/YOUR-FORM-ID"
```

### Contact form

The contact form uses [Formspree](https://formspree.io/) (free for 50 submissions/month):

1. Sign up at formspree.io
2. Create a new form
3. Copy the form endpoint URL into `hugo.toml`

## Writing Content

### New blog post

```bash
hugo new blog/my-post-title.md
```

This creates `content/blog/my-post-title.md` using the blog archetype. Edit the front matter:

```yaml
---
title: "My Post Title"
date: 2026-03-14
summary: "A one-liner that appears on the blog listing page."
tags: ["aws", "terraform"]
draft: false    # Set to false when ready to publish
---
```

### New project case study

Create a file in `content/projects/`:

```yaml
---
title: "Project Name"
date: 2026-01-01
tag: "Migration"          # Category badge shown on the card
tech: ["Terraform", "Lambda", "S3"]
summary: "One-liner for the project card."
---
```

### Certifications

Edit `data/certs.yaml` to add or update certifications.

## Project Structure

```
tomwilliams.cloud/
├── .github/workflows/hugo.yml   # GitHub Actions deploy pipeline
├── archetypes/                  # Templates for new content
├── content/
│   ├── blog/                    # Blog posts (Markdown)
│   └── projects/                # Project case studies (Markdown)
├── data/
│   └── certs.yaml               # Certification data
├── layouts/
│   ├── _default/                # Fallback templates
│   ├── blog/                    # Blog list + single templates
│   ├── page/                    # Homepage template
│   ├── partials/                # Nav, footer
│   └── projects/                # Project list + single templates
├── static/
│   ├── css/style.css            # All styling
│   └── images/                  # Static images
└── hugo.toml                    # Site configuration
```

## Customisation

- **Theme colours**: Edit CSS variables in `static/css/style.css` under `:root`
- **Fonts**: Change the Google Fonts import and `--font-sans` / `--font-mono` variables
- **Layout**: Templates are in `layouts/` — Hugo uses Go templating
