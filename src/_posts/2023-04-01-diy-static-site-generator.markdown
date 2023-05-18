---
title: "DIY static site generator"
short: "It's surprising how easy it is"
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

* Python is my goto language when not working on something Apple related
* [markdown-it](https://pypi.org/project/markdown-it-py/) - Markdown parsing and rendering
* [markdown-it-plugins](https://pypi.org/project/mdit-py-plugins/) - Markdown front-matter parsing 
* [Jinja2](https://pypi.org/project/Jinja2/) - HTML templating
* [livereload](https://pypi.org/project/livereload/) - what the name says

2 evenings and some 400 lines of code later I had everything I wished for. 

# Flow

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

And that's it. 

You can have a look the sourcecode for KSSG on my [GitHub](https://github.com/spherefoundry/kssg). I have released it 
under the MIT license. But I'm sharing it only for educational reasons. I have no intention of making it a real OSS
project. Accordingly, My needs will be prioritized over any backward compatibility, external feature request, etc.
