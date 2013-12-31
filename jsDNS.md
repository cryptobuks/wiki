# Javascript DNS
JavaScript DNS is a client-side/browser DNS resolver which complies with the Namecoin [[domain name specification]].  This is the specification outlining the expected behavior of a jsDNS and any related APIs.

Note that this poorly written specification is written according to [Joel Spolsky's excellent specification specification](http://www.joelonsoftware.com/articles/fog0000000035.html).  Bad jokes, painful puns, sloppy grammar, and revisable facts (i.e. revisions to the spec) lay ahead.

# Why Bother
Let's say you want to bridge the the ICANN and Namecoin DNS'.  The basic approach is to replace a Namecoin [Top Level Domain](http://en.wikipedia.org/wiki/Top-level_domain) (i.e. `.bit`) with an ICANN Second Level Domain (i.e. `.bit.pe`).  Previously, you had to choose one of three backends:

1. Proxy the content.
2. Mirror the content.
3. Mirror Nameservers/URL forwarding  (basically the same thing legally: linking to content).

There are many, many problems with these choices.  Here are some of them:

* They either link to or host content that will piss off [idiotic judges](http://en.wikipedia.org/wiki/Bank_Julius_Baer_vs._WikiLeaks), equally idiotic [politicians](http://wikileaks.org/tpp/#QQC12), and [evil lawyers](http://www.chillingeffects.org/domain/faq.cgi#QID226).
* Web-proxies are crazy insecure: both the client _and_ the origin server have trust the proxy to decrypt and re-encrypt everything.  It's not _like_ a man-in-the-middle-attack, it _is_ a man-in-the-middle attack.
* They all scale poorly: for every request on the .bit namespace a single charity-driven website has to respond.  With a jsDNS, you can have a **single static web-page** which [scales at log(n)](https://docs.google.com/spreadsheet/ccc?key=0Am9LbxJR3-Q-dFQ3c2lJa1BVejdrT2JnZjBRWm83YlE&usp=sharing) (aka [webscale](http://www.mongodb-is-web-scale.com/)!!!!!).

So why not just install Namecoin?  Well, telling people to install software/change a system setting hasn't worked very well for [AlterNic](http://en.wikipedia.org/wiki/AlterNIC), [OpenNIC](http://www.opennicproject.org/), or any [other alternative DNS system](http://en.wikipedia.org/wiki/Alternative_DNS_root) to have popped in the past ~20 years.

Not impressed with my bullet points?  Check out this biased, out-of-context feature matrix!

| Backend                | Domain Name System Resolver                                   | End-user requirements                 | Linkable by ICANN sites | Absolute .alt URL   | Direct Https | Verifiable Privacy  | Scaling   | Additional .alt website config         | DMCA Liability   | Domain Seizure Reistantance | SOPA Passive Censorship Resistance | Chinese National Firewall Censorship Resistance |
| :--------------------  | :-----------------------------------------------------------: | :-----------------------------------: | :---------------------: | :-----------------: | :----------: | :-----------------: | :-------: | :------------------------------------: | :--------------: | :-------------------------: | :--------------------------------: | :---------------------------------------------: |
| Custom DNS/User Config | DNS Servers                                                   | Sys Config                            | No                      | Full                | Full         | No                  | Log       | None                                   | Linking          | Full                        | Partial                            | Minimal
| Proxy                  | Webhost                                                       | None                                  | Yes                     | Partial-Full        | None         | No                  | Linear    | None                                   | Linking          | Partial                     | Minimal                            | Minimal
| Mirror/NS              | Server                                                        | None                                  | Yes                     | None                | Full         | Yes                 | Sub-Linear| NS change, CORS                        | Linking, hosting | Partial                     | Minimal                            | Minimal
| jsDNS Best-Effort      | Client storage, WebHook sourced from social network(s) or DHT | Onetime Social Network Log-In or none | Yes                     | None                | Full         | Yes                 | Constant  | Dedicated IP or ICANN URL              | None             | Full                        | Minimal-Full                       | Minimal
| Cooperative jsDNS      | Client storage, WebHook sourced from social network(s) or DHT | Onetime Social Network Log-In or none | Yes                     | Full                | Full         | No (maybe)          | Constant  | Dedicated IP or ICANN URL + JS include | None             | Full                        | Minimal-Full                       | Minimal

In designing products, it helps to imagine a few real life stories of how actual (stereotypical) people would use them. We'll look at two scenarios.

Scenario 1: Mike.
Mike is a busy executive. He is the president of a large, important company that makes dynamite-based products for children which are sold through national chains such as Toys 'R' Us. During the course of a typical day, he has many meetings with many very important people. Sometimes a man comes over from the bank to harass him for not paying the interest that was due three months ago on his line of credit. Sometimes another man comes from another bank trying to get him to sign up for another line of credit. Sometimes his venture capitalists (the nice people who gave Mike the money to start his business) visit him to complain that he is earning too much money. "A bonfire!" they demand. "Wall Street likes to see a bonfire!"

These visitors are very upset if Mike has previously promised to meet with them at a certain time, but when that time comes around, Mike is nowhere to be found. This happens because Mike doesn't know what time it is. At his secretary's recommendation, Mike signs up for a WhatTimeIsIt.com account. Now, whenever Mike is wondering about the time, he simply logs onto WhatTimeIsIt.com, enters his username and password, and finds out the current time. He visits the site several times during the day: to find out when it's time for lunch, to check if he's late for the next meeting, etc. Towards the end of the day, in fact, from about 3:00 p.m. onwards, he checks the site increasingly frequently to see when it's time to go home. By 4:45 he's basically just hitting "Refresh" again and again.

Scenario 2: Cindy.
Cindy is a teenager in high school. She goes to a pretty pathetic public high school, and she's pretty smart, so when she gets home at 2:00 pm, it only takes her about 7 minutes (on average) to do her Algebra homework. None of her other teachers even bother to give her homework. Her baby brother (half brother) is vegged out in front of the only TV set watching Teletubbies, so she spends the afternoon (from 2:07 until about 6:30, when her new mommy serves dinner) surfing the net and chatting with her friends on AOL. She's always looking for exciting new web sites. As a result of typing "What Time Is It?" randomly into a search engine (by mistake, she meant to ask one of her friends using Instant Messenger) she gets to WhatTimeIsIt.com, and sets up a new account. She chooses a user name and "RyanPhillipe" as her password, selects her time zone, and voila -- finds out what time it is.

Non Goals

This version will not support the following features:

multiple time zones for one member. All members are assumed to be in the same time zone.
changing passwords.
appointments.
WhatTimeIsIt.com Flowchart

We'll have time later to go into mind-numbing detail, but for now, let's look at a quick flowchart of the service so you get the big picture. This flowchart is not complete, but it does give you the right idea for the "storyboard" of using WhatTimeIsIt.com:

[Image]

Screen by Screen Specification

WhatTimeIsIt.com consists of quite a few different screens. Most screens will follow a standard format, with a look and feel to be designed in the future by a graphic designer. This document is more concerned with the functionality and the interaction design, not the exact look and layout.

All screens are created in HTML. (The single exception is the Splash Screen, which is created using Macromedia Shockwave).

Each screen in WhatTimeIsIt.com is known by a canonical name which will always appear, in this document, with an underline, so you know we're referring to a screen by name, for example, Home Page.

Splash Screen

An annoying, gratuitous Shockwave animation that plays stupid music and drives everyone crazy. Splash Screen will be commissioned by a high-paid graphics animation boutique in a loft in Soho from people who bring their dogs to work, wear found objects safety-pinned to their ears, and go to Starbucks four times before lunch.

After the animation has played for about 10 seconds, a link that says "SKIP THIS" will fade into view in the bottom right corner. To avoid people seeing this and clicking on it, SKIP THIS will be so far down and to the right that most people won't see it. It should be at least 800 pixels from the left border of the animation and 600 pixels from the top.

Clicking on SKIP THIS goes to Home Page. When the animation is complete, it will redirect the browser to Home Page automatically.

Open Issue
If Marketing allows, we should deposit a cookie on the user's computer if they click SKIP THIS which will cause the animation to always be skipped in the future. Frequent visitors should not have to see the animation more than once. I talked to Jim in Marketing about this and he's going to take point in convening a committee of Sales, Marketing, and PR to discuss.

Home Page

Displayed when the Shockwave animation is complete, the Home Page serves three purposes:

Allow people to learn about the service and consider whether they want to sign up
Allow members who have already signed up to log on
Allow people who want to sign up to create an account.
The Home Page looks like this:

WhatTimeIsIt.com
Welcome to WhatTimeIsIt.com, the service that tells you exactly what time it is!
(Your results may vary. We are not responsible for delays in transmission or on your computer which could cause the actual time to be a bit later than the time displayed. This service is provided as-is and is merely for entertainment purposes, not for accurate time-keeping. Do not stick WhatTimeIsIt.com in your ear or use it to clean your ear.)

WhatTimeIsIt.com is easy and fun. If you're not a member, sign up today and start finding out what time it is!

Already a member? Click here to log on!
Not a member yet? Don't worry - membership is free! Yes, that's right, FREE! Just click here to sign up, and within minutes you'll be able to find out what time it is!

Privacy Notice | About Us | Jobs | Contact Us
About WhatTimeIsIt.com

On this, and on all screens, clicking on the WhatTimeIsIt.com logo in the top left corner goes back to Home Page.

Technical Note
Because of the high similarity between the various screens, some system of includes should be used on the server so that if the name of the service changes, or if we can't purchase the domain name we want, we'll be able to change all the screens in one place. I suggest Vignette Story Server. Sure, it's overkill. Sure, it costs $200,000. But it's a heck of a lot easier than using server-side includes!

Clicking on the link that says "click here to log on" goes to Log In Form. Clicking on the link that says "click here to sign up" goes to Sign Up Form. The other five links display pages with static text to be provided by management, which are beyond the scope of this specification. They will not have to change very often.

Log In Form

The Log In Form is used by current members to log into their accounts in order to find out the current time. It looks like this:

WhatTimeIsIt.com
Please enter your email address:


Enter your password: 



Forgot your password? Just enter your email address and we'll email it to you.

Not a member yet? Don't worry - membership is free! Yes, that's right, FREE! Just click here to sign up, and within minutes you'll be able to find out what time it is!
Privacy Notice | About Us | Jobs | Contact Us
About WhatTimeIsIt.com

The right side of the screen behaves the same way as described previously under Home Page.

The email box allows for up to 60 characters to be typed. The password box allows for up to 12 characters to be typed. To disguise them and prevent hacking, as the user types in the password box, asterisks (*) will appear instead of the characters that they type.

Technical Note
This is accomplished using <INPUT TYPE=PASSWORD>

When the user clicks Log In, the following checks are performed on the server:

If the email address was provided, but it could not be a real email address because it is not formatted correctly (e.g. there is no @ sign or it contains characters that are not permitted in email addresses by RFC-822), the server returns another page that looks just like Log In Form, only this time, a red error message is inserted above the address box, saying "The email address you provided is not valid. Please double check it." Although this text is in red, the text "Please enter your email address" still appears in black. The incorrect email address that the user originally typed will now be pre-populated in the edit box.

If the email address was provided, but it does not correspond to a registered member, the server returns another page that looks just like Log In Form, only this time, a red error message is inserted above the address box, saying "The email address you provided is not a member. Please double check it. To become a member, click on the link on the right side of the screen." Although this text is in red, the text "Please enter your email address" still appears in black. The incorrect email address that the user originally typed will now be pre-populated in the edit box. [ Question to developers. Can we use JavaScript in this case so that if the user then clicks on the link to become a member, we automatically pre-populate the email address on the sign up form? ]

If the email address was provided, and it does correspond to a registered member, but no password was typed at all, we send an email to that address containing the password. The subject of the email is "Your WhatTimeIsIt.com membership". The email is in plain text. The exact wording of this email is still being debated hotly by the board of directors and will be provided sometime before shipping. [ Developers: for now I suggest using a nasty word. That will light a fire under Chucks' seat. ]

If the email address was provided, and it does correspond to a registered member, and a password was provided, but the password is incorrect, the server returns another page that looks just like Log In Form, only this time, a red error message is inserted above the password box, saying "The password you provided is not valid. Please double check it. Remember, passwords are case-sensitive." If the password typed does not contain any lower case alphabetic characters, we add this text to the message: "Perhaps you've accidentally turned on CAPS LOCK?" Whenever the password is incorrect, the Log In Form comes back with the password box clear.

If the email address and password are OK, jump straight to Display Time.

Open Issue
Need to decide about JavaScript in case #2

Open Issue
Need wording for password email from CEO