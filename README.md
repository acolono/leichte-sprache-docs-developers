# Leichte Sprache Documentation for developers

This directory contains the complete MkDocs documentation for developers.

## Setup

### Prerequisites

- Python 3.11+
- pip or uv package manager

### Installation

```bash
# Install MkDocs and required plugins
pip install -r requirements.txt

# Alternative with uv
uv add mkdocs mkdocs-material mkdocs-awesome-pages-plugin mkdocs-git-revision-date-localized-plugin mkdocs-minify-plugin
```

## Development

### Local Development Server

```bash
# Start development server with auto-reload
mkdocs serve

# Open in browser
open http://127.0.0.1:8080
```

### Build Static Site

```bash
# Build static HTML files
mkdocs build

# Output will be in site/ directory
ls site/
```

## Deployment Options

### GitHub Pages

```bash
# Deploy to GitHub Pages
mkdocs gh-deploy
```

### Netlify

Build settings:
- Build command: `mkdocs build`
- Publish directory: `site`

### Self-Hosted

```bash
# Build and copy to web server
mkdocs build
rsync -avz site/ user@server:/var/www/docs/
```

## Directory Structure

```
docs/mkdocs/
├── mkdocs.yml              # MkDocs configuration
├── requirements.txt        # Python dependencies
├── README.md              # This file
│
├── docs/                  # Documentation source
│   ├── index.md           # Homepage
│   ├── getting-started/   # Setup guides
│   ├── user-guide/        # User documentation
│   ├── api/              # API reference
│   ├── architecture/      # System design
│   ├── development/       # Contributing guides
│   ├── deployment/        # Deployment guides
│   ├── reference/         # Reference materials
│   └── assets/           # Images and static files
│
└── site/                  # Generated static files (after build)
```

## Customization

To customize for your deployment:

1. **Update repository URLs** in `mkdocs.yml`
2. **Configure site URL** for your domain
3. **Add analytics** if needed
4. **Customize theme colors** and branding

## Usage

This documentation can be:

1. **Copied to a separate repository** for documentation hosting
2. **Deployed directly** from this location
3. **Integrated** into CI/CD pipelines
4. **Served locally** during development

---

Ready to deploy? Install the requirements and run `mkdocs serve` to get started!