---
title: "DIY static site generator"
short: "To support this blog and my various other online presences I had a need for a static site generator. I needed something 
simple and flexible without overcomplicating things."
order: 2
---

# TL;DR;

To support this blog and my various other online presences I had a need for a static site generator. I needed something 
simple and flexible without overcomplicating things. So I deiced to go with a custom solution. The KSSG (Kedracki Static 
Site Generator, dah) was born. The project turned out to be surprisingly simple and fun to write.

# Static site generator

I guess a lot of people know what a static site generator (short SSG) is and what benefits it has, but for the sake of 
completeness let me draft an outline. 

A static site generator is exactly what the name suggests. It takes some form of input (let's get back to that 
in a second) and generates a static website out of it. Static in the sense that it can be served from a webserver 
directly as a set of HTML/CSS/JS/whatever files, without executing any additional code on each request. Serving 
files in this way is very fast. And because it is static it also reacts to caching very nicely, so a CDN can make it 
even faster. 

On the input side of things we are pretty free to choose whatever we like as long as it can be used to generate HTML. 
A common approach is to keep the template (HTML, CSS, JS, etc) separate from the content (e.g. Markdown). Beneficial, 
as it will yield uniform presentation for all content, with the ability to quickly re-style the whole site.

An added benefit of static site generators is how well they integrate into a CI/CD pipeline. You can keep the content and 
template in a git repository, and run the generator on each commit.   

All of the above make static site generators very popular for writing blogs and landing pages. Free hosting on Github
Pages is just the cherry on the cake.

# DIY?

There are a great many tools for SSR. One of the most popular ones is [Jekyll](https://jekyllrb.com/). I used it 
previously, so it was my first choice when starting to work on this blog site. Sadly, as was the case in the
past, I quickly started to feel constrained by the rigid way Jekyll organizes things. This triggered my decision 
to try to build a generator myself.

I quickly wrote down what I need:

* markdown for content - for blog type sites
* ability to freely shape other sites - for landing page type sties 
* some sort of inheritance for HTML files - I hate to repeat my self
* local preview with livereload - because it makes writing so much nicer

Not that much, right? I specifically don't need plugins, switchable/reusable templates/themes, etc. With the 
requirements sorted out, it was time to select the tools:

* [Python](https://www.python.org/) is my goto language when not working on something Apple related
* [markdown-it](https://pypi.org/project/markdown-it-py/) - Markdown parsing and rendering
* [markdown-it-plugins](https://pypi.org/project/mdit-py-plugins/) - Markdown front-matter parsing 
* [Jinja2](https://pypi.org/project/Jinja2/) - HTML templating
* [livereload](https://pypi.org/project/livereload/) - what the name says

2 evenings and some 400 lines of code later I had everything I wished for. 

# Generation Flow

The data flow is really simple:

* Scan the input folder in search of files
* Classify the files depending on path, name and extension. KSSG distinguishes between following types: 
  * Ignore - file has a "_" prefix
  * Post - is located in the dedicated directory, has a properly formatted name (year-month-day-slug), and one of the 
      valid post filename extensions 
  * Template - has one of the valid template filename extensions
  * Static - everything else
* Collect additional data about the items (e.g. load front-matter from Post items)
* Process each input file taking its type into account:
  * Ignore - skip
  * Post - first render the markdown into content HTML. Then render the post site using the dedicated Jinja2 template
  * Template - render the site using Jinja2
  * Static - just copy the file

And that's it. One area of improvement I would like to have a look soon is to generalize handling of Posts. 
Specifically, being able to configure how the resulting files are being placed in the output hierarchy would allow for 
the same mechanism to be used for other forms (e.g. documentation).          

# CI/CD

This site is hosted on [GitHub Pages](https://docs.github.com/en/pages). Setup is 
super easy. You create a repository named `<your_org>.github.io` and push your static site. Then GitHub automatically 
deploys the site to the domain matching the repository name. Additionally, you can configure yourself a [custom domain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages) 
including HTTPS.

A slight shortcoming is that there are only two settings for where in the repository GitHub will look for the site 
source. It can be either the root `/` or a doc directory `/doc`. As a result, when using KSSG, we have to either commit 
only the build output to the repository (selecting root as the source), or configure KSSG to output into the `/doc` 
directory. In both cases you will have to remember to build your site before committing.

But what would a software project be without CI/CD? GitHub allows for Actions to control the deployment process of your 
site. Below you will find a workflow file that automates building the site. In case of this blog it resides under 
`.github/workflows/kssg-gh-pages.yml`. 

```yaml
name: Build site with KSSG and deploy to GitHub Pages

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
    env:
      # FIX: needed because of some ubuntu/setuptools incompatibility, resulting in packages named UNKNOWN
      DEB_PYTHON_INSTALL_LAYOUT: deb_system
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Install KSSG
        run: |
          pip install --force-reinstall "setuptools>=67.7.2"
          pip install -r requirements.txt
      - name: Build with KSSG
        run: |
          python -m kssg rebuild
      - name: Setup Pages
        uses: actions/configure-pages@v3
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./output

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
```

This defines two jobs: `build` and `deploy`. The first installs kssg inside the runner, setups the GitHub Pages 
environment, build the site, and uploads the build result to GH. The second job triggers the deployment of the 
uploaded artifact.

The whole thing takes around 50 seconds to complete, with most of this time spend installing KSSG.

# Code? 

You can have a look the sourcecode for KSSG on my [GitHub](https://github.com/spherefoundry/kssg). I have released it 
under the MIT license. But I'm sharing it only for educational reasons. I have no intention of making it a real OSS
project. Accordingly, My needs will be prioritized over any backward compatibility, external feature request, etc.
