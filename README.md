# Barakah Tech Journey 🌙

A personal blog documenting the journey of building a meaningful career in tech while staying true to Islamic values and serving the Muslim community.

## About

**Barakah** (بركة) means "blessing" or "divine abundance" in Arabic. This blog explores what it means to pursue barakah in a tech career—choosing purpose over prestige, values over compromise, and excellence (ihsan) in all we do.

I'm Malik Wensman, a Computer Science senior at NC State University and Full-Stack Software Engineer at Axiom Software. This blog chronicles my experiences building technology that serves the Muslim community, along with technical learnings, career reflections, and lessons on maintaining Islamic principles in the modern tech industry.

## What You'll Find Here

- **Technical Posts**: Full-stack development, API integrations, AI agents, and real-world problem-solving
- **Career Insights**: Navigating the tech industry with intentionality and Islamic values
- **Community Impact**: How technology can uplift and serve our communities
- **Personal Growth**: Balancing ambition with contentment, progress with principle

## Tech Stack

This blog is built with:
- **[Hugo](https://gohugo.io/)** - Fast and flexible static site generator
- **[Hextra Theme](https://imfing.github.io/hextra/)** - Modern, responsive documentation theme
- **GitHub Pages** - Free hosting at [mdw223.github.io/barakah-tech-journey](https://mdw223.github.io/barakah-tech-journey/)
- **Markdown** - Simple content creation

## Getting Started

### Prerequisites

You'll need to install the following tools:

1. **Hugo (Extended Version)**
   - Download from [Hugo Installation Guide](https://gohugo.io/installation/)
   - The extended version is required for Hextra theme

2. **Chocolatey** (Windows users)
   - Installation instructions at [chocolatey.org](https://chocolatey.org/)
   - Makes installing Hugo easier on Windows
   - Mac/Linux users can use Homebrew or package managers

### Installation

1. Clone the repository:
```bash
git clone https://github.com/mdw223/barakah-tech-journey.git
cd barakah-tech-journey
```

2. Install Hugo (if not already installed):

**Windows (with Chocolatey):**
```bash
choco install hugo-extended
```

**macOS (with Homebrew):**
```bash
brew install hugo
```

**Linux:**
```bash
# Check Hugo installation guide for your distribution
# https://gohugo.io/installation/
```

3. Install Hextra theme as a Hugo module:
```bash
hugo mod init github.com/mdw223/barakah-tech-journey
hugo mod get github.com/imfing/hextra
```

The theme is configured as a module import in `hugo.yaml`.

### Running Locally

Start the Hugo development server:
```bash
hugo server --buildDrafts
```

The site will be available at `http://localhost:1313/`

Hugo will automatically rebuild when you make changes to content or configuration.

#### Troubleshooting

Remove the old build, rebuild, then view in local development
```bash
rm -rf public/
hugo
hugo server --buildDrafts
```

## Project Structure

```
barakah-tech-journey/
├── .github/
│   └── workflows/
│       └── hugo.yaml      # GitHub Actions deployment workflow
├── content/              # Blog posts and pages (Markdown files)
│   ├── blog/            # Blog post directory
│   ├── docs/            # Documentation pages
│   ├── about/           # About page
│   └── _index.md        # Homepage content
├── static/              # Static assets (images, CSS, JS)
│   └── images/          # Logo and image files
├── go.mod               # Hugo modules configuration
├── hugo.yaml            # Hugo configuration (YAML format)
└── README.md            # This file
```

## Creating Content

### New Blog Post

Create a new post:
```bash
hugo new content/blog/my-new-post.md
```

This creates a file with front matter:
```yaml
---
title: "My New Post"
date: 2025-10-08
draft: true
tags: ["tag1", "tag2"]
---

Your content here...
```

### Writing Tips

- Write in Markdown
- Set `draft: false` when ready to publish
- Use tags to categorize posts
- Add featured images in the post's front matter
- Reference the [Hextra documentation](https://imfing.github.io/hextra/docs/getting-started/) for advanced features

## Deployment

### Build for Production

Generate static site files:
```bash
hugo --minify
```

The output will be in the `public/` directory.

### GitHub Actions Deployment

The site automatically deploys to GitHub Pages using a GitHub Actions workflow (`.github/workflows/hugo.yaml`):

**Key Features:**
- ✅ Automatically builds and deploys on every push to `main`
- ✅ Uses Hugo Extended v0.151.0
- ✅ Go 1.22 for Hugo modules
- ✅ Minified production builds
- ✅ Manual deployment trigger via `workflow_dispatch`

**Workflow Overview:**
```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.151.0
    steps:
      - Checkout repository with submodules
      - Setup Go 1.22
      - Install Hugo Extended
      - Build with hugo --gc --minify
      - Upload artifact to Pages
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - Deploy artifact to GitHub Pages
```

**To Deploy:**
1. Push changes to the `main` branch
2. GitHub Actions automatically builds and deploys
3. View live site at: [mdw223.github.io/barakah-tech-journey](https://mdw223.github.io/barakah-tech-journey/)

**Manual Deployment:**
- Go to Actions tab in GitHub
- Select "Deploy Hugo site to Pages"
- Click "Run workflow"

### Alternative Deployment Options

- **Netlify**: Connect your GitHub repo for alternative hosting
- **Vercel**: Similar to Netlify with great Hugo support
- **Cloudflare Pages**: Fast global CDN deployment

## Configuration

The site is configured using `hugo.yaml` (YAML format). Key settings:

```yaml
baseURL: https://mdw223.github.io/barakah-tech-journey/
languageCode: en-us
title: Barakah Tech Journey

module:
  imports:
    - path: github.com/imfing/hextra

menu:
  main:
    - name: Documentation
      pageRef: /docs
      weight: 1
    - name: Blog
      pageRef: /blog
      weight: 2
    - name: About
      pageRef: /about
      weight: 3
    - name: Search
      weight: 4
      params:
        type: search
    - name: GitHub
      weight: 5
      url: "https://github.com/mdw223"
      params:
        icon: github

params:
  navbar:
    displayTitle: true
    displayLogo: true
    logo:
      path: images/logo.svg
      dark: images/logo-dark.svg
      link: /barakah-tech-journey/
      width: 40
      height: 20
  
  banner:
    key: 'announcement-xxx'
    message: |
      🎉 Welcome to my blog!
```

### Customization Options

- **Menu Items**: Add or remove navigation links in the `menu.main` section
- **Logo**: Replace `static/images/logo.svg` and `logo-dark.svg` with your own
- **Banner**: Update the welcome message or change the key to show/hide
- **Search**: Built-in search functionality via Hextra theme
- **GitHub Integration**: Direct link to your repository in the nav bar

## Resources

- **Hugo Documentation**: [gohugo.io/documentation/](https://gohugo.io/documentation/)
- **Hextra Theme Docs**: [imfing.github.io/hextra/docs/](https://imfing.github.io/hextra/docs/getting-started/)
- **Markdown Guide**: [markdownguide.org](https://www.markdownguide.org/)

## Contributing

This is a personal blog, but I welcome:
- Bug reports and fixes
- Typo corrections
- Suggestions for topics to cover
- Questions and discussions in Issues

## Core Values

This blog is built on three principles:
1. **Positively Impact Your Community** - Technology should serve something greater
2. **Never Compromise Your Values** - Draw clear lines and have courage
3. **Strive for Excellence (Ihsan)** - Do beautiful work as an act of worship

## Connect

- **LinkedIn**: [linkedin.com/in/malik-wensman/](https://linkedin.com/in/malik-wensman/)
- **GitHub**: [github.com/mdw223](https://github.com/mdw223)
- **Email**: [Contact through website]

## License

Content is © Malik Wensman. Code (Hugo configuration and templates) is open source.

---

*May our work be filled with barakah, and may we find success in both this life and the next.*