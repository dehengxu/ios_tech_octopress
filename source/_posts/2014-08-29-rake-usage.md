---
layout: post
title: "Rake usage"
date: 2014-08-29 14:42:06 +0800
comments: true
categories: Ruby
---
## What is rake

Rake is a ruby build tool likes ant, make...

In ant, it composed with `taget`s, but in rake it composed with `task`

##Add your task

### Normal task
Define your own task  

{% highlight ruby %}
task :taskName do
	#Your task code in ruby syntax.
end
{% endhighlight %}

### Task with params.

```
task :taskName, :param1 do |t, params|
	#Here params is an hash table object.
end
```


> <http://jasonseifer.com/2010/04/06/rake-tutorial>