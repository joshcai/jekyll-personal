---
layout: post
title:  "repl.it for Jekyll + GitHub Pages"
date:   2022-01-26 06:13:34 +0000
categories: tech
shortlink: jekyll-setup
---

Recently, I have been using [repl.it](https://repl.it) more and more for my small side projects - it's been super convenient for several reasons:

- no needed to set up dev environment on different computers / OSes
- automatic dependency isolation between projects
- convenient layout for editing / viewing changes in one screen
- native GitHub integration - especially useful for GitHub Pages-based sites

I have also been wanting to revamp my website for a while (my old website has said that I am 24 years old for the past three years...). I thought it would be fun to attempt blogging (again), so I decided to finally try [Jekyll](https://jekyllrb.com).

I first checked out [jekyll-now](https://github.com/barryclark/jekyll-now), which was really easy to get started with, but I ended up not using it since I wanted a way to easily preview my posts before publishing them. Instead of installing Jekyll locally to do that, I wanted to see if it was possible to install it in a repl.it project. 

I couldn't find a guide for setting up Jekyll in a repl.it project (the closest was [this thread](https://replit.com/talk/ask/Is-there-a-way-to-install-Jekyll-for-Github-Pages/28944), which was helpful), so I wanted to write a post to share some tips I found while trying to set it up myself.

### Setup

If you want to skip the setup, you can fork this [GitHub project](https://github.com/joshcai/jekyll-replit-template) or this [repl.it project](https://replit.com/@josh_cai/jekyll-replit-template).

##### Jekyll

For the most part, you can just follow the steps under "Creating your site" in the GitHub documentation for [setting up Jekyll with GitHub Pages](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll). You can ignore the `git` commands, since `repl.it` has its own GitHub integration (we'll get to it below).

When the post asks you to run:

```
$ jekyll new --skip-bundle .
```

It may give you a prompt starting with `jekyll: command not installed ...` - you can choose `jekyll.out`. 

Alternatively, you can also install `jekyll` through the package manager (which is slower) and do the following:

```
$ bundle exec jekyll new --skip-bundle .
```

When it asks you edit the `Gemfile` in the `docs` folder, you might find that the `Gemfile` doesn't show up in the Files tab. However, it does show up if you use `ls` in the Console or Shell. So far, the only workaround I know to edit the `Gemfile` is to use `vim` (or any other terminal-based text editor) in the Console / Shell. 

After editing the `Gemfile`, we can move on to editing the repl.it config.

##### repl.it Config

To work seamlessly with repl.it, I wanted to be able to press the `Run` button and have it compile the site. To do that, you can add a `.replit` config to the top-level directory if it's not there already and then configure the `run` variable:

{% highlight ruby %}
run = "cd docs && bundle install && bundle exec jekyll serve --host=0.0.0.0" 
{% endhighlight %}

Note: if you don't see the `.replit` file in the Files tab, you can click the three dots to the right of `Files` and click `Show hidden files`.

The `run` command first does a`bundle install`, since repl.it doesn't seem to save the installed gems across multiple working sessions. This does make it a bit slow to start up the first time you press the `Run` button, but it should be pretty fast after that within the same session.

You should now be able to press the `Run` button and see the default Jekyll page being served.

##### Push to GitHub

When you're ready to push to GitHub, just navigate to the "Version control" tab on the left panel in repl.it. After creating a GitHub project from that tab, you can then continue following the guide for [setting up Jekyll with GitHub Pages](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll) - you'll just need to modify the settings so that the GitHub Pages deployment is set up.

That's pretty much it! Once GitHub Pages is configured, you should be able to see your Jekyll site served at the GitHub Pages URL.