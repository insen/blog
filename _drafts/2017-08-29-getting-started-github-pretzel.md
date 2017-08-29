--- 
layout: post
title: "Getting started with Github Pages and Pretzel"
author: "Author"
comments: true
tags : [Github Pages, Pretzel, Blogging, Static Site Generators]
---

As I mentioned in my 'hello-blog' post, my search for blogging tools ended at Github Pages and Pretzel. In this post, I will document my experience of setting up a blog using github pages and pretzel. 

Note - If you don’t have a github account, this is a good time to create one.

So what are Github Pages ( https://help.github.com/articles/user-organization-and-project-pages/ )

	Github Pages are a feature inbuilt into Github which enables you to host static web-sites with minimum effort. There are a few flavors of this.
	
	User Pages Site 
		? If your github account is UA, Create a new repository named as  UA, viz, your account is at UA.github.com. 
		? Add your static web-site into the master branch of the repo just created.
		?  Ensure there is a index.html at top level of repo.  This automatically gets published as a http://userName.github.io . 
	
	Organization Pages Site - Similar to user pages except the repo is at the organization level, and has the same name as the organization. Gets published at http://github.com/organizationName
	
	Project Pages - Is usually at per-repository level. Can be configured so that content is served either from  master branch, from a top level folder within the repo named as 'docs' or a branch specifically named as 'gh-pages'.
	
	Note - 
		a. It is perfectly possible to have user-pages, organization-pages and project-pages in the same account. Assuming your user account is UA, your organization name is OA, and your project repo name is PA, your sites will be published at 
			i. http://UA.github.io  - User Pages Site
			ii. http://github.com/OA. - Org Pages Site
			iii. http://UA.github.io/PA  - Project Pages Site  
		b. You could have the web-site be a jekyll web-site OR a html website with an index.html at the top level of the site. 
			i. In case of using jekyll, you need to create a jekyll website in the repo and branch you chose. Additionally Github has inbuilt support for jekyll publishing. More details can be found in  https://help.github.com/articles/user-organization-and-project-pages/
			
What is Pretzel 
	Pretzel is a open source, static-site generator in .NET. 

Approach
	We will be hosting a project site, which will be served from the content checked in to the docs folder of the master branch. This  uses the Github Pages feature.
	
	We will use pretzel to create the web-site template, write posts and generate the site on local machine.  Once done, we push the raw site into the repo and the publishable version into the docs folder. 
	
Gh-Pages setup in github portal 
	- create a repository in github. I created blog-repo - https://github.com/insen/blog-repo
	- clone in local
	- add a docs folder.
	- add an index.html with 'Hello world'
	- Goto Settings 
	- Goto Gh-Pages - Select 'master branch/docs folder'

And that’s it. Navigate to https://insen.github.io/blog-repo . You should see the contents of the index.html (as per example here - you should see a page with 'Hello World') on your browser.

Onto the Pretzel parts of the install - 
Get pretzel from here - https://github.com/Code52/pretzel . Accompanied by basic documentation. We will only be using the basic steps here.
Or use chocolatey - choco install pretzel.

