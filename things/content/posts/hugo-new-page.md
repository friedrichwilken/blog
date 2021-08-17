--- 
author: Friedrich Wilken
title: hugo new post
date: 2021-08-17T10:52:59+08:00
description: how to create a new post
tags: [ markdown, hugo, ]
categories: [ how-to, tech ]
series: [hugo]
toc: false 
---

First, `cd` into your page's repo.
```bash
cd ~/friedrich/go/src/github.com/friedrichwilken/blog/things/
```
Now, lets create a new post. I like to have a directory only for posts, so:
```bash
hugo new posts/hugo-new-page.md
```
Your new markdown file will contain some boilderplate code:
```markdown
  ---
  title: "Hugo New Post"
  date: 2021-08-17T23:42:37+02:00
  draft: true
  ---
```
It's likely that your theme supports more advanced fields like tags, categories or series. You might find these in the docs or you could just steal them from the theme's example repo ([here is mine](https://h.xjj.pub/)):
```markdown
  ---
  author: Friedrich Wilken
  title: hugo new post
  date: 2021-08-17T10:52:59+08:00
  description: how to create a new post
  tags: [ markdown, hugo, ]
  categories: [ how-to, tech ]
  series: [hugo]
  toc: false 
  draft: false
  ---
```
Now might be a good moment to start a local server to get a preview of your new post with some nice hot reloading ❤️.
```bash
hugo server
```
Finally, just write some markdown in your lovely new post.