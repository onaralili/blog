---
title: "OOPSpam API multiple spam filter"
date: 2017-08-02
tags: ["spam filter", "oopspam", "spam filter API", "SpamAssassin"]
draft: false
repo: "https://www.onaralili.com/posts/oopspamfilter"
---
<a href="https://oopspam.com/"> <img src="/images/oopspamWebsite.png" style="margin-bottom:1em;"/> </a>
visit https://oopspam.com/ </p>
I was thinking about some ideas as a side-project. Well, I thought it would be nice to pass my project-based exams Mobile Applications and Cloud Computing, Human-Computer Interaction with the same idea. I came up with the idea of VirusTotal-like service but as a spam filter service. As a result, OOPSpam API came to light. After making a few research I found out there is actually well-developed solution like mine, however, the price seemed to be high. I started development by integrating a few major spam filer services such as Apache SpamAssassin and deployed the application to Heroku. Roughly, this is how it works:
![how Oopspam works](/images/oopspam.png)

### So, what is it?

It is a multiple spam filter service that has built-in integration with powerful machine learning and text analyzers. It takes a content as an input and sends it to various spam filter services and returns the likelihood of the content to be a spam.
Integration is quite simple, the only thing you have to do is to send a request to
```url
https://oopspam.herokuapp.com 
```

with JSON object :
{{< highlight json >}}
{"text":"The example content!"}
{{< /highlight >}}

As a result, you will be responded with JSON object as following :

{{< highlight json >}}
{
"spam": 2,
"nospam": 1
}
{{< /highlight >}}

As obvious it might seem It is worth to mention that **spam** and **nospam** stand for a number of services that recognized the content as a spam/not spam.

I’m planning to integrate other providers as well to make it even accurate. The project can be helpful for the folks who want to protect users from the spam contents.
