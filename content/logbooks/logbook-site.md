---
title: "Logbook - How this site came to be"
date: 2023-06-22T22:14:27+03:00
draft: true
---
If you haven't already checked out the [about site page](/about/about-site), please read it first for some context.

## Attempt 1: Wordpress

Link to [Wordpress](https://wordpress.org/).

When one usually looks to create a website as a complete newbie to the web development space, the first results usually tend to be either website services like Squarespace or Wix, or Wordpress (the open source web content management system, not the hosting service). Squarespace and Wix felt a little bit restrictive in terms of what type of content I could create with it, which made me attempt to work with Wordpress.

I eventually gave up on Wordpress due to the following reasons:
1. I spent way too much time configuring the site, since there are simply an overwhelming number of plugins and options.
2. Wordpress itself isn't very secure, and due to most websites being built on Wordpress, there are many malicious bots out there trying to crack the admin account to gain access to the site. I ended up spending days researching and implementing all sorts of security measures such as 2FA, relocating the admin login URL, lots of hardening plugins, and even setting up the Wordpress SMTP mailing service in order to get emails whenever suspicious activity was spotted. Suffice to say, I felt like I was living in fear of my site being breached, despite there not being any content on the site.
3. When I actually got around to building the website, I discovered the default page builder offered by Wordpress to be very restrictive, and when I went to look for alternative options, I found out that some types of content blocks were simply paywalled. It then became a game of "finding the best page builder", which eventually led to some more analysis paralysis on my end.
4. I eventually realized that Wordpress is sorely overkill for the purpose I needed it to serve. I just wanted to create some content, yet this platform is geared up for creating e-commerce websites or other fancy applications.

After going through the endless cases of analysis paralysis from all the options Wordpress had to offer, it was safe to say that my motivation was completely drained. Until...

## Attempt 2: Ghost

Link to [Ghost](https://ghost.org/).

A couple months later, I watched a [video](https://youtu.be/acBJsjCqgtM) from Ali Abdaal about how to build a website, where one of his recommendations was Ghost. At first glance, it appeared to be a more batteries-included version of Wordpress, which was great to hear as it removed all the hardships I had with Wordpress.

After setting up an instance of Ghost myself and starting to write content, I discovered a few things that really bugged me:
1. There was no way to implement 2FA, and I really did not want my admin account to be secured solely by a password.
2. The page builder felt quite weird. it was not Markdown, but more like a locked-down version of Notion's editor. This annoyed be as it did not really offer the features I wanted for easily getting my ideas in text. There was a markdown block available, but writing in markdown in another editor just felt weird.

It didn't help that back then, I also struggled really badly with getting my thoughts and ideas down in text, meaning that when it came to writing content, I always just blanked out.

## Attempt 3: Hugo

Link to [Hugo](https://gohugo.io).

I caught wind of GitHub Pages from a classmate of mine, which prompted me to look into static site generators further. It is basically perfect security-wise, since static sites means that the entire site is just basic HTML with some very light JavaScript. In other words, there is no database behinds the scenes that could be hacked.