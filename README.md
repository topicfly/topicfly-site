## Installation

```
> npm i -g hexo-cli
> git clone {this-repo} && cd {this-repo}
> npm install
> hexo server
```

## New post

```
> hexo new super_new_post
```

Modify `source/_posts/super_new_post.md` front-matter to look something like this:

```
---
title: Super New Post
date: 2018-05-14 21:48:40
tags:
author: Your name
---

<-- the content of your post -->
```

## Deployment

```
> hexo generate && hexo deploy
```
