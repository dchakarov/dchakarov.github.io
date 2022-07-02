---
title: Drafting away
date: 2022-06-12 00:00:00 Z
layout: post
image: 
description: How to use the Jekyll drafting feature so we don't publish our article
  prematurely.
---

<span class="dropcap">A</span>s [I promised last week](https://dchakarov.com/blog/ruby-on-my-mac/), let's see how we can have drafts in our GitHub pages blog instead of posting live and fixing on the fly.

Turns out, once we have Ruby installed, it's a piece of cake.

Step 1: Install Jekyll:

``` bash
$ gem install jekyll
```

Step 2: Go to your website folder and run:

``` bash
$ bundle install
```

Step 2.1: If you are using Ruby 3+, you need to add webrick manually:

``` bash
$ bundle add webrick
```

Step 3: Run Jekyll with drafts enabled ([as per the instructions](https://jekyllrb.com/docs/posts/#drafts)):

``` bash
$ bundle exec jekyll serve --drafts
```

Step 4: Create a draft post in the `_drafts` folder and make sure the file name doesn't include the date, e.g. `drafting-away.md`

Step 5: Go to http://127.0.0.1:4000 and see your draft post in the list.

<img src="{{ '/assets/img/drafts-screenshot.png' | prepend: site.baseurl }}" style="border-width: 1px; border-color: #b20600; border-style: double;" alt="">

Once you are ready to go live, move the post over to `_posts` and don't forget to add the date to the file name, e.g. `2022-11-11-drafting-away.md`.

Much more professional, innit?

P.S. Ideally, I want to be able to send the draft post to a friend to review before publishing. I am not sure that's possible on GitHub though. Please [ping me on Twitter](https://twitter.com/gimly) if you know how.