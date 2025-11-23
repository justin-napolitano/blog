# blog

Personal website and blog of Justin Napolitano built using the Hugo static site generator with custom themes and content.

---

## Features

- Static site generated with Hugo using multiple themes (`hugo-coder`, `hugo-shortcodes`)
- Supports rich blog content with markdown, shortcodes, and emoji
- Pagination support for blog posts
- Syntax highlighting with Pygments
- Dark and light color schemes with toggle support
- Social media and contact integration (GitHub, Twitter, LinkedIn, Calendly, CV)
- Custom archetypes and layouts for posts
- Automated build and deployment pipeline via Python script and Makefile

## Tech Stack

- [Hugo](https://gohugo.io/) - Static site generator
- HTML, CSS, SCSS for styling
- Python for build automation
- JavaScript for theme toggling and UI enhancements

## Getting Started

### Prerequisites

- [Hugo](https://gohugo.io/getting-started/install/) installed
- Python 3.x installed
- `make` utility available

### Installation & Running Locally

1. Clone the repository

```bash
git clone https://github.com/justin-napolitano/blog.git
cd blog
```

2. Install Python dependencies (if any) via requirements.txt

```bash
pip install -r requirements.txt
```

3. Build the site using the Python build script or Makefile

```bash
python python-build.py
# or
make html
```

4. Serve the site locally

```bash
hugo server -D
```

Visit `http://localhost:1313` in your browser to view the site.

## Project Structure

```
blog/
├── archetypes/           # Hugo archetypes for content templates
├── config.toml           # Site configuration file
├── content/              # Markdown content for posts, pages
│   ├── posts/            # Blog posts
│   ├── about/            # About page
│   ├── projects/         # Projects overview
│   └── contact/          # Contact page
├── layouts/              # Hugo layouts and templates
├── public/               # Generated static site output
├── python-build.py       # Python script to automate build and deployment
├── resources/            # Hugo resource files (CSS, JS, images)
├── static/               # Static assets like images, icons
└── themes/               # Hugo themes used
```

## Future Work / Roadmap

- Improve documentation and add more detailed usage instructions
- Automate deployment to GitHub Pages or other hosting
- Add more interactive features and analytics
- Expand blog content with more technical articles
- Refactor build scripts for enhanced error handling and logging

---

*Note: Some assumptions were made about usage and deployment based on typical Hugo projects and the existing build scripts.*