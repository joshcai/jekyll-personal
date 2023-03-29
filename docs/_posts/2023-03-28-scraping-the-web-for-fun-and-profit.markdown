---
layout: post
title:  "Scraping the Web for Fun and Profit"
date:   2023-03-28 06:13:34 +0000
categories: tech
shortlink: web-scraping
---

Recently, I was trying to scrape a website for the purposes of knowing when there would be new available units in an apartment complex and monitoring for price changes. 

The main program loop consisted of:

- SQLite file for holding the most recent prices
- Python `requests` library for fetching HTML
- BeautifulSoup for parsing HTML and extracting relevant information
- Daemon thread executing periodic checks and updating SQLite file if prices changed
- Email-to-text alert when prices changed / dropped below a threshold
- Flask server + VueJS frontend for easily displaying all the extracted prices in one page

Along the way, I learned a couple of neat tricks:

### Sending a text notification for free

One of the crucial bits was that I wanted to be notified as soon as a new place became available, so I basically needed a way to get a text alert. I looked into a few services, like Twilio, but I didn't want to pay / set up an account. 

Luckily Email to SMS (which I had never heard of before) is a thing - if you send an email to the right address, it gets forwarded as a text to your phone! 

You can use a snippet of Python code like this:

```python
def send_message(message, phone_number, carrier):
  CARRIERS = {
    "att": "@mms.att.net",
    "tmobile": "@tmomail.net",
    "verizon": "@vtext.com",
    "sprint": "@messaging.sprintpcs.com",
    "fi": "@msg.fi.google.com"
  }
  recipient = phone_number + CARRIERS[carrier]
  email = "<email>"
  password = "<password>"

  server = smtplib.SMTP("smtp.gmail.com", 587)
  server.starttls()
  server.login(email, password)

  server.sendmail(email, recipient, message)
```

And with that, you can get free text alerts - they're not instant but fast enough for the purposes of what I needed.

### Debugging failed network requests

After hacking up the prototype and using it successfully for a week, I ended up needing to monitor a few more floor plans. I went and added the code for scraping the information of the other floor plans, but for some reason all the network requests were returning status code 400, even the ones that were working before.

It turned out that the server was expecting some particular headers to be set or else it wouldn't respond with the normal HTML response.

I inspected the network requests in the `Network` tab in Chrome and found all the cookies that were being set - I added all of those to the request, but that still didn't help. 

I found out that you can actually copy the exact network request as a cURL command by right clicking a request and going to `Copy > Copy as cURL`, for me that looked something like: 

```
curl 'https://www.apartmentname.com/...' \
  -H 'Accept: */*' \
  -H 'Accept-Language: en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7' \
  -H 'Connection: keep-alive' \
  -H 'Cookie: LOTS=OF_YUMMY_COOKIES' \
  -H 'Referer: https://www.apartmentname.com' \
  -H 'Sec-Fetch-Dest: empty' \
  -H 'Sec-Fetch-Mode: cors' \
  -H 'Sec-Fetch-Site: same-origin' \
  -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36' \
  -H 'X-NewRelic-ID: XAICfRGwxQBXFdaBAI=' \
  -H 'X-Requested-With: XMLHttpRequest' \
  -H 'sec-ch-ua: "Google Chrome";v="111", "Not(A:Brand";v="8", "Chromium";v="111"' \
  -H 'sec-ch-ua-mobile: ?0' \
  -H 'sec-ch-ua-platform: "macOS"' \
  --compressed
```

Executing that in a command line actually returned what I wanted - from there I was able to roughly binary search over all the different parameters and found that the server wanted the `Referer` and `X-Requested-With` headers, which fixed the issue.