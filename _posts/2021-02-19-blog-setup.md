---
title: "My Blog Setup"
date: 2021-02-19
layout: post
tag: setup
---

This is my first post after blog creation.<br/>
Explains how to start with your own blog using jekyll for building,
github pages for deployment and docker for development.

### Setup
Fork [this](https://github.com/madhavtummala/blog) repository. In settings, turn on github pages
and set the url accordingly

### Adding posts
It uses minima jekyll theme, add a new post in the same naming format as the ones already existing. 
You don't have to install anything other than docker. Just run `docker-compose up` and you will have 
a development environment read at [localhost](http://localhost:4000). After you are happy with the look 
and content, push the changes to github, and the live deployment should take effect in some time.

### Archive tab
About Me tab is self-explanatory, in archive tab, instead of simply having all blog posts, I
categorised them by tag. You can look at the code for that in `archive.md`.