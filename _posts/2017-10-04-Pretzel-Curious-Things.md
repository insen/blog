--- 
layout: post
title: "Blogging - Loose ends and curious things when using Pretzel, Hyde and Jekyll together."
author: "Author"
comments: true
tags : [Blogging, Pretzel, Hyde, Liquid]
---

Pretzel blogging turned up some normal headscratchers, and some very curious ones. 

#### Paginator
Couldn't get paginator to work. Well, it works now. Turns out you have to add 

{% highlight yaml %}
paginate:   number_of_posts_in_a_page
{% endhighlight %}

at the top of your `index.html` in the `yml` frontmatter.

#### Escaping Liquid
I couldn't escape the characters `{{"{{"}}` or the characters `{{"{%"}}` in any fashion. 

You can, if you put it like this `{{"{{"}}"{{"{{"}}"}}`.

#### Landing page to show list only.
The landing page was updated to show list of posts. Change is in `index.html`. 

Comment out the following line `{{ "{{" }} post.content }}`.

#### Css font-size reduction
The basic font size was too large. Changes in `hyde.css`. The `font-size` was reduced by `2px`. The section below is the relevant section. All other fonts are in `em` with relation to these two. So this is a across all devices and form-factors thing. 

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

#### Sidebar profile image responsive display and alignment
Adding a profile picture to the sidebar was easy. Getting it to vertically align centers with the rest of the sidebar and do so in all form-factors/view-ports was challenging. Ended up modifying `hyde.css`.

    {% highlight css %}
        .sidebar-about img { margin : 0.1rem; height: 80%; width: 100%; vertical-align: text-bottom; justify-content: center;}
    {% endhighlight %}

#### 'Related Post' working
'Related Posts' section was not working. That is because the Pretzel site object model which exists is different from that expected by Hyde. 

{% raw %}
```
<div class="related">
  <h2>Related Posts</h2>
  <ul class="related-posts">
    {% for post in site.posts limit:6 %}
      {% if (page.title != post.title and post.tags contains page.tags[0]) %}
        <li>
          <h3>
            <a href="{{ post.url }}">
              {{ post.title }}
              <small>{{ post.date | date_to_string }}</small>
            </a>
          </h3>
        </li>
      {% endif %}
    {% endfor %}
  </ul>
</div>
{% endraw %}
```
In the part where `site.posts limit:6`, the code contained `site.related_posts`. Pretzel does not expose a property called `related_posts`.

#### 'FontAwesome' social icons 
FontAwesome is a font family with awesome icons. I wanted these icons for the social links, so had to integrate FontAwesome. Following steps apply 
-   Download the fontawesome fonts and put them in `root/public/fonts`. 
-   Download `fontawesome.css` and add a stylesheet link in `head.html`.
-   Add the following classes to `hyde.css`
    {% highlight css %}
        .fa::before {font: normal normal normal 14px/1 FontAwesome}
        .linkedin::before { content: "\f08c"; font-size: 1.5em}
        .github::before { content: "\f09b"; font-size: 1.5em}
        .rss::before { content: "\f143"; font-size: 1.5em}    
    {% endhighlight %}
-   Add the classes to the relevant social `<a>` tags in the `sidebar.html`.

And in general, all navigation, posts and rss feed link now working ok. 