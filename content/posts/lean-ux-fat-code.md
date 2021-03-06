+++
title = "Lean UX, large code"
author = ["Benoit Joly"]
date = 2020-12-23T20:49:46-05:00
lastmod = 2020-12-23T21:32:03-05:00
tags = ["100DaysToOffload", "blogging-setup"]
categories = ["tech"]
draft = false
+++

## It all started that day {#it-all-started-that-day}

Two weekends ago, bored, I was looking for something to do.

I remembered posts on fosstodon about some website size ranking called "[The 512kb club](https://512kb.club)".

Last time I worked on my blog setup, I tried porting it to Emacs org publish.

Following the instructions on 512KB site, I ran my site through GTMetrix.

To my surprise, I discovered my simple blog index page was roughly 600KB.

What!!! 600KB, that is 600000 characters, just for an index page that has a listing of my 15 or so blog posts. 600KB for 15 posts, this is an average of 40KB per titles. Even more insane!

The whole point of the 512KB is the fight bloat, promote real lean website. My blog was far from being lean.

I was motivated, I had a plan: ****make my blog leaner****


## What is the root cause {#what-is-the-root-cause}

I never optimize without data, so a quick look at GTMetrix told me the problem: the CSS, and fonts were the majority of that 600KB.

I got these from the theme I imported during my initial setup (on my 1st post). It was based of a Hugo theme posted on GitHub made to look like motherfuckingwebsite, but I had to admit, like many site with lean UX, under the hood it was not lean.


## In search of a real lean CSS {#in-search-of-a-real-lean-css}

I could have just modified the CSS, but with that much dead code, there is just no point of doing so.

I'm also a practical coder, I'm not the only one with the idea of a lean CSS. After couple of searches, I found [new.css](https://newcss.net), a CSS that brings lean UX in a relatively small package (4.5KB).

The best way to succeed would be to generate this site with plain semantic HTML, trying to avoid DIVs and other general container. Possibly sticking to HTML5.


## The wrong approach {#the-wrong-approach}

I decided to continue porting my blog to Emacs org publish. The idea being it would remove Hugo from the equation since I already use Emacs org-mode for this blog.

After couple of hours of work, I finally succeeded: my blog was rendering like before, with the same CSS as before, and I even got the RSS properly rendered.

Next would be the CSS, so I decided to open up the produced HTML to get ready for some CSS work. The result? I was way further from my original goal. The generated page was just full of divs, almost unreadable.

I tried to fix this, but the more I fixed issues in the generator (I had to customize many Emacs functions in lisp), the further I was to keep things simple.

I just had to admit, that the only thing productive about this effort was the learning I gained :P


## Second try {#second-try}

I had to find a simpler solution. Could I make this work with my Hugo setup? I discovered my CSS was bloat, but the generated HTML was actually really good and lean.

30 minutes is all it took to replace the existing theme with a simpler one that imports new.css + the font suggested in the new.css site.

Satisfied with the look, the simpler CSS, I decided to push it to my server.


## New data point {#new-data-point}

Time for a new size check. Result: 8.5KB

Success!!!


## Time to submit my site to 512kb.club {#time-to-submit-my-site-to-512kb-dot-club}

Satisfied with this new improvement, I submitted my site to 512kb club, happy to be somewhere between 13 and 15 position (many people are submitting sites).


## Can I do better? {#can-i-do-better}

What can I do to improve?

I could always try to reduce that 4.5KB CSS, but this would mean spending time learning this code, or writing my own. Not enough time for that yet.

What else can I learn from the GTMetrix site? ohh, a 404 on favicon.ico cost roughly 1KB

What If I could create the smallest possible file, to prevent my 404 page to render. This could work.

I did not know that you can create a favicon inlined in the HEAD section of a page.

```html
<link href="data:image/x-icon;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQEAYAAABPYyMiAAAABmJLR0T///////8JWPfcAAAACXBIWXMAAABIAAAASABGyWs+AAAAF0lEQVRIx2NgGAWjYBSMglEwCkbBSAcACBAAAeaR9cIAAAAASUVORK5CYII=" rel="icon" type="image/x-icon" />
```

his is all you need to generate a small blank favicon.

Metrics after this improvement? 8KB.

his is 75 times smaller than the original size. This is 0.53 KB per post instead of ~40 KB per post.

At the time of writing this post, this puts me at the top 12 smallest sites on the 512kb club.

<a id="org6821e46"></a>

{{< figure src="https://512kb.club/images/green-team.svg" alt="green banner from 512kb club for sites < 100kb" link="https://512kb.club" >}}


## Before / After {#before-after}

{{< figure src="/ox-hugo/lean-ux-fat-code-before.png" alt="picture showing the original version with large CSS" caption="Figure 1: Before" >}}

{{< figure src="/ox-hugo/lean-ux-fat-code-after.png" alt="picture showing after new css. minimal changes visible compared to before" caption="Figure 2: After" >}}


## Thoughts {#thoughts}

Creating simple solutions is not necessarily easy, but is fun and rewarding.

At the same time, you can do a lot with not much time.

I wish to inspire peers (especially in the IT industry). The web is bloat, but it's not better in backend code.

****Note****: as a side note, I still motivate myself with 100DaysToOffload. It will take some time to reach my goal (17 posts so far), but I will get there sometime in the next few months, or years.

---

_This is day 17 of my #100DaysToOffload. You can read more about the challenge here: <https://100daystooffload.com>._

<!--more-->
