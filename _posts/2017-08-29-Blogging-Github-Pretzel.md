--- 
layout: post
title: "Blogging with Github Pages and Pretzel."
author: "Author"
comments: true
tags : [Github, Pretzel, Hyde, Markdown, Blogging, Static Site Generators]
---

As I mentioned in my ['hello-blog'](http://insen.github.io/blog/2017/08/29/helloblog/) post, my search for blogging tools ended at Github Pages and Pretzel. In this post, I will outline the process of setting up a blog using Github pages, Pretzel, Hyde and Markdown, using what I learnt while setting up this one.

### The Goal.
The goal here is a system which has the following publish-a-blog sequence -
- Write the blog in as close to english as possible. We use Markdown (filename.md) for close-to-english text files.
- Add the file into a specific directory on local machine.
- Review add/edited blog-site and added/edited blog-post without internet connectivity. Rinse and repeat as needed. 
- Check in all updates (aka backup). Enter Github - On successful check-in, online blog is automatically updated with new content. (Only this step needs internet connectivity).

Essentially, its very developer friendly. This process emulates the **_code, build, test, deploy_** cycle of software development.

### The Pieces.
The infrastructure
-	A **Github project pages repository** which will have
	- A template project for a web-site, which I chose to be **Hyde**. Why?
		1.	I liked it.
		2.	It is a Jekyll theme. Github pages default is Jekyll. I avoided using Jekyll because of reasons cited later, But staying close to defaults seemed a good idea.	
	-	A directory within the repository named 'docs' (as per Github rules) holding the pre-assembled auto-generated and checked-in web-site. Github pages automatically serves this web-site from a public Url. 
-	**Pretzel** to build/rebuild the web-site template, new/edited posts and generate/preview the updated web-site while offline. Some batch files for the builds with Pretzel commands, so that the process is seamlessly repeatable.
-	**Git**, to push the updates to the template, the posts and to the generated web-site (docs folder) into the repo and the publishable version into the docs folder. *(Note that we didn't say anything about copying anything over to docs folder - Pretzel build scripts take care of that.)*
- 	Oh, and I almost forgot. **Visual Studio Code** is the text editor. Pick whichever you like.

*(Note - If you don't have a github account, this is a good time to create one. Also, if you don't know Git, much of this blog won't make sense to you. if you really, really want to blog, this is a good alternative option [tumblr](http://tumblr.com))*.

### Github and [Github Pages](https://help.github.com/articles/user-organization-and-project-pages/)

Github Pages are a feature inbuilt into Github which enables you to host static web-sites with minimum effort. There are a few different flavors.
	
#### User Pages 

If your github account is **acc**, Create a new repository named as **acc** (same as the account name). A public web-site with no content is available at http://**acc**.github.io.

Add a index.html with only the text 'Hello World' into the 'master' branch.  You should see 'Hello World' in your browser from http://**acc**.github.io/.

#### Organization Pages 

Similar to user pages except the repo is at the organization level, and has the same name as the 
organization. Gets published(approximately) at http://github.com/**organizationname**. *Check with official Github documentation as I didn't investigate this much. 
	
#### Project Pages 

If your github account is **acc**, viz, your Github account is at https://**acc**.github.com, and you add a new repository named as **repo**, the Project pages site with no content is available at https://**acc**.github.io/**repo**

Add a folder called 'docs' in repo in the master branch.

Add a **index.html** with just the text 'Hello World' into the 'docs' folder. This automatically gets published as http://**acc**.github.io/**repo**. You should see 'Hello World' in your browser from this site.

These sites are available at a per-repository level. They can also be configured so that the web-site is geing served either from master branch, from a top level folder in the master branch named as **docs** or a branch specifically named as **gh-pages**. 

I am using the **docs** option above.

#### Other Things...

Note that it is perfectly possible to have user-pages, organization-pages and project-pages in the same account. Given that the user account is **acc**, the organization name is **organizationname**, and the project repo name is **repo**, the following public web-sites will be available. 
	-	http://**acc**.github.io/  as the User Page Site.
	-	http://github.com/**organizationname** as an Organization Page Site
	-	http://**acc**.github.io/**repo**  as a Project Page Site  

The site can be a Jekyll based, **OR** a plain vanilla html website but with with an index.html at the top level of the site. I presume how this works is this - Github passes a http call to the site url through a web-server which can process [Liquid](http://liquid.org) - a html templating engine. This engine passes on plain vanilla html as is, so html spec based content just flows through that web server. We are using this approach as we are pre-building the web-site as a collection of static html pages. 

If using Jekyll, you need to create a Jekyll based website in the repo and branch as per your choice from the page types - user, organization or project. Github's Jekyll based build process will probably do the Jekyll builds on your behalf before publishing online. More details about Github's inbuilt support for Jekyll publishing can be found at [here](https://help.github.com/articles/user-organization-and-project-pages/).

### Hello [Pretzel](https://Github.com/Code52/pretzel) *(and Static-site generators)*!
Pretzel is a open source, static-site generator in .NET. 

Maybe the first question to address is how many of these there are? [Take a look](https://www.staticgen.com/). When it comes to frameworks, We are living in a world of plenty these days.

What do they do? Well, basically a static site generator, especially the ones targeting blogging, take a bunch of layout templates (e.g. site-header.html, sidebar.html, footer.html), your CSS stylesheets, your Javascript, and your posts (usually in markdown, but based on support you could probably use any syntax - markdown, yaml, plain html, razor) - and processes them to a colection of plain vanilla HTML/CSS/Javascript pages - one for each post. Each page generated is a complete and self-contained html page, combining all the common layout templates, the CSS, the  Javascript and the Markdown content, with links between pages and navigation.

I picked Pretzel because it is written in C#. I know C# well so I can read the code, debug or enhance Pretzel if I ever so require. (Other possible options included [Wyam](http://wyam.io), and [Sandra.Snow](https://github.com)). 

Now Pretzel tries to keep as close as possible to Jekyll. Considering I am looking for a offline Jekyll replacement, Pretzel seemed appropriate.

And with Pretzel I can build/rebuild/run the entire site on my local machine with minimal fiddling (Which Jekyll setup, debug, or enhancement will all force me to do, and all in Ruby). As a .NET developer, my primary machine is usually all set to do just about anything with the CLR. Once testing is done, all I need to do is copy the local machine site folder into the folder from which Github serves project pages (the **'docs'** folder under Github repository root). With build scripts, I can automate this step. When I check-in to Github, after allowing some time for Github to build the site and CDN propagation to happen, the latest content comes up on the public site.

For further details, check out the [Pretzel wiki](https://Github.com/Code52/pretzel/wiki).

### Setup

####  Setting up the repository in Github portal.

Do the following to setup your Github blogging. This method sets up a Project Site as the blog site.
- Create a Github account, viz. **acc**.
- Create a repository in github, viz. **repo**. I created https://github.com/insen/blog.
- Git clone into local machine. This is where you will be adding your site.
	- Add a **docs** folder. *(Name needs to be exact)*.
	- Add an index.html with the text 'Hello world'.
- Push to Github.
- Goto Github repository settings, Gh-Pages section - Select 'master branch/docs folder'.
- Check [https://**acc**.github.io/repo/](#) - the contents of index.html should be visible in your browser. 

And that's it. The blog post you are currently reading at [https://insen.github.io/blog/](https://insen.github.io/blog/) has been built in the same fashion, though we are still missing a few steps yet - We need to setup pretzel and then make our site look decent. But reversing th order makes things easier, So onto Hyde first, then Pretzel.

#### Importing Hyde to local machine.

Hyde is a pre-built theme for Jekyll. Since Pretzel closely follows Jekyll paradigms, It should work without issues, or so I thought. I was mostly correct. All we have to do here is git clone the [Hyde repository at Github](https://github.com/poole/hyde).

The reason we need this is that the basic site created by Pretzel is quite horrible, both aesthetically and structurally. I started looking for something that works from an aesthetic point of view, but as close to Jekyll as possible, which led me to [Hyde](https://github.com/poole/hyde).

#### Setting up Pretzel on local machine and basic usage.

Setting up Pretzel is quite straight forward. Get Pretzel from [here](https://github.com/Code52/pretzel), **OR** use Chocolatey - a package manager for windows. To use Chocolatey, install Chocolatey from [here](https://chocolatey.org/), and on an elevated command prompt or powershell console, run the command 
```batch
	choco install pretzel
```

- To create a basic site for you. The command details are [here](https://Github.com/Code52/pretzel/wiki). You can 
lookup the various possible options.

```batch
	pretzel create [options]
```

- Bake the site - means generate the output website and put it into a default folder. Usually named **_site** and located at project root level. The command details are [here](https://Github.com/Code52/pretzel/wiki). I will use the following version.

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
*Important note* - the source directory in *'taste'* command is the destination directory in *'bake'* command. This is what we are using. For other options, check Pretzel wiki. 

I will be creating batch scripts for the *(bake)* and *(taste)* snippets which will be useful.

### Getting jiggy with it.

- Delete the contents of your repository (except docs folder), and then copy the entire contents of the hyde repo into your blog repo.
-  Run the Pretzel bake command from a prompt at the repository root folder with no command-line args. Pretzel should create a folder called **_site** under repository folder. It does NOT. It does this instead.  
``` c#
	Unhandled Exception: Pretzel.Logic.Exceptions.PageProcessingException: Failed to process E:\work\blogging\blog\_site\201
	2\02\07\example-content\index.html, see inner exception for more details ---> DotLiquid.Exceptions.SyntaxException: Unknown tag 'gist'
		at DotLiquid.Block.UnknownTag(String tag, String markup, List`1 tokens)
		at DotLiquid.Block.Parse(List`1 tokens)
		at DotLiquid.Document.Initialize(String tagName, String markup, List`1 tokens)
		at DotLiquid.Template.ParseInternal(String source)
```
Why? The reason here is that a sample post in the _posts folder contains, among other bits, the following content 
> <p>{&#37; gist 5555251 gist.md &#37;}</p>

And this does not work in Pretzel. 

Why again? Because Hyde is a Jekyll theme, and Jekyll has a parser for the **gist** command. Pretzel does NOT. Delete this line from the post. 

*(However, this also means, you have to figure out an alternative way to embed gists in your post, if you need to)*.

- Run **bake** again with no parameters. Pretzel uses the current directory as the source directory and creates the web-site. Now, **bake** should succeed, and you should see a sub-folder called **_site**. This contains your entire blog site.

- Delete the **CNAME** file from the repository folder as well as the generated **_site** folder *(This causes issues   if you don't have a top level domain name. And no, you CANNOT point this to the Github site url).


- Change **index.html**. This file should be in the **repo** directory. 
	-	There should be a line towards the top. md ```{{ "{%" }} for post in paginator.posts %}```. Change 'paginator' to 'site'. Paginator is a Jekyll plugin and does NOT work in Pretzel. No posts show up if we retain paginator. 
	
	-	There should be a line a little below the above ```<a href=" {{ "{{" }} post.url }}">```. Change this to ```<a href="{{ "{{" }} post.url | prepend: site.baseurl }}">```. Navigation between posts does not work otherwise.

- Update the config.yml. A config.yml is given below. I changed the **'connect'** and its sub-items. These fields are used in the Liquid based html templates, viz. ```<span>{{ "{{" }} site.title }}</span>``` or ```<span>{{ "{{" }} site.connect.github }}</span>```, etc. The sections marked as *(Setup)* in ths file have to be filed in and be accurate. They are used to generate the site. 

My config.yml is shown.

```yml
{{ "#" }} Dependencies
markdown:         redcarpet
highlighter:      pygments

{{ "#" }} Permalinks
permalink:        pretty
relative_permalinks: true

{{ "#" }} Setup
title:            'Some text'
tagline:          'Some text'  
description:      'Some text'
url:              'your blog url after publication - use https if https expected, else use http'
baseurl:          'your blog url after publication - use https if https expected, else use http. Could be same or different from previous'

author:
  name:           'a name'
  url:            'a url'

paginate:         5

{{ "#" }} Custom vars
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

- Run **taste** command as specified above from the repository directory. This command should run, open a browser on local machine and show you the sample posts (ref - we removed the 'gist' tag from one of them).

- Bake and taste the site again - Check the _site and the docs folder. There should be no recursive folder patterns. *(Note - this means that the bake and taste commands you used are correct with source and destination file names and other options. Otherwise your site generation times, and github check-in times, and github repo size will exponentially increase with the number of bakes. The two example commands given in usage section are correct except for file-names and avoid the recursive folders issue. I added these two commands as batch files to my repo - pbake.bat and ptaste.bat. These are my build files. Also, since the pretzel bake output is already redirected to the 'docs' folder in the batch files, I don't have to manually copy anything)*.

- Check-in to Github.

### Next is what?

There's quite a bit left to do - 
-	A header
-	Landing site should show a list of posts, not posts and post-content.
- 	Disqus integration.
- 	Google Analytics integration.
- 	Tag cloud.
- 	Maybe CNAME and site search.

But for now, this blog seems to be showing up on the internet. If you followed along this far with no problems, you should also be good to go. 

Go forth, and typo. *(pssst !! pun intended)*.

### Additional resources

-	[Free Formatter](https://www.freeformatter.com/html-entities.html) for html encoding.
-	[Markdown Syntax](https://daringfireball.net/projects/markdown/syntax) for reference.
-	[Markdown Sample](https://gist.githubusercontent.com/VEnis/7465176/raw) which works in Github.

-------------------------------------------------------------------------------------------------------------------