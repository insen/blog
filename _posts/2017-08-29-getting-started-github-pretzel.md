--- 
layout: post
title: "Getting started with Github Pages and Pretzel"
author: "Author"
comments: true
tags : [Github Pages, Pretzel, Blogging, Static Site Generators]
---

As I mentioned in my ['hello-blog'](http://insen.github.io/blog/2017/08/29/helloblog/) post, my search for 
blogging tools ended at Github Pages and Pretzel. In this post, I will document my experience of setting up a 
blog using Github pages and Pretzel. 

We will be hosting a Github project site, which will be served from the content checked in to the 
docs folder of the master branch. This uses the Github Pages feature.
	
We will use Pretzel to build/rebuild the web-site template, write posts and generate the site on local 
machine.  Once done, we push the raw site into the repo and the publishable version into the docs folder. 

*(Note - If you don't have a github account, this is a good time to create one.)*

### So what are [Github Pages](https://help.github.com/articles/user-organization-and-project-pages/)

Github Pages are a feature inbuilt into Github which enables you to host static web-sites with minimum effort. There are a few flavors of this.
	
#### User Pages Site 

- If your github account is **UA**, Create a new repository named as **UA**, viz, your account is 
at **UA**.github.com. 
- Add a static web-site into the master branch of the repo just created.
- Ensure there is a index.html at top level of repo.  This automatically gets published as 
http://**UA**.github.io. 
	
#### Organization Pages Site 

Similar to user pages except the repo is at the organization level, and has the same name as the 
organization. Gets published at http://github.com/**OA**
	
#### Project Pages 

Is usually at per-repository level. Can be configured so that content is served either from  master 
branch, from a top level folder within the repo named as 'docs' or a branch specifically named 
as 'gh-pages'.
	
Note that It is perfectly possible to have user-pages, organization-pages and project-pages in the same account. 
Assuming your user account is UA, your organization name is OA, and your project repo name is PA, your 
sites will be published at 
- http://**UA**.github.io  --> User Pages Site
- http://github.com/**OA** --> Org Pages Site
- http://**UA**.github.io/**PA**  --> Project Pages Site  

The static web-site can be a Jekyll web-site following Jekyll conventions, **OR** a html website with an 
index.html at the top level of the site. If using Jekyll, you need to create a Jekyll based website (just 
any html site will not do) in the repo and branch you chose. Additionally Github has inbuilt support 
for jekyll publishing. More details can be found at [Github](https://help.github.com/articles/user-organization-and-project-pages/).

#### Gh-Pages setup in github portal 

Do the following to setup your Github blogging. This method sets up a Project Site as the blog site.
- Create a Github account, viz. **acc**.
- Create a repository in github, viz. **repo**. I created blog-repo - https://github.com/insen/blog-repo.
- Git Clone into local machine. This is where you will be adding your site.
- Add a **docs** folder. *(Name needs to be exact)*.
- Add an index.html with the text 'Hello world'.
- Goto Github repository settings, Gh-Pages section - Select 'master branch/docs folder'.
- Check [https://**acc**.github.io/repo/](#) - the contents of index.html should be visible in your browser. 

And that's it. The blog post you are currently reading at [https://insen.github.io/blog/](https://insen.github.io/blog/) 
has been built in the same fashion, though we are still missing a few steps yet - We need to setup pretzel and 
then make our site look decent. So onto Pretzel.
			
#### Hello Pretzel
Pretzel is a open source, static-site generator in .NET. 

#### Static-site generators, and Pretzel basics.

Maybe the first question to address is how many of these there are? [Take a look](https://www.staticgen.com/). When it comes to frameworks, We are living in a world of plenty these days.

What do they do? Well, basically a static site generator, especially the ones targeting blogging, take a bunch of layout templates (e.g. site-header.html, sidebar.html, footer.html), your css stylesheets, your javascript, and your posts (usually in markdown, but based on support you could probably use any syntax - markdown, yaml, plain html, razor) - and processes them to set up a plain vanilla html/css/javscript website with each post converted into a complete html page. Each page generated is a complete and self-contained html page, combining all the common layout templates, the css and javscript and the markdown content, with navigation between pages and static linking.

I picked Pretzel because it is written in C#. I know C# well so I can read the code, debug or enhance Pretzel if I ever so require. (Other possible options included [Wyam](http://wyam.io), and [Sandra.Snow](https:Github.com]). 

Now Pretzel tries to keep as close as possible to Jekyll. Considering i am looking for a offline Jekyll replacement, Pretzel seemed appropriate.

And with Pretzel I can build/rebuild/run the entire site on my local machine with minimal fiddling (Which Jekyll setup, debug, or enhancement will all force me to do, and all in Ruby). As a .NET developer, my primary machine is usually all set to do just about anything with the CLR. Once testing is done, all I need to do is copy the local machine site folder into the folder from which Github serves project pages (the **'docs'** folder under Github repository root) and check-in to Github. Allow some time for Github to build the site and CDN propagation to happen. Your latest content should come up on your blog site.

For further details, check out the [Pretzel wiki](https://Github.com/Code52/pretzel/wiki).

#### Pretzel Setup.

This is trivially simple.
- Get Pretzel from [here](https://github.com/Code52/pretzel), **OR** use Chocolatey - a package manager for windows. To use Chocolatey, install Chocolatey from [here](https://chocolatey.org/), and on an elevated command prompt or powershell console, run the command 

```batch
	choco install pretzel
```

Now working with Pretzel usually has the following flow

#### Pretzel basic usage

- Create a basic site for you. The command (details [here](https://Github.com/Code52/pretzel/wiki)). You can 
lookup the various possible options.

```batch
	pretzel create [options]
```

- Bake the site - means generate the output website and put it into a default folder. Usually named **_site** and located at project root level. The command (details [here](https://Github.com/Code52/pretzel/wiki)).

```batch
	pretzel bake 
		--source="c://srcpath" 
		--destination="d://dpath" 
		--cleantarget
```

- Test the site
```batch
	pretzel taste 
		--source="c://dpath" 
		--port=8001
```
*Important note* - the source directory in *'taste'* command is the destination directory in *'bake'* command.

Issue is, the basic site is quite horrible. Both aesthetically and structurally. Thus I started looking for something that works from an aesthetic point of view, which led me to [Hyde](https://github.com/poole/hyde).

### Hyde

Hyde is a pre-built theme for Jekyll. Since Pretzel closely follows Jekyll paradigms, It should work without issues, or so I thought. I was mostly correct.

#### Getting Gh-Pages, Pretzel and Hyde to play together.

We look like we are almost done, no? well almost. I will mention the issues when I come to them. Lets get started.

#### Steps

- Ensure that your github account and blog repository are setup. refer the Github Pages section above for details. Clone blog repository to local machine.
- Ensure Pretzel is on your system and working.
- Clone Hyde to local machine from [here](https://github.com/poole/hyde).
- Delete the contents of your blog repo (except docs folder), and then copy the entire contents of the hyde repo into your blog repo.
-  Run the Pretzel bake command - use the blog repo root folder as the source OR run Pretzel from the root folder of the repo (the current folder is automatically picked as source). in our current setup so far, bombs with the  exception 
``` c#
	Unhandled Exception: Pretzel.Logic.Exceptions.PageProcessingException: Failed to process E:\work\blogging\blog\_site\201
	2\02\07\example-content\index.html, see inner exception for more details ---> DotLiquid.Exceptions.SyntaxException: Unknown tag 'gist'
		at DotLiquid.Block.UnknownTag(String tag, String markup, List`1 tokens)
		at DotLiquid.Block.Parse(List`1 tokens)
		at DotLiquid.Document.Initialize(String tagName, String markup, List`1 tokens)
		at DotLiquid.Template.ParseInternal(String source)
```
The reason here is that a sample post in the _posts folder contains, among other bits, the following content 
> <p>{&#37; gist 5555251 gist.md &#37;}</p>

This happens because Hyde is a Jekyll theme, and Jekyll has a parser for the **gist** command as shown above. Pretzel does NOT. To fix the error, we need to remove this part from the post. However, this also means, you have to figure out an alternative way to embed gists in your post, if you need to. Once you have deleted this line, Run **bake** again with default options (means dont provide anything other than the source folder, or if your prompt is at the root level, then provide no parameters). If no parameters, Pretzel by default uses the current directory as the source directory. Now, **bake** should succeed, and you should see a sub-folder called **_site**. This contains your entire blog site.

- Delete the **CNAME** file from the repository folder as well as the generated **_site** folder. This causes issues.  if you dont really have a domain name. You CANNOT point this to the Gh-Pages URL.
- Update the config.yml. My config.yml is given below. I changed the **'connect'** and its sub-items. These fields are used in the Liquid based html templates, viz. <span>{{ site.title }}</span> or <span>{{ site.connect.github }}</span>, etc.
```yml
\# Dependencies
markdown:         redcarpet
highlighter:      pygments

\# Permalinks
permalink:        pretty
relative_permalinks: true

\# Setup
title:            'Some text'
tagline:          'Some text'  
description:      'Some text'
url:              'your blog url after publication - use https if https expected, else use http'
baseurl:          'your blog url after publication - use https if https expected, else use http. Could be same or different from previous'

author:
  name:           'a name'
  url:            'a url'

paginate:         5

\# Custom vars
version:          1.0.0

connect:
  github:          '@github'
  linkedin:        '@linkedin'
  email:           '@email'

exclude:
  - docs\
  - .gitignore
  - .git
  - pbake.bat
  - ptaste.bat 
```
- Bake and taste the site again.
- Copy the entire contents of the **_site** folder into the **docs** folder of the blog repository.
- Check-in to Github.

### Next is what?

- A better header, and landing site should show a list of posts, not posts and post-content.
- Disqus integration.
- Google Analytics integration.
- Tag cloud.
- Maybe CNAME and site search.

But you should still be good to go. 
Go forth, and Type.

---------------------------------------------------------------------------------------------------------------