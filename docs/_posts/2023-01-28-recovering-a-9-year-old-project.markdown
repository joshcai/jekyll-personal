---
layout: post
title:  "Recovering a 9-year Old Project"
date:   2023-01-28 06:13:34 +0000
categories: tech
shortlink: website-recovery
---

Last August, Heroku announced that they were [removing their free tier](https://blog.heroku.com/next-chapter). Heroku was one of the first PaaS solutions I used when learning how to create web apps, and it made it insanely easy to actually deploy a project. 

When I was a college student at the University of Texas at Dallas, I helped TA a Computer Science camp aimed at teaching young kids how to code. This particular camp was a week long and taught kids how to use processing.js via Khan Academy.

After the first day, the instructor told the students to email him their code so he could share some of the images they created the next day. I had just started learning how to develop web apps with Django, and I thought - "what if I made a simple web app where they could host their code so they could share it with each other?" 

So I went home and hacked together a prototype, and thanks to Heroku, got it deployed - I told the instructor and they were onboard with trying it out. 

For my first web app to ever be used by others, it went surprisingly well! The students were excited to share the code they had made with each other, and I personally am glad I was able to preserve a bit of the code for my own memory.

Here's examples of what they created:

[![Square Circle Fractal](/assets/img/website_recovery/square_circle_fractal.png){:class="img-medium"}](https://utdcs.joshcai.repl.co/post/47/)

[![Line Vortex](/assets/img/website_recovery/line_vortex.png){:class="img-medium"}](https://utdcs.joshcai.repl.co/post/18/)

After the camp, I just left the code running on Heroku - and it ran happily there for around 7 years.

At some point, I got a notification that the Heroku stack it was running on would be deprecated - I had tried initially to fix the issue on Heroku, but it had been a while and never got around to fixing the issue. But before the stack was deprecated, I decided to go ahead and export all the raw HTML files by doing a simple curl script:

```bash
#!/bin/bash
for i in {2..93}
do
   curl http://utdcs.herokuapp.com/post/$i/ >$i.html
done
```

My reasoning at the time was that all the data was on the webpages, and if I really needed to, I could scrape the data from it and reconstruct the site at a later point. 

Since the free tier was now being discontinued completely at Heroku, I realized that I should attempt to migrate the app while I still had a chance - I decided to try and migrate the app to be hosted on Replit instead.

### Django migration errors

For some reason, I never saved the requirements.txt (this was probably so early on that I hadn't learned the importance of that) - so doing a fresh pip install downloaded the latest Django. 

I quickly ran into some migration issues, due to [Django deprecations](https://docs.djangoproject.com/en/dev/internals/deprecation/). 

Luckily, only a few lines needed to be updated here and there, mostly around the URL resolving, and having Google + StackOverflow made finding the right replacements pretty straightforward.

### Restoring the database

With the app running again, I needed to re-populate the database so that all the student's entries were there again. For the original app, I used Postgres, but I decided that was a bit overkill for such a small app so decided to use SQLite in the migrated version.

One benefit of using SQLite is that the database is just a file - that meant that I can check in the database at the end and not have to worry about losing access to the database server in the future. 

I couldn't access the Postgres database directly anymore, so I decided to scrape the information from my exported HTML files (thank you, past Josh!!). 

I had heard of [BeautifulSoup](https://beautiful-soup-4.readthedocs.io/en/latest/) for extracting data from HTML but had never used it before. As it turns out, it was actually super easy to use - a couple of lines of Python was able to extract all the info I needed:

```python
for file in files:
  print('Restoring file: ', file)
  path = os.path.join('backup', file)
  text = open(path, 'r').read()
  soup = BeautifulSoup(text, 'html.parser')
  title = soup.find(class_='post-title').find('strong')
  if title.string:
    title = title.string.strip()
    link = None
  else:
    link_element = title.find('a')
    link = link_element['href']
    print('  Link: ', link_element['href'])
    title = link_element.string
  content = soup.find(class_='fit-box').string
  author_and_time = soup.find(class_='post-title2')
  author = author_and_time.find('a').string.strip()
  time = author_and_time.getText().split(' on ')[-1].strip()
```

Now that I had all the data extracted, I just needed to find a way to add them back to the database. I thought about inserting them manually into the database, but with Django managing the schema of the database, I eventually settled on re-using the web app itself to do the submission.

Getting the exact `curl` command turned out to be a bit tricky, butysome experimental [Postman](https://www.postman.com/) queries allowed me to figure out exactly what syntax I needed to use. Since I wrote the data extraction part in Python, I just called `curl` in a subprocess with Python:

```python
def encode(name, content):
  return f" --data-urlencode '{name}={content}'"

def curl(author, title, content, time, link=None):
  command = """curl --location --request POST 'https://utdcs.joshcai.repl.co/submit/' --header 'Content-Type: application/x-www-form-urlencoded' """
  command += encode('title', title)
  command += encode('content', content)
  command += encode('author', author)
  command += encode('time', time)
  if link:
    command += encode('link', link)
    command += encode('linked', 'true')
  result = subprocess.check_output(command, shell=True)
```

### Recovering lost code

After restoring the database, I found there were a lot of missing features in the version of the app I was running. I realized that, in the past, I had been pushing code directly to Heroku without checking it in! 

I got lucky here again - I was able to find all of the raw files via git on Heroku directly (and it was actually on a second Heroku account that I had forgot about). I copied the files that were deployed in the production app into my local git repository and re-did any Django migrations. 

With that, my app was fully restored! In case you want to browse any submissions, you can check out [joshc.ai/utdcs](https://joshc.ai/utdcs).