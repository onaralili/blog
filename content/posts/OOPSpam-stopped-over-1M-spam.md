---
title: "My anti-spam API product stopped over 1M spam with %99 accuracy, and here are things I learned"
date: 2020-10-04
tags: ["spam filter", "oopspam", "anti-spam API"]
draft: false
repo: "https://www.onaralili.com/posts/OOPSpam-stopped-over-1M-spam"
---
<a href="https://oopspam.com/"> <img src="/images/oopspam-illustration.png" style="margin-bottom:1em;"/> </a>


[OOPSpam Anti-Spam API](https://www.oopspam.com) made its next milestone. It processed and filtered over 1 000 000 spam messages with %99 accuracy. This is big for me. When I started the project in 2017, I couldn’t imagine it will reach this point. Back then, I was doing research in this area as part of my master’s degree in Italy. Studying and digging research papers from Semantic Scholar and Google Scholar.

The first version of OOPSpam was a collection of APIs that merged into one. Nowadays, It is a standalone SaaS with powerful analysis capabilities.

After seeing so many different kinds of spam content, I relearned and abounded some lessons I got from my previous academic experience. In this article, I will lay out some thoughts about spam and their nature.

<center><img src="/images/spam-charter.jpg" alt=""/></center>

### It is a hard problem but …

Just when you think you have a way to completely solve this problem, a new wave of spam campaign proves you otherwise. Many companies struggle to tackle this issue. The problem is that there are usually two types of spammers: manual (human) and bot. While a bot crawls the web and submits messages to millions of websites, manual spammers go to a website and submit their content.

<center><img src="/images/robot-submit.png" width="35%" alt="a robot illustration"/></center>

I would still go ahead and claim that it is possible to reach a nearly perfect result with well-polished rules, Machine Learning models, content and IP analyses (that is how OOPSpam has 99% accuracy :) ). It is rather a strange problem. I can show a form submission that you would categorize it as spam but it is certainly not spam for someone. This brings us to my next point.

### One Man's Trash Is Another Man's Treasure

As Anti-Abuse Engineer, I dealt with a variety of spam. From a plain old spam to well-written, grammatically accurate (better than this article for sure) spam. Believe it or not, some people want to get SEO offers through their contact forms. Most people I know hate this kind of spam. So, how a spam-filter should approach this problem?

<center><img src="https://media2.giphy.com/media/SWoFzPlyaYpFgU02Ig/giphy.gif?cid=ecf05e47nj5jnpai80aslw0obnowgj9vm8uq9j5paq195kit&rid=giphy.gif" alt=""/></center>
 
Well, one way is: *Let them decide themselves*. OOPSpam produces Spam Score, the output from a variety of analyses. It is up to OOPSpam users to adjust spam sensitivity. Spam Score isn’t a new invention or any things but it is a rather forgotten one. You see, Spam Score was a standard back when people fought with email spam (well, they still do). For some reason, this method is abundant (perhaps for simplicity) over True/False, Spam/Not Spam for web spam. Web spam is a difficult problem because seemingly spam-like messages are not spam for many, so you may end up filtering out your potential customers’ message.

### Who are those manual spammers?

There are companies people pay to spam you. This is a million-dollar (estimated to be over $3M per year) industry. I recall seeing a contact form submission where a company offers to send 1 million messages for $49. What is problematic about human spammer is that the traditional techniques like captcha, the honeypot will not work. They will solve the captcha, fill the form, and submit it just like any other legitimate visitor would. To detect this kind of spam message are not easy. Machine Learning models alone tend to fail to detect these messages as they are trained to recognize a certain group of spam content. You need a set of different analyses to detect them. A combination of ML models and rule-based spam filers seems to work better.

### Spam bots are getting smarter

Ah... Bots. A nice little script usually enough to stop them. At least it used to be. Today, they also solve a captcha, bypass honeypot. Did I mention there are [AI-based spam solutions](https://www.itpro.co.uk/security/34784/the-future-of-spam-is-scary)? Here is a small paragraph from the article, just to acknowledge how crazy things may get:

> "... Neural networks that can read text, understand the context of an image and write believable messages all without human interaction so spammers can build more realistic, personalised messages, making it more difficult to filter them out from legitimate mail ..."

<center><img src="https://media4.giphy.com/media/3o751WENiYYifHgVlS/giphy.gif?cid=ecf05e471afdrxs3ggprrvrlmfaws4j2re4vhtpvn7pwbeu7&rid=giphy.gif" alt=""/></center>


So, both spammers and anti-spam filters are using the same technology to fight each other. As a result, more and more studies are needed to fight this new type of spam.

### Final thoughts 

I am happy how OOPSpam API progresses. There are still lots of work that needs to be done. First and foremost, now the API is served through RapidAPI as I was focusing on the actual problem. But it seems like many people do not want to deal with RapidAPI. So, this ought to be addressed. Also, hoping to write more blog posts about spam prevention techniques.