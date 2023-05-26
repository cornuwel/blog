---
layout: post
title:  "Hosting a Website for Free: Using Jekyll and GitHub Pages"
date:   2023-05-21
tags: blogging website hosting free
---

![A relaxed writer](/assets/jekyll-github.png)

In the era of digital prominence, hosting a website has become an integral part of building your online presence, be it for personal branding or business purposes. While there are many paid solutions, the beauty of the digital world lies in the fact that you can host a website for free, and with minimal technical knowledge. This article will introduce you to static site generators and demonstrate how you can use GitHub Pages to host your website at no cost.

## A Primer on Static Site Generators

Static Site Generators (SSGs) are a type of software that converts source files into a full website. They're increasingly popular because they create fast, secure, and scalable sites. Here's a list of some of the most popular static site generators:

1. Jekyll: Built on Ruby, Jekyll is one of the most popular SSGs, largely due to its integration with GitHub Pages. It has a vast user community, plenty of plugins, and supports Liquid templating language.
2. Hugo: Known for its blazing-fast build times, Hugo is an SSG written in Go. It has a robust content management system, supports custom output formats, and does not require any dependencies.
3. Gatsby: Gatsby is a React-based, GraphQL powered SSG. It's ideal for those who favor modern web tech stack and seek a rich set of features including image optimization, data prefetching, and more.
4. Next.js: Built on React, Next.js is not just a static site generator but a versatile framework that supports server-side rendering as well. It's preferred for complex projects that require scalability and flexibility.
5. Hexo: Hexo is an SSG built on Node.js, specifically designed for blogging. It supports a wide range of themes and plugins and offers a simple setup process.
6. Eleventy (11ty): A simpler and more flexible alternative to many other SSGs, Eleventy is based on JavaScript and supports various templating languages. It's highly praised for its speed and simplicity.

## Creating your site with Jekyll

Getting started with Jekyll is a straightforward process, even for beginners. Before you begin, ensure that you have a working Ruby environment with Bundler installed. Once this prerequisite is met, you can install Jekyll by running the command ``gem install jekyll bundler`` in your terminal. After the installation, you can create a new Jekyll site by executing ``jekyll new my-awesome-site``. Replace 'my-awesome-site' with the name of your site. This command will generate a new directory with the necessary files and folders to get you started.

Navigate into this new directory using cd my-awesome-site, and run ``bundle exec jekyll serve`` to start your site. By default, your new Jekyll site will be accessible in your browser at http://localhost:4000. From here, you can begin customizing your site's theme, adding posts, and more by modifying the provided files and adding your own. The simplicity and power of Jekyll make it a great choice for both beginners and experienced web developers.

## Using GitHub Pages for Free Website Hosting

GitHub Pages is a hosting service offered by GitHub for hosting static websites directly from a GitHub repository. It's particularly well-suited to Jekyll since it has built-in support for it. Here is a step-by-step guide to host your Jekyll website for free:

### Step 1: Create a GitHub Account

If you don't already have a GitHub account, head over to [GitHub's website](https://github.com/) and sign up. It's free and just takes a few minutes.

### Step 2: Create a New Repository

Once your account is set up, create a new repository. The repository name should be yourusername.github.io, replacing 'yourusername' with your actual GitHub username. This will be the URL where your website will be hosted.

### Step 3: Install Jekyll and Create Your Website

On your local machine, you need to install Jekyll. Instructions can be found in the official Jekyll installation docs. After the installation is complete, you can create a new site by using the command jekyll new my-awesome-site.

### Step 4: Push Your Website to the GitHub Repository

After building your site locally, you need to push the Jekyll site's content to your new GitHub repository. The content to push is usually everything in your Jekyll directory except for the _site folder, which contains the generated site. There is no need to push it because we will generate it with a Github Action below.

### Step 5: Configure the Repository

In your repository settings on GitHub, scroll down to the GitHub Pages section, and under Source, select the main branch for GitHub Pages.

Your website should now be live at https://yourusername.github.io.

### Step 6: Automate Jekyll Builds with GitHub Actions

A powerful feature of GitHub is the ability to use GitHub Actions to automatically build your Jekyll site every time you push your code to the repository. This process involves creating a workflow file in your repository (in ``.github/workflows/jekyll.rb``). Below is a simple workflow to get you started:

    name: Deploy Jekyll site to Pages
    
    on:
      # Runs on pushes targeting the default branch
      push:
        branches: ["main"]
    
      # Allows you to run this workflow manually from the Actions tab
      workflow_dispatch:
    
    # Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
    permissions:
      contents: read
      pages: write
      id-token: write
    
    # Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
    # However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
    concurrency:
      group: "pages"
      cancel-in-progress: false
    
    jobs:
      # Build job
      build:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout
            uses: actions/checkout@v3
          - name: Setup Ruby
            uses: ruby/setup-ruby@55283cc23133118229fd3f97f9336ee23a179fcf # v1.146.0
            with:
              ruby-version: '3.1' # Not needed with a .ruby-version file
              bundler-cache: true # runs 'bundle install' and caches installed gems automatically
              cache-version: 0 # Increment this number if you need to re-download cached gems
          - name: Setup Pages
            id: pages
            uses: actions/configure-pages@v3
          - name: Build with Jekyll
            # Outputs to the './_site' directory by default
            run: bundle exec jekyll build --baseurl "${{ steps.pages.outputs.base_path }}"
            env:
              JEKYLL_ENV: production
          - name: Upload artifact
            # Automatically uploads an artifact from the './_site' directory by default
            uses: actions/upload-pages-artifact@v1
    
      # Deployment job
      deploy:
        environment:
          name: github-pages
          url: ${{ steps.deployment.outputs.page_url }}
        runs-on: ubuntu-latest
        needs: build
        steps:
          - name: Deploy to GitHub Pages
            id: deployment
            uses: actions/deploy-pages@v2
    
    
This workflow checks out your code, sets up Ruby, builds your Jekyll site, and deploys it to GitHub pages every time you push to the main branch. It uses the GitHub Pages action by peaceiris to deploy your site.

## Bonus 1: Automatic dependencies updates

Dependabot is an invaluable tool provided by GitHub for automatically updating your project dependencies, or in this case, your Ruby gems. It periodically checks your project for outdated dependencies and opens pull requests with updates. This way, your project stays secure and up-to-date with the latest features and fixes. To set up Dependabot for your Jekyll site, you'll need to add a new file to your repository named ``.github/dependabot.yml``. The completed file should look like this:

    # Basic dependabot.yml configuration file
    # Updates gems and github-actions
    version: 2
    updates:
      # Enable version updates for Bundler
      - package-ecosystem: "bundler"
        directory: "/"
        schedule:
          interval: "daily"
    
      # Enable version updates for github-actions
      - package-ecosystem: "github-actions"
        directory: "/"
        schedule:
          interval: "daily"


With this setup, Dependabot will check for outdated gems in your Jekyll site on a daily basis, and automatically open pull requests when it finds updates. You can review and merge these pull requests to keep your site's gems updated, ensuring your project remains secure and utilizes the latest gem features and improvements.

## Bonus 2: Using a custom domain

Using a custom domain with GitHub Pages adds a professional touch to your website and enhances its visibility on the internet. To set this up, you first need to purchase a domain from a domain registrar of your choice. After you've secured a domain, you'll need to configure your DNS records with the domain provider. Specifically, you'll need to set up a CNAME record that points to your ``username.github.io`` site. Next, navigate to your GitHub Pages repository and go to 'Settings'. Scroll down to the 'GitHub Pages' section and you'll find a field to enter your custom domain. Type your domain here and save it. GitHub automatically creates a CNAME file in your repository with this custom domain. Remember to also select the 'Enforce HTTPS' checkbox to ensure your website uses SSL for secure connections. It may take up to 24-48 hours for all changes to propagate through the DNS system. After that, your GitHub Pages site will be accessible at your new custom domain.

## Conclusion

Jekyll combined with GitHub Pages offers a powerful, flexible, and cost-effective solution for hosting your static website. Utilizing GitHub Actions for automatic build and deployment ensures your site stays updated with every push, providing a seamless web development workflow. Happy website building!
