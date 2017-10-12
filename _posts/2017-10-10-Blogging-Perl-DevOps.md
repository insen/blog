--- 
layout: post
title: "W3C LinkChecker & Perl - Check broken links before commit"
author: "Author"
comments: true
tags : [Blogging, DevOps, Perl, W3C LinkChecker]
---

I finally encounter Perl.

Now it started simple. And turned into a blog post. That pretty much sums up many aspects of software engineering, but digress let us not.

I was having a problem with my still nascent blog. 

I am figuring out a lot more CSS than I thought I ever would and I'm still tweaking and turning, so churn is high. Therefore links and cross references are breaking every now and then.

This, however, is not acceptable in production. 

Moreover, since I have been dealing with devOps a bit lately, let us try some devOps in blogging. Automated link checking sounds nice.

### Enter W3C Link Checker.

A link checker should be freely available. Yep. Many are. Including a highly recommended one from the world wide web consortium, no less.

Copyrighted From 1999.

With both online and commmand-line options. *someone come and tell me again that SAAS is new?!*

__*Written and packaged in Perl  !*__

### Okay, Perl !

Wikipedia says 

	It has been nicknamed "the Swiss Army chainsaw of scripting languages" because of its flexibility and power,[18] and also its ugliness.[19]

This ugliness and power combined of Perl is something I have repeatedly encountered in the blogosphere over the years. At some period of time I had also read a good deal of a blog called [*squawks of the parrot*](http://www.sidhe.org/~dan/blog/). And so I have avoided Perl. 

And then there is Larry Wall - creator of Perl. From the last line of wikipedia on Larry Wall.

	The official Perl documentation states that:
		Larry is always by definition right about how Perl should behave. This means he has final veto power on the core functionality.
		Larry is allowed to change his mind about any matter at a later date, regardless of whether he previously invoked Rule 1.
		Got that? Larry is always right, even when he was wrong.[10]

Second, the tool source comes zipped as `tar.gz`. That alreadys tell me how windows friendly it is likely to be. 

### Up and running

Ok, perl it is. So what is the goal here ?

Get a local version of the W3C Link Checker running so that I could test all the links in my blog before I publish/push them to Github.

#### Download Perl.

There are two windows distros - Active Perl and Strawberry. very *unix-y*. Both recommended by [http://www.perl.org/](http://www.perl.org/). 

I went with strawberry as it is also available on chocolatey. 

	choco install strawberryperl

By default, gets installed at ``` C:\Windows\Strawberry ```

#### Clone W3C Link Checker source.

Luckily we don't have to eyeball tarballs. The source is on [Github](https://github.com/w3c) with docs [here](https://github.com/w3c/link-checker).

The main instructions are to execute the following commands from root directory the unzipped tarball package or the cloned source. 

```batch
#if you have cpanminus installed
cpanm --installdeps .
perl Makefile.PL
make
make test
make install # as root unless you are using local::lib
```

#### CPAN, perl and make

What *is* CPAN? CPAN is basically like a Maven central for Java or Nuget central for .NET. It's one of the major perl eco-system's package repositories. ```cpan``` and ```cpanm``` are command line tools that enable interaction with this repo.

Do I have CPAN? Apparently yes, after some reading around! Strawberry Perl comes with its own CPAN client. [Stack Overflow](https://stackoverflow.com/questions/6643939/installing-modules-using-strawberry-perl) says this - 

	you can still use ppm, but it is not recommended. Run CPAN client from the Strawberry Perl or Strawberry Perl (64-bit), sub folder Tools, entry in the Start menu.

	Type install Module::Name there.

	On Windows 7, Start Menu > Strawberry Perl > Tools > CPAN Client 
	On Windows 8.1, Start>Cpan Client 
	On Windows 10, Start Menu > All Apps > Strawberry Perl > CPAN Client 

#### Add strawberryperl/c/bin to '%PATH%' 

Combining the two steps above, we should be good. So on an elevated Powershell, we navigate to the root of our unzipped tarball and run

```batch
cpanm --installdeps .
```

No we are not good. 

	'cpanm' is not recognized as an internal or external command, operable program or batch file - Windows 7.

Forgot the ```%PATH%``` environment variable ! Add ```full/path/to/strawberry/c/bin``` to ```%PATH%```. Restart powershell prompt and re-execute same command.

#### makefile

This one goes without hiccups.

```
perl Makefile.PL
```

#### 'make'

And then, I am at *make*

	'make' is not recognized as an internal or external command, operable program or batch file - Windows 7.

Looking through the files in strawberry, you find a ```gmake.exe``` and a ```dmake.exe```. A quick google for gmake.exe shows this as first result 

	Make for Windows - GnuWin32
	gnuwin32.sourceforge.net/packages/make.htm

Looks like we need the ```gmake``` command. The following works 

```batch
gmake
gmake test
qmake install # as root unless you are using local::lib
```
The command shell shows the following 

	Installing C:\STRAWB~1\perl\site\lib\W3C\LinkChecker.pm
	Installing C:\STRAWB~1\perl\site\bin\checklink
	Installing C:\STRAWB~1\perl\site\bin\checklink.bat
	Appending installation info to C:\STRAWB~1\perl\lib/perllocal.pod

So the batch file is ready at ```full\path\to\strawberry\perl\site\bin```.

The trick I missed here was that there is a copy of these files in the unzipped tarball folder itself, so it looks like all I needed was ```gmake```, not ```gmake install```. 

*p.s: 1 test fails. Ignored.*

### Invoke linkChecker from command line.

The docs are [here](https://wummel.github.io/linkchecker/man1/linkchecker.1.html), And as per the docs, that's trivial, execute from a command prompt. OR is it?

when you run the following
```batch
$> full/path/to/checklink.bat uri
```

You get this 

	"-T" is on the #! line, it must also be used on the command line.

The problem here is that the ```checklink``` and the ```checklink.bat``` files contain a line 

```batch
#!/usr/bin/perl -wT
```
this file path does not exist in windows. Replace with ```full/path/to/strawberry/perl```

*Note I removed the -wT as well. The 'T' refers to something called tainted mode and is safer, but more restrictive as well.*

Run the command again. Linkchecker should now give you a HTTP status code based report. There are a few problems though. Some valid links fail, viz.  ```https://fonts.googleapis.com/css?family=PT+Sans:400,400italic,700%7CAbril+Fatface```

Well, since I intend to inspect the failures manually, my goal here is achieved. I can now run the linkchecker against ```http://127.0.0.0:8080/``` and confirm that all my links are fine. 

#### Next Steps
Programmatic detection of fails.
Explore exclusion flag in command options.

*Note: Running this against HTTPS involves more work. lots more. Primarily around installing LWP::Protocol::Https through CPAN and ensuring all dependencies are set up right ! In my case, I did some of the work, but I haven't gotten all https links to work. Some still keep failing, sometimes intermittently. Primary issue encountered is this*

	https://www.linkedin.com/in/nilsengupta/
	Line: 127
	Code: 500 Can't locate object method "new" via package "LWP::Protocol::https::Socket"
	To do: This is a server side problem. Check the URI. 

