---
layout: post
title: "Jekyll configuration fileds"
date: 2015-08-19 11:32:06 +0800
comments: true
published: true
sharing: true
footer: true
categories: Jekyll
---

This is Jekyll configuration source file:

For configurating _config.yml use.

```
    # Default options. Overridden by values in _config.yml.
    # Strings rather than symbols are used for compatibility with YAML.
    DEFAULTS = {
      # Where things are
      'source'        => Dir.pwd,
      'destination'   => File.join(Dir.pwd, '_site'),
      'plugins'       => '_plugins',
      'layouts'       => '_layouts',
      'data_source'   =>  '_data',
      'collections'   => nil,

      # Handling Reading
      'safe'          => false,
      'include'       => ['.htaccess'],
      'exclude'       => [],
      'keep_files'    => ['.git','.svn'],
      'encoding'      => 'utf-8',
      'markdown_ext'  => 'markdown,mkdown,mkdn,mkd,md',
      'full_rebuild'  => false,

      # Filtering Content
      'show_drafts'   => nil,
      'limit_posts'   => 0,
      'future'        => true,           # remove and make true just default
      'unpublished'   => false,

      # Plugins
      'whitelist'     => [],
      'gems'          => [],

      # Conversion
      'markdown'      => 'kramdown',
      'highlighter'   => 'rouge',
      'lsi'           => false,
      'excerpt_separator' => "\n\n",

      # Serving
      'detach'        => false,          # default to not detaching the server
      'port'          => '4000',
      'host'          => '127.0.0.1',
      'baseurl'       => '',

      # Output Configuration
      'permalink'     => 'date',
      'paginate_path' => '/page:num',
      'timezone'      => nil,           # use the local timezone

      'quiet'         => false,
      'verbose'       => false,
      'defaults'      => [],

      'rdiscount' => {
        'extensions' => []
      },

      'redcarpet' => {
        'extensions' => []
      },

      'kramdown' => {
        'auto_ids'       => true,
        'footnote_nr'    => 1,
        'entity_output'  => 'as_char',
        'toc_levels'     => '1..6',
        'smart_quotes'   => 'lsquo,rsquo,ldquo,rdquo',
        'enable_coderay' => false,

        'coderay' => {
          'coderay_wrap'              => 'div',
          'coderay_line_numbers'      => 'inline',
          'coderay_line_number_start' => 1,
          'coderay_tab_width'         => 4,
          'coderay_bold_every'        => 10,
          'coderay_css'               => 'style'
        }
      }
    }

```
