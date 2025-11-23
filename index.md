---
slug: "github-blog"
title: "blog"
repo: "justin-napolitano/blog"
githubUrl: "https://github.com/justin-napolitano/blog"
generatedAt: "2025-11-23T08:17:37.010931Z"
source: "github-auto"
---


# Behind the Scenes of My Personal Blog: Building with Hugo and Python

Hey there! I wanted to share some insights into the personal website and blog I've been working on — the one you can find at [jnapolitano.io](https://jnapolitano.io). This project has been a rewarding blend of static site generation, content curation, and automation, all wrapped up in a clean, performant package.

## Why Build a Personal Blog This Way?

I needed a platform that could handle rich content — from detailed project write-ups to data-driven posts — while being fast, secure, and easy to maintain. Static site generators like Hugo fit the bill perfectly. They let me write in Markdown, organize content intuitively, and generate a fully static site that loads lightning fast.

But I also wanted some automation around building and deploying the site, so I crafted a Python script (`python-build.py`) that wraps common build tasks. This way, I can run a single command to clean, build, commit, and push changes — streamlining my workflow.

## How It's Built

### Hugo as the Core

At the heart of the project is Hugo, a fast and flexible static site generator. I use multiple themes (`hugo-coder` and `hugo-shortcodes`) to get a sleek look and handy content features like custom shortcodes and syntax highlighting. The `config.toml` file is where I configure everything — from site metadata and pagination to social links and menus.

### Content Organization

Content lives in the `content` directory, neatly split into posts, about page, projects, and contact info. Posts are written in Markdown with front matter for metadata like author, date, tags, and categories. This structure makes it easy to add new posts and pages.

### Styling and Scripts

I use SCSS for styling, with separate files for light and dark themes. There's also JavaScript to handle color scheme toggling, respecting user preferences and saving choices in localStorage.

### Automation with Python

The `python-build.py` script is a handy utility I wrote to automate my build pipeline:

- It installs Python dependencies from `requirements.txt`.
- Runs `make clean` and `make html` to build the site.
- Handles git commit and push steps with timestamps.

This script saves me time and reduces errors when updating the site.

## Interesting Details

- The site supports emoji rendering globally, which adds a fun and expressive touch to posts.
- Social and menu links are configurable in the `config.toml`, making it easy to update contact info or add new links.
- The blog supports pagination (20 posts per page) and syntax highlighting with Pygments.
- I use Hugo taxonomies like categories, tags, series, and authors to organize content.

## Why this project matters for my career

Building and maintaining this blog is more than just a personal outlet — it's a living portfolio of my skills in web development, data analysis, and automation. It showcases my ability to design and implement a full-stack solution, from content creation to deployment. Plus, it keeps me sharp with modern tools like Hugo, Python scripting, and front-end technologies. Sharing my projects and insights here helps me connect with the community and opens doors for professional opportunities.

Thanks for reading! If you're interested in static site generators or automating your own blog builds, I hope this gives you some useful ideas.

— Justin Napolitano