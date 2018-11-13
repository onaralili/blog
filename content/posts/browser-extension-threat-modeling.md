---
title: "Browser Extensions Threat Modeling"
date: 2018-11-10
tags: ["Threat Modeling", "browser", "browser-extension"]
draft: false
repo: "https://www.onaralili.com/posts/browser-extension-threat-modeling"
---
_This is a report I wrote for one of courses while doing my master studies. Some parts are heavily influced by other realted studies. Please check References section_

## Introduction
Today, users surf the internet using browsers such as Google Chrome, Firefox, Safari, IE (Edge) etc., these are main browsers are being utilized by users. Statistically, 56.31% of the users use Chrome as their default browser [1]. Therefore, we will mostly focus on Chrome and discuss the way the company protects users from malicious threats from the extensions. Extensions virtually can access all resources just like a browser. Chrome protects users from behaviors that were not originally offered by the extension by running each extension in the sandbox mode. As an example, if an extension would like to make HTTP request outside of its origin, it is bound to ask additional privileges, however, it can make an HTTP request to its own installation sandbox, say if the extension contains .json file, there is no need for extra permission to ask. According to the survey [2], most of the users don't pay attention to permissions asked by the extension or don't understand technical detail behind it. This raises concern about misuse of the privileges which may lead to stolen credit card information, passwords and other sensitive personal information.
Beside permission-based protection and sandbox installation, there are other counter measurements are taken by the companies to reduce risks. One of them is an automated code review of the submitted extension. The effectiveness of automated reviews is questionable since there are many cases where malicious extensions have passed the test successfully and being downloaded thousands of users [3]. On the other hand, Firefox used to take a different approach by carrying manual review, however, this program ended in September 2017 [4]. After a week the program dropped, cryptocurrency miners were being reported by users in the Firefox add-ons [5].
As we see, automated threat analyzes are flawed and obviously not as effective as a manual review. In this report, we will go through potential risks, how they work and how they can get sensitive information from your browser activities, how they can secretly mine cryptocurrency and inject ads. To the end, we will point out possible measurements that can be taken to limit these activities.

## Architecture and Potential threats
From now on, we will see the system from an attacker's perspective and how a possible malicious extension can misuse its privileges. Before we start, let's see how Chrome architecture works and handles extensions. I roughly demonstrated how the architecture works in Figure 1. As you see from the illustration, a website runs within it is origin as well as an extension. Chrome defines an extension's origin based generated unique URL address. An extension can access and interact its own file system as websites do. On the contrary to websites, an extension can ask for the cross-origin permission, therefore, having an access to make cross-origin requests. What this means for an end-user is that a malicious extension can inject a code and send an HTTP request on behalf of the inject web pages.\
<img src="/images/browser-extension-threat-modeling/browser-extension-architecture.png" alt="Browser-Extension Architecture" width="300"/> \

Back to the figure, an extension can interact with a browser's and system's APIs in the case of granted privileges, obtaining almost the same rights as a browser. An extension works in Execution environment [6] also known as an isolated world where say inject JS code work within its environment and doesn't conflict with a websites JS functions. As a part of the security model, in order to an extension has access to resources it defines its desired permissions in a manifest.json file, which looks like following:
```
{ ...
    "permissions": [
    "tabs" 
 ]
}
```
In Figure 2, we can see how above permission syntax shows up for a user. It may not be obvious from the dialog box what exactly Read your browsing history means for a user and this is why many users think of it as an extension will be able to read my browsing history as it claims. The catch is that this permission alone allows a developer to:\
─ Full access to DOM elements of a webpage \
─ Inject executable JS code into a webpage \
─ Insert CSS code into a webpage \
<img src="/images/browser-extension-threat-modeling/extension-permission.png" alt="Chrome Extension Permission" width="300"/> \

These are only three of the permissions that have been given based on <tabs> privileges. Content Scripts is another system where a developer may want to always inject a JavaScript code into every page opened by a user. To achieve this content_scripts field utilized in manifest.json as following:
```
"content_scripts": [ 
    {
    "matches": ["http://www.example.com/*"],
    "js": ["jquery.js", "myscript.js"] 
    }
]
```
As you can see permission system opens a wide range of attack opportunities for an intruder. These rich features are potential tools for an attacker to manipulate a system. I will talk more about this in the Thread Model section.
Once an extension has been installed it plays a role as a "bridge" between a user and a website. Figure 3 demonstrates this concept, this feature mostly used by ad blockers since they try to filter out included JavaScript codes from the DOM, therefore before showing webpage to the user it has to go through the extension to achieve the desired result. \
<img src="/images/browser-extension-threat-modeling/extension-interaction.png" alt="User Access" width="300"/> \
Unfortunately, this is also a shiny opportunity for the malware companies to inject ad-
vertisement into the page without user's consent.


## Threat Models
In the threat model, we are going to model an extension that takes advantage of the features available for a developer. We also assume that malicious extension has already been installed. There are many ways to do so, such as installing it to the file system under the directory where extensions live via malicious software, legitimately publishing a malicious extension on Chrome Web Store by avoiding automatic malware analysis. Another common technique is to buy high ranked extensions from developers, this is a widespread method has been observed in very popular extension where advertisement companies contact to a developer and buy the extension then publishes it with ad injection code which as a result, injects ads to the active user’s browser activities. In order to demonstrate threats, we will design a number of malicious extension by utilizing APIs offered by the extensions.

### Data Leak
In the following model we simply ask for the following permission:
```
"permissions": [ 
    "webRequest",
    "http://*/" 
]
```
and a user will see it as a form of _Read and change all your data on the websites you visit._ This is a common privilege in the extensions and almost all the time ignored and is not being taken seriously by users.
This is the feature mostly misused for hijacking users' credentials by collecting information. The malicious extension will fire an event whenever a user types something on “password” field and stops, at this stage, the extension will assume the password is being typed. Afterward, it will try to find "username" field in the DOM and get its value as well as URL of the website. At this point, the extension will successfully collect username, password, URL. Let's see how extension gets username and password in detail, a password field in the DOM is pretty straightforward since it has type="password" and JavaScript selector can easily access it, however, the username is challenging since developers name username field differently. In order to overcome this challenge, we created following simple wordlist:
```
"loginEmail", 
"username", 
"email", 
"loginName", 
"login", 
"id_username", 
"UserId",
"userid", 
"user_email",
"useremail", 
"userEmail", 
"login_email"
```
Despite the fact that it is a short list, it is very effective. This list will be used as lookup table each time when a user signs in a webpage to get the username of the user.
At this point, we can send the collected data to remote server. To meet our needs, I developed a simple RESTful API server to receive collected information. The extension automatically posts the collected sensitive information to the API and the server stores it in the local database. In Figure 4, we can observe stored data from the user's browser. \
<img src="/images/browser-extension-threat-modeling/sample-data.png" alt="Collected sample data from a user" width="300"/> \
As we can see, a simple common privilege access can lead a massive leak of sensitive data, this technique can be extended to collect other kind of personal information such as payment credentials (PayPal, credit card etc.), or identity stealing.
### HTTP FLOOD
Since our extension has a right to make an HTTP request, we can perform HTTP Flood [7] which is a type of DDoS attack where an injected machine will make GET or POST request to the target. We are going to modify above extension to model HTTP Flood. Firstly, we need a target where the requests will be sent. As we mentioned above, an extension receives an update from the web store. This is a practical way to use Command and Control technique to send a target address to the bot. This can be done as simple as creating a .txt file that contains information about target URL and interval. Here is the example of such a file:

```
URL = http://twitter.com/ 
Interval = 1000
```
In order to tell injected machines that we want to attack to a certain address. We publish an updated .txt file on web store as a part of new version of the extension. Afterward, the browser takes care of distributing the file to the users. Once a command has been received from the server, in every opened tab the extension will perform HTTP Flood by making an HTTP request to the target with the indicated interval. This technique has been also utilized for mining cryptocurrencies as well [8].
## Countermeasures
Under this header, we will discuss what kind of measurement we can take to ensure integrity, confidentiality, and privacy. As we observed above, the extension has a full access to DOM and right to modify it. This is achieved by simply asking for http://*/* permission. There is no clear indication for users to know that full access to DOM as well as cross-origin HTTP request has been granted.
### In detail privileges
As we saw so far, the permission system currently utilized by the browser is simple, meaning it covers a wide range of features, however, the only message a user sees is as simple as _Reading your browsing history or Read and change all your data on the websites you visit_, which are unclear for the end-user.
One way to solve this is to expand the scope of the privileges presenting a new, more advanced privileges system. Let's look at this using the extension we demonstrated on the Threat Model section. In our case, the bad actor wants to access to: 
    
| Desired access  | Permitted under the permission  |
|--- |---|
| DOM  | http://*/  |
| Read sensitive field from the DOM  |  http://*/ |
| Send cross origin requests  |  http://*/ |  

These features allow the malicious actor to accomplished what illustrated.
We introduce new permission keywords that need to be asked by the developers to have those accesses.  
  
| Desired access  | Permitted under the permission  | Access level |
|--- |--- | ---|
| DOM  | dom_access  | read, write
| Read sensitive field from the DOM  |  dom_sensitivity_access | low, medium, high
| Send cross origin requests  |  same_origin / cross_origin | allow  
  
In this table, we show newly introduced access privileges by allowed access values. These relatively in detail permission system, will allow us to show a user clear message what exactly has been accessed by the malicious actor. The implementation of the new permission is straightforward and can be easily indicated manifest.json file as current permission system. In order to keep it simple and compact, we will use a hierarchical tree as shown:

```
"permissions": [ 
    {
    "dom_access": [ 
        "read",
        "write" 
    ],
    "dom_sensitivity_access": [ 
        "high"
    ], 
    "same_origin": [
    "allow" 
    ],
    "cross_origin": [ 
        "allow"
    ] 
},
]
```
The _dom_access_ reveals that we want to access to DOM to read and write. These allow us to show user following message before installing an extension:
_Read and Write the websites you visit_
The _dom_sensitivity_access_ is an interesting property we can utilize to protect websites from malicious extensions as well as improve overall security on the web applications. The similar approach has been introduced in some research papers [9]. This permission only supported if the website implements it on its DOM. In our Thread Model, we accessed following HTML element from DOM to get the password:
```
<input type="password" id="password" name="password">
```
By employing HTML5 data attributes [10], we define _data-sensitivity_ and set the value one of the access level: low, medium, high. For personal information fields such a credit card, a password can be defined using _data-sensitivity = “high”_ :
```
<input type="password" id="password" name="password" data-sensitivity = "high">
```
Once this implementation is applied the extension will only be able to access the ele- ments that have been allowed by the user. More precisely, _dom_sensitivity_access:_ low will not able to get the value of password because of _data-sensitivity = "high"_ attribute. A user can be notified before installing the extension as the following message:
_Read, Write and access personal data (credit card, password etc.) on the websites you visit_
The _same_origin_ and _cross_origin_ allows the system know access level of HTTP requests. In case the same_origin permission has allow status, it will let an extension to send only HTTP request originated from the same source. The same goes for cross_origin, the system will permit an extension to make unlimited cross-origin HTTP requests. The following message can be presented to the end-users before installation to reveal what they are about to install:
>Send a request on behalf of you to remote services
Going back to the the first table after introducing the new permission system the malicious system will have a hard time to trick a user to install the malicious extension since the following permission needs to be accessed by a user:  
  
| Desired access  | Message for a user  |
|--- |---|
| DOM  | Read and Write the websites you visit  |
| Read sensitive field from the DOM  |  Read, Write and access personal data (credit card, pass- word etc.) on the websites you visit |
| Send cross origin requests  |  Send a request on behalf of you to remote services |

These messages introduces the better detail about an extension and conscious user will not allow such behavior in simple extensions such as a calculator, weather, translator etc.

## Conclusion
Nowadays, the extensions are widely being used by users to increase browser experience. As the browser extensions are rapidly growing so the number of malicious extensions. In this report, we looked at how an extension works, what threats and risks it can bring such as identity and sensitive information stealing, making a user part of the bot-net network and performing DDoS attacks. We also went through what kind of counteracts can be taken to prevent these threats.

References \
1. Browser usage, http://gs.statcounter.com/ \
2. Adrienne Porter Felt, Elizabeth Ha, Serge Egelman, Ariel Haney, Erika Chin, David Wagner.: Android Permissions: User Attention, Comprehension, and Behavior, https://cups.cs.cmu.edu/soups/2012/proceedings/a3_Felt.pdf \
3. Dan Goodin.: Google Chrome extensions with 500,000 downloads found to be malicious,
https://arstechnica.com/information-technology/2018/01/500000-chrome-users-fall-prey-to-malicious-extensions-in-google-web-store/ \
4. Jorge Villalobos: Extension review wait times are about to get much shorter,
https://blog.mozilla.org/addons/2017/09/21/review-wait-times-get-shorter/ \
5. Reddit Firefox: https://www.reddit.com/r/firefox/comments/737kze/min-ing_codes_been_discovered_in_two_reviewed/ \
6. Execution environment: https://developer.chrome.com/extensions/content_scripts#execu-tion-environment \
7. HTTP FLOOD: https://www.incapsula.com/ddos/attack-glossary/http-flood.html \
8. Chrome extensions are mining cryptocurrency now: http://bgr.com/2017/12/29/mining-cryptocurrency-home-pc-chrome-extension/ \
9. Chrome Extensions: Threat Analysis and Countermeasures:
http://dev1.www.isocdev.org/sites/default/files/11_4.pdf \
10. Using data attributes: https://developer.mozilla.org/en-US/docs/Learn/HTML/Howto/Use_data_attributes

<style>
table{
    border-collapse: collapse;
    border-spacing: 0;
    border:2px solid #ff0000;
}

th{
    border:2px solid #000000;
}

td{
    border:1px solid #000000;
}
</style>