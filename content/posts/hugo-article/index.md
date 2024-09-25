---
title: "Guide to Setting Up a Blog with Hugo and GitHub Pages"
date: 2024-09-20
tags: ["Hugo", "GitHub Pages", "Blogging", "Web Development"]
description: "A step-by-step guide on how to set up your own blog using Hugo and GitHub Pages."
draft: false
---

## Introduction

In today's fast-paced digital world, having your own blog is a great way to share knowledge, experiences, and ideas. In this guide, I will show you how to set up a static website using Hugo and host it for free on GitHub Pages. Hugo is a fast and flexible static site generator, and GitHub Pages makes it easy to host the site directly from your GitHub repository.

By the end of this guide, you will have a fully functional blog running on Hugo and deployed through GitHub Pages.

## What is Hugo?

Hugo is a powerful and fast Static Site Generator (SSG). It allows you to create websites that are completely static, which makes them fast, secure, and easy to host. With Hugo, you can write content in Markdown and easily customize your site's design using templates and themes.

## Why GitHub Pages?

GitHub Pages is a hosting service from GitHub that lets you publish websites directly from a GitHub repository. It's particularly useful for projects, personal blogs, and documentation. The integration with GitHub makes it easy to manage changes and keep your site up to date.

---

## Prerequisites

Before we get started, make sure you have the following installed on your system:

- [Git](https://git-scm.com/)
- [Hugo](https://gohugo.io/getting-started/installing/)
- [GitHub Account](https://github.com/)

---

## Step 1: Setting Up Your Hugo Site

First, create a new Hugo site on your local machine. Navigate to the directory where you want to store your blog files and run the following commands:

```bash
hugo new site my-blog
cd my-blog
```

This will create the structure of your Hugo site in a directory called my-blog.
Next, you'll need to add a theme. In this case, let's use the popular "Blowfish" theme. Run:

```bash
git init
git submodule add https://github.com/nunocoracao/blowfish.git themes/blowfish
echo 'theme = "blowfish"' >> config.toml
```
---

## Step 2: Create Your First Blog Post
Now that the site structure is ready, create your first blog post by running:

```bash
hugo new posts/my-first-post.md
```
This will generate a new Markdown file for your post inside the content/posts/ directory. You can edit this file with your favorite text editor and add the content for your first post.

---
## Step 3: Preview Your Site Locally
Before deploying your blog, you can preview it locally to see how it looks. In the root directory of your Hugo project, run:
```bash
hugo server -D
```
Open a browser and go to http://localhost:1313/ to see your site in action.

---
## Step 4: Preparing for Deployment to GitHub Pages
Now it’s time to deploy your blog to GitHub Pages.

### 4.1 Create a GitHub Repository
Go to GitHub and create a new repository. You can name it <username>.github.io to use it for GitHub Pages. If you want to host it on a subdomain, you can use another name like my-blog.

### 4.2 Build the Static Site manually
Run the following command to build your site:
```bash
hugo --minify --themesDir ../.. -d docs --baseURL https://username.github.io/
```
This will create a docs/ directory with all the static files of your site.

### 4.3 Push to Github
```bash
cd my-blog
git init
git remote add origin https://github.com/<username>/<repository>.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

---

## Step 5: Automating the Build with GitHub Actions
Instead of manually building and deploying your site every time you make a change, you can automate this process with GitHub Actions.

### 5.1 Create the GitHub Action Workflow
In the root directory of your project (not the docs folder), create a directory called .github/workflows (if it does not exist already):

```bash
mkdir -p .github/workflows
```
Create a new file inside the workflows directory called gh-pages.yml and add the following content (adjust the URL):

<details>
<summary>gh-pages.yml</summary>

```bash
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Blowfish Docs Deploy

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]
#
  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow one concurrent deployment
concurrency:
  group: "pages"
  cancel-in-progress: true

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.102.3
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_Linux-64bit.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Build with Hugo
        env:
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo --minify --themesDir ../.. -d docs --baseURL https://username.github.io/
          echo "Build completed"
          ls -la docs
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./docs


  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: https://emrehayta.github.io/
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
</details>

### 5.2 Commit the Workflow
Commit and push the workflow file to your repository:
```bash
git add .github/workflows/gh-pages.yml
git commit -m "Add GitHub Actions workflow for Hugo"
git push origin main
```


---
## Step 6: Configure GitHub Pages
1. In your GitHub repository, go to Settings
2. Scroll down to the GitHub Pages section.
3. Under Source, select the docs/ folder in our case (depending on your setup)
4. Save the settings, and your site will be live!

---

## Conclusion
Congratulations! You've successfully set up a blog using Hugo and GitHub Pages. From here, you can continue customizing your theme, adding posts, and growing your site. Hugo and GitHub Pages offer a great, scalable way to maintain a static blog with minimal costs and maximum flexibility.

Happy blogging!

