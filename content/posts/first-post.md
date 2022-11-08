---
title: "First Post: The New Website"
date: 2019-01-27T17:53:22-05:00
draft: false
---
My website used to be static html that I created with a little bit of bootstrap help.  I love bootstrap - not least 
because its creator ([mdo](https://github.com/mdo)) is a fellow Hubber - but the truth is that I don't need everything
that it does.  The truth is that I'm better with markdown than I am with HTML and CSS these days.  So, why not make a 
static website using markdown?  Enter [Hugo](https://gohugo.io/)!

Since I'm a hubber too, I felt obliged to make use of [github pages](https://pages.github.com/) so I googled up [this
documentation](https://gohugo.io/hosting-and-deployment/hosting-on-github/#github-user-or-organization-pages) for help with
getting hugo and githubpages to work together.  In addition, I wrote this little script to start my local development server so that I can get things right before I 
publish it to the repo:

```bash
#!/usr/bin/env bash
set -e

hugo server -D --bind "0.0.0.0" -b "http://compute.ruiz.house:1313/" --disableFastRender
```

`compute.ruiz.house` is my ubuntu compute server here at home.  The `:1313` is the hugo server port.  Since I'm SSH'd 
into `compute` from my Windows machine, I need to bind on `0.0.0.0` so that I can hit `compute` from my browser here.

So my workflow is this: 

1. run ./dev
1. make my changes and preview them
1. run ./deploy.sh

I love that my site is static.  I love that the content is driven by markdown.  I love LOVE that I'm not writing any 
CSS and the site looks acceptable.  The only dynamic content is the [disqus](https://disqus.com/), but it's all 
handled by the excellent [onepress](https://github.com/ijsucceed/onepress) theme.

The only remaining question is whether the comments section will be useful or a [wretched hive of scum and villainy](
https://www.youtube.com/watch?v=Xcb4_QwP6fE).
