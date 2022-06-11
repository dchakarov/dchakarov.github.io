---
title: Ruby on (my) M1 Mac
date: 2022-06-11 00:00:00 Z
layout: post
image: ruby.jpg
description: How to install Ruby on my mac.
---

<span class="dropcap">O</span>nce again I went deep into a rabbit hole. It started with writing an article about Apple's TabularData framework (coming soon) and deciding to preview it. I am using Jekyll for this blog and it's hosted on GitHub Pages. From what I read, in order to preview it, I needed to enable drafts. For that to happen, I needed to run Jekyll locally. This is where the fun started.

Installing Jekyll locally is a piece of cake, [they say](https://jekyllrb.com/docs/installation/macos/). You _just_ need Ruby. To manage that, you need a _ruby version manager_, and they recommend `chruby`. That's all good but I use [Fish shell](https://fishshell.com) and that's where things get a tad more complicated.

I tried with `brew install chruby ruby-install`, `brew install chruby-fish`, `ruby-install ruby-3.1.2`, `echo 'ruby-3.1.2' >> .ruby-version` + a bunch more commands from various sources. And, of course, I restarted my Mac a bunch of times _just to be sure_. It didn't work. Then I stumbled upon [Ruby on Mac](https://www.rubyonmac.dev). I felt excited at first but then decided I don't want to spend $49 just to install Ruby. I am not even a ruby developer.

Then I remembered that we have a "How to get your Mac up and running" guide in my team's wiki. And part of the guide is on how to install Ruby because - doh - we are iOS developers, after all.[^1]

Anywho, here's what I did, and what I wanna have written down for, you know, _next time_.

0. Uninstall chruby, chruby-fish, delete the .rubies folder, etc. (skip that on fresh install)
1. `brew install gnupg`
2. `ln -s /opt/homebrew/bin/gpg /opt/homebrew/bin/gpg2`
2. `gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB`
3. `curl -sSL https://get.rvm.io | bash -s stable`
4. `curl -L --create-dirs -o ~/.config/fish/functions/rvm.fish https://raw.github.com/lunks/fish-nuggets/master/functions/rvm.fish`
5. `echo "rvm default" >> ~/.config/fish/config.fish`
6. (optional?) `rvm osx-ssl-certs update all`
7. `rvm install 3.1.2` (obviously, update that)
8. `rvm list`
9. `ruby -v`

And then install Jekyll (even if you don't need Jekyll, you might want to install openssl)
1. `brew install openssl`
2. `set -gx LDFLAGS "-L/opt/homebrew/opt/openssl@3/lib"`
3. `set -gx CPPFLAGS "-I/opt/homebrew/opt/openssl@3/include"`
4. `set -gx PKG_CONFIG_PATH "/opt/homebrew/opt/openssl@3/lib/pkgconfig"`
5. `gem install jekyll`

Voila.

Well, not that fast.

$ `jekyll`

*Could not find proper version of jekyll (4.1.1) in any of the sources*

$ `bundle install`

*listen-3.2.1 requires ruby version >= 2.2.7, ~> 2.2, which is incompatible with the current version, ruby 3.1.2p20*

It took me embarrassingly long time at this moment to figure out that the issue was my existing `Gemfile.lock` with an older version of jekyll in it.

$ `rm Gemfile.lock`

$ `bundle install`

$ `jekyll`

Finally!

Stay tuned for the "How to draft an article on Jekyll."

[^1]: I blame Cocoapods.