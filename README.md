## Installation

```
> npm i -g hexo-cli
> git clone {this-repo} && cd {this-repo}
> npm install
> hexo server
```
## New draft

Prefer to start your post as a draft and then convert it to a post:

```
> hexo new draft your_title
> hexo server --draft --open
```


## New post

```
> hexo new super_new_post
>hexo server --open
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
