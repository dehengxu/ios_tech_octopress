---
layout: post
title: "Jekyll site class available attr"
date: 2015-08-19 11:35:34 +0800
comments: true
published: true
sharing: true
footer: true
categories: Jekyll
---

```
  class Site
    attr_reader   :source, :dest, :config
    attr_accessor :layouts, :posts, :pages, :static_files, :drafts,
                  :exclude, :include, :lsi, :highlighter, :permalink_style,
                  :time, :future, :unpublished, :safe, :plugins, :limit_posts,
                  :show_drafts, :keep_files, :baseurl, :data, :file_read_opts,
                  :gems, :plugin_manager

    attr_accessor :converters, :generators, :reader
    attr_reader   :regenerator, :liquid_renderer

	...
```