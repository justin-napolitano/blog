---
slug: github-blog-note-technical-overview
id: github-blog-note-technical-overview
title: Blog Repository Overview
repo: justin-napolitano/blog
githubUrl: https://github.com/justin-napolitano/blog
generatedAt: '2025-11-24T18:31:32.038Z'
source: github-auto
summary: >-
  This repo is my personal blog and website, built using the Hugo static site
  generator. It features flexible themes, rich markdown content, and social
  media integrations.
tags: []
seoPrimaryKeyword: ''
seoSecondaryKeywords: []
seoOptimized: false
topicFamily: null
topicFamilyConfidence: null
kind: note
entryLayout: note
showInProjects: false
showInNotes: true
showInWriting: false
showInLogs: false
---

This repo is my personal blog and website, built using the Hugo static site generator. It features flexible themes, rich markdown content, and social media integrations.

### Key Components:
- Built with **Hugo** for static site generation
- Supports markdown, shortcodes, and syntax highlighting using **Pygments**
- **Python** for build automation and a **Makefile** for easy execution
- Custom layouts and archetypes for posts

### Running Locally:
1. Clone the repo:
   ```bash
   git clone https://github.com/justin-napolitano/blog.git
   cd blog
   ```
2. Install any Python dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Build the site:
   ```bash
   python python-build.py
   # or
   make html
   ```
4. Serve it locally:
   ```bash
   hugo server -D
   ```

Visit `http://localhost:1313` to see it in action. 

**Gotcha:** Ensure Hugo and Python are installed beforehand. Happy blogging!
