--- 
layout: post
title: "Blogging - Curious things with Pretzel."
author: "Author"
comments: true
tags : [Blogging, Pretzel, Liquid]
---

Pretzel blogging turned up some very curious head-scratchers. 

#### Paginator
Couldn't get paginator to work. Well, it works now. Turns out you have to add 

{% highlight yaml %}
    paginate:   3
{% endhighlight %}

at the top of your `index.html` in the yml frontmatter.

#### Escaping Liquid
I couldn't escape the characters `{{"{{"}}` or the characters `{{"{%"}}` in any fashion. 

You can, if you put it like this `{{ "{{ \"\" }}" }}`.

#### Landing page to show list only.
The landing page was updated to show list of posts. Change is in `index.html`. 

Comment out the following line `{{ "{{" }} post.content }}`.

#### Css font-size reduction
The basic font size was too large. Changes in `hyde.css`. `font-size` reduce by `2px`. The section below is the relevant section. All other fonts are in `em` with relation to these two. So this is a across all devices and form-factors thing. 

{% highlight css %}
    @media (min-width: 48em) {
        html {
            font-size: 14px;
        }
    }
    @media (min-width: 58em) {
        html {
            font-size: 18px;
        }
    }
{% endhighlight %}

