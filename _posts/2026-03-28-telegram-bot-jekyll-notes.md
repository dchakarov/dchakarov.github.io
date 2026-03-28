---
title: How I Built a Telegram Bot to Post Notes to My Jekyll Blog
date: 2026-03-28 08:00:00 Z
layout: post
image: notes.jpg
description: A step-by-step guide to building a Telegram bot that publishes quick notes to a GitHub Pages Jekyll blog, with link previews and zero extra services.
---

<span class="dropcap">I</span>'ve been wanting to add a "micro-blog" to my site for a while -- a place for quick thoughts, links I've found interesting, or a video worth sharing. Not a full article, just a line or two with a link. The kind of thing you'd share in a group chat.

Posting on my site isn’t that hard right now. I duplicate the last post (as an MD file), rename it, open it in Panda, and start writing. Once I’m done, I open Fork and push my changes to GitHub. A few seconds later, the article is live.

That’s all good when I have my laptop with me. If I only have my phone, I would have to do all that in the GitHub web interface or develop a complicated workflow in order to use an app on the phone. Another problem is styling. The current blog theme works for long technical articles. To a lesser extent, it also works for a short story or a review. It doesn’t work for link posts or for hot takes.

Then my friend Costa wrote about [how he built something similar](https://www.costafotiadis.com/things/) using a Telegram bot, and I decided to build my own version. His setup uses Railway to host the bot and appends entries to a single page. Mine is slightly different: each note becomes its own Jekyll post, and the bot runs locally on my Mac. That way, I avoid having to sign up for a new service.

## The Architecture
The setup is simple:
1. I send a message to my Telegram bot (from my phone, desktop, wherever)
2. A Python script running on my Mac picks it up
3. The script creates a Jekyll post file via the GitHub Contents API
4. GitHub Pages rebuilds the site automatically

No servers, no containers, no CI pipelines. The bot talks directly to GitHub's REST API to create files in my `_posts/` directory, and GitHub Pages does the rest.

## The Bot
The bot is about 150 lines of Python code, built on two frameworks: a Python wrapper for the Telegram API called [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) and [httpx](https://www.python-httpx.org/). It handles three types of messages (for now):
- **Just a link** - fetches the page's Open Graph metadata (title, description, image) and creates a note with a link preview card
- **A link + my thoughts** - As above, but use my notes as the post body
- **Plain text** - saved as a quick thought, no link involved

### Creating Posts via the GitHub API
The clever bit (borrowed from Costa's approach) is that the bot never touches a local git repo. It uses the [GitHub Contents API](https://docs.github.com/en/rest/repos/contents) to create files directly:
```python
async def gh_create_file(client, path, content, message):
    url = f"https://api.github.com/repos/{GITHUB_REPO}/contents/{path}"
    payload = {
        "message": message,
        "content": base64.b64encode(content.encode("utf-8")).decode("utf-8"),
    }
    r = await client.put(url, headers=GH_HEADERS, json=payload, timeout=15)
    r.raise_for_status()
```

A single `PUT` request creates the file and commits it. GitHub Pages picks up the new commit and rebuilds the site within a couple of minutes.

### Fetching Link Previews
When you share a link in Telegram, WhatsApp, or Slack, you get a nice preview card with the page's title, description, and image. I wanted the same thing on my blog.

The bot fetches Open Graph metadata from the URL:
```python
async def fetch_og_metadata(client, url):
    r = await client.get(url, timeout=10, follow_redirects=True,
        headers={"User-Agent": "Mozilla/5.0 ..."})
    page = r.text
    return {
        "title": _get_meta(page, ["og:title", "twitter:title"]),
        "description": _get_meta(page, ["og:description", "twitter:description"]),
        "image": _get_meta(page, ["og:image", "twitter:image"]),
    }
```

It then generates an HTML preview card that gets embedded in the post. One gotcha: sites behind Cloudflare (like Medium) return a "Just a moment..." challenge page instead of real content. The bot detects these bad titles and falls back to extracting a human-readable title from the URL path.

### Security
Since the bot publishes directly to my site, I needed some restrictions. Posts are only allowed from my Telegram user ID. And all the keys to connect to GitHub and Telegram are stored on my Mac.

```python
ALLOWED_USER_ID = int(os.environ["ALLOWED_USER_ID"])

async def handle_message(update, context):
    if update.effective_user.id != ALLOWED_USER_ID:
        await update.message.reply_text("Not authorised.")
        return
    # ... create the post
```

## The Jekyll Side
I needed a few tweaks on my blog templates to make this work.
### A Separate Layout for Notes
Regular blog posts use my `post.html` layout, which shows a featured image, read time estimate, and prev/next navigation. That's too heavy for a one-liner. I created a minimal `note.html` layout:

```html
{% raw %}---
layout: default
---

<div class="post note">
  <p class="meta">{{ page.date | date: '%B %d, %Y' }} &middot; Note</p>

  {{ content }}
</div>{% endraw %}
```

No title heading (the body is the content), no read time, no post navigation. Just the date and the text.

### A Notes Tab
I added a `notes.md` page that lists only posts in the `notes` category, and added it to my site navigation. The filter is a simple Liquid condition:
```html
{% raw %}{% for post in site.posts %}
  {% if post.categories contains 'notes' %}
    <!-- show the note -->
  {% endif %}
{% endfor %}{% endraw %}
```

Notes also appear on the homepage alongside regular posts. The homepage template detects notes and shows them differently -- just the date and content text, without the usual title-as-link treatment.

### Link Preview Cards
The preview cards are styled with CSS to look like the link previews you see in messaging apps -- a border, rounded corners, the page's image on top, and the title/description/domain below. The key CSS:
```scss
.link-preview {
  display: block;
  border: 1px solid darken(white, 15%);
  border-radius: 8px;
  overflow: hidden;
  margin: 1.5rem 0;

  img {
    width: 100%;
    max-height: 300px;
    object-fit: cover;
    display: block;
  }
}
```

One thing to watch out for: Jekyll uses kramdown for Markdown processing, and kramdown can mangle raw HTML if you're not careful. The fix is to wrap the HTML card in kramdown's `{::nomarkdown}...{:/nomarkdown}` tags, which tells it to pass the HTML through untouched.

## Running It Locally
Costa uses [Railway](https://railway.com/) to host his bot. I didn't want to sign up for another service, so I run mine locally on my Mac using `launchd`.

A `.plist` file tells launchd to start the bot on login and restart it if it crashes:
```xml
<dict>
    <key>Label</key>
    <string>com.dchakarov.notebot</string>
    <key>ProgramArguments</key>
    <array>
        <string>/path/to/my-bot/.venv/bin/python</string>
        <string>/path/to/my-bot/bot.py</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
```

Install it with:
```bash
cp com.dchakarov.notebot.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/com.dchakarov.notebot.plist
```

The bot runs as long as my Mac is on. Since Telegram uses polling, it works regardless of whether I send the message from my phone, tablet, or desktop. Messages sent while the Mac is off simply queue up in Telegram and get processed the next time the bot starts. And yes, my Mac is a laptop, so my thoughts don’t go live immediately. So what?

## Setting It Up Yourself
If you want to build your own, here's what you need:

### Prerequisites
1. A Jekyll blog on GitHub Pages
2. Python 3.9+
3. A Telegram account

### Steps
**0. Think of a good name for your bot.** Don’t be lame.

**1. Create a Telegram bot.** Message [@BotFather](https://t.me/BotFather) on Telegram, send `/newbot`, and follow the prompts. Copy the bot token.

**2. Get your Telegram user ID.** Message [@userinfobot](https://t.me/userinfobot) on Telegram. It replies with your numeric ID.

**3. Create a GitHub personal access token.** Go to [GitHub Settings > Developer settings > Fine-grained tokens](https://github.com/settings/tokens?type=beta). Create a token scoped to your blog repository with "Contents: Read and write" permission.

**4. Set up the bot.**
```bash
mkdir cool-bot && cd cool-bot
python3 -m venv .venv
.venv/bin/pip install python-telegram-bot httpx python-dotenv
```

**5. Create a `.env` file** with your credentials:
```
TELEGRAM_BOT_TOKEN=your-token-here
GITHUB_TOKEN=your-github-pat
GITHUB_REPO=yourusername/yourusername.github.io
ALLOWED_USER_ID=your-telegram-id
SITE_URL=https://yoursite.com
```

**6. Add the bot script** - you can find the full source code here: [Telegram notes bot](https://github.com/dchakarov/telegram-notes-bot). Feel free to contribute!

**7. Test it.** Run `.venv/bin/python bot.py`, send a test message to your bot, and check your GitHub repo for the new file. Since one of the features of this setup is that posts get published immediately, be ready to delete your test posts from GitHub.

**8. Set up launchd** to run it automatically once you’re happy (see the section above).

## What I Learned
I didn’t learn Python, that’s for sure. With Costa’s article and code on one side and Claude on the other, implementing this setup was a fun journey. Testing something that publishes live is challenging, so I had to pull locally and delete the test posts rather quickly on each iteration.
There are still a few wrinkles to fix, and a few that will never get fixed, but overall, I am quite pleased with the result. Expect more frequent, shorter posts in the near future.

Check out the [Notes](/notes) section to see it in action.

Hero image by <a href="https://unsplash.com/@dtravisphd?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">David Travis</a> on <a href="https://unsplash.com/photos/brown-fountain-pen-on-notebook-5bYxXawHOQg?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>