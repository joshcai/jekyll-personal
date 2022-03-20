---
layout: post
title:  "Painless URL Redirector with GitHub Pages"
date:   2022-02-27 06:13:34 +0000
categories: tech
shortlink: url-redirector
---

A few years ago, I had the idea to grab the domain name [joshc.ai](https://joshc.ai/). I thought it would be particularly apt because I could write about AI/ML (given that I've been working in ML infrastructure for a few years now).

I didn't at the time, since I already owned [joshcai.com](https://joshcai.com) and didn't know what I would do with another domain name.

Recently, I was thinking about getting the domain name again since I wanted to revamp my website. In the end my thought process boiled down to - *what I be sad if someone else bought [joshc.ai](https://joshc.ai/) later? Yes, I would...* So I bought it. I'm sorry if your name is Josh Cai and you had this idea too (but not that sorry).

I eventually thought that a good use of it could be a URL link shortener for my site, e.g. something like [bit.ly](https://bit.ly), especially since the full Jekyll blog post URLs can get quite lengthy. 

Initially, I was planning on doing a server-side implementation, e.g. write a small Golang binary to listen for requests and issue an HTTP redirect. I was chatting with my roommate about the idea and he was wondering - what it would look like if I did it on the client side instead?

I started wondering if it would be possible to do it entirely with [GitHub Pages](https://pages.github.com/), which I have been a huge fan of for its simplicity. 

It turns out it's super simple to do! You can create a [custom 404](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-custom-404-page-for-your-github-pages-site) page when the path doesn't match any resource.

Since the custom 404 page can include JavaScript, you can easily add something like this:

```html
  <script>
    const redirectDict = {
      '/shortlink': '/some/really/long/link/that/is/too/long/to/share/with/people/directly'
    }
    
    let pathname = window.location.pathname;
    if (window.location.pathname in redirectDict) {
      pathname = redirectDict[window.location.pathname];
    }
    window.location = 'https://domain.name' + pathname;
  </script>
```

This basically just checks if the path has a corresponding long URL, if so, return the long URL, otherwise pass the path as it is to the actual domain. Now just set up the GitHub Pages settings and you have a URL redirector!

You can check out my implementation here: [github.com/joshcai/redirector](https://github.com/joshcai/redirector).

For my setup, I have [joshc.ai](https://joshc.ai) pointing directly at the redirector and [joshcai.com](https://joshcai.com) pointing at my Jekyll blog at: [github.com/joshcai/joshcai.github.io](https://github.com/joshcai/joshcai.github.io). 

Why don't I have [joshcai.com](https://joshcai.com) have its own 404.html with the redirect logic? That's also definitely an option - but for me personally, I wanted [joshc.ai](https://joshc.ai) to expand to [joshcai.com](https://joshcai.com) each time, and having it all in one repo would make that harder - if [joshc.ai/foo](https://joshc.ai/foo) pointed to something that existed, it wouldn't expand. 

I'll end this post with an exercise for the reader - what does [joshc.ai/redirector](https://joshc.ai/redirector) redirect to? What about [joshc.ai/redirector/redirector](https://joshc.ai/redirector/redirector) and so on? 