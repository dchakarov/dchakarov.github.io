---
title: new site, who dis?
date: 2018-06-14 00:00:00 Z
layout: post
---

<span class="dropcap">A</span>fter a successful redesign of our company website seenit.io I felt inspired to redesign and rearrange my personal web presence. I made a long list of all my websites and all the most important places I had some kind of presence on (i.e. StackOverflow, GitHub, LinkedIn, Twitter) and started planning a strategy. I decided that, since most of these places invite you to link to your personal website, it would be smart to start with that, so I can have something I like to link to.

There were two aspects I had to make a decision on - tech and story.

Tech was the easier one. I decided to go with Jekyll as this was the trendy new thing (at least here in London). As an ex-backend developer going with a static website was a bit weird (to put it mildly). But one does not go against fashion. Also, trying new things.

I already had a domain (quite a few, actually - but more on that later). After some research (thx @LeeroyDing) I decided to give GitHub Pages a try. It was free and easy (for developers). You go on [https://pages.github.com](https://pages.github.com) and follow the steps. By far the hardest step was configuring my domain to point to my new page - having to deal with my domain registrar is never quite as straightforward as it should've been.

Next step was to deploy an actual page and for that I followed the next guide - <img src="{{ '/assets/img/image1.png' | prepend: site.baseurl }}" alt="">  [https://jekyllrb.com/docs/quickstart/](https://jekyllrb.com/docs/quickstart/). Even for someone that had never worked with _bundler_ and had very limited knowledge about what _gem_ is, it was very easy to make it working. Keep in mind that not all Jekyll themes are supported on GitHub Pages, so better [check here](https://pages.github.com/themes/) before spending too much time tweaking a theme which is not supported. It's important to note that a theme might still be supported even if not listed on that list. So if you can't find a suitable enough theme on that list, [select one from here instead](https://jekyllrb.com/docs/themes/) and upload it to GitHub to test if it'd work. Chances are it would.

With all that setup behind me I spent the next few days enjoying my new website. Then, out of a sudden, I received an email from Google, telling me a page from one of my subdomains had been de-listed from their index because it was violating someone's copyright. This was scary. Moreover, I didn't have any subdomains active on my domain. None that I knew about anyway. I clicked the link and - surprise, surprise - a book (PDF) file appeared.

<img src="{{ '/assets/img/image2.jpg' | prepend: site.baseurl }}" alt="">

I triple checked my newly updated website and couldn't find any mentions of this or any other PDF file. Clueless about what to do next I contacted GitHub and they kindly informed me I had made a mistake configuring my domain forwarding (DNS ALIAS rules, for those that know what that is) in a way that anyone could host something using any subdomain of my primary domain. I went on the domain registrar site one more time and disabled the star catch-all subdomain which did the job. I am still not convinced that wasn't partly GitHub's fault but since they were very helpful in resolving the issue, I am not too mad about it.

And that's it. The website is live. You are experiencing it. What do you think?