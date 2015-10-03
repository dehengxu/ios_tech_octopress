---
layout: post
title: "Keys for using jekyll on github.com"
date: 2014-08-31 11:48:21 +0800
comments: true
categories: jekyll, blog, github.com
---

##Why choose jekyll?
1. First, it's an static blog system, so it can response huge accessing.
2. You don't need configure your http server, sql server, and blog system, you can host your own blog site only depend `rvm`.
3. If you don't wanna maintance your own server, you can use github's : )

##How to install all of jekyll.
#### Install rvm a ruby version management tool.

```
	
```

#### Install jekyll gem.

```
	gem install jekyll
```

#### Install depended gems

```
	gem install bundle
	
	bundle install
```

All done, you can take rest.

##How to use jekyll with github.com
You can find some tips on this page:  
<https://help.github.com/articles/using-jekyll-with-pages>. 


##Why your blog looks poorly.
Follow this link <http://jekyllrb.com/docs/github-pages/>

> It figure out you should modify _config.yml, filed `base_url` value to `/your_repository_name`, the slash must be there.


##Use your own domain
Add CNAME file to your repostory root dirctory. Writre your domain in one line.

##Final

That's all, just go writing valuable articles.

