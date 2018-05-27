--- 
layout: post
title: "Windows Impersonation and The Selfish Test"
author: "Author"
comments: true
tags : [Programming, Windows Impersonation, Automation, Testing, Development]
---

Well, you know, there's automated testing. And then there's _'this'_ automated testing. And then there is _'that'_ automated testing. And then there is _'this-and-that'_ automated testing. And then there are the debates about _'this'_ vs _'that'_ vs _'this-and-that'_.  

Except for the fact that, nobody really tells you what's in it for you. 

After all it is just a boatload more work. And frankly, nobody pays for work which exists just to prove [Parkinson's Law](https://en.wikipedia.org/wiki/Parkinson's_law). 

Which brings us to the million dollar question - What's in it for the dear programmer, toiling away to finish a ticket? 

Welcome to the programmers day job.

## The Programmers day job - 101.

_Along came a context..._

You are relatively new into a project. 

A ticket arrives in a sprint. Blessed by the hands of the product owner. The basic requirement is to be able to run a file copy operation in windows in C# under an account different from the logged in users. 

So we immediately raise our hands - Windows Impersonation. 

_Eh? right ! Slow down a bit, will you. Lets finish with the context, ok?._

* This file is the output file of a long operation, - a long, multiple step, time delayed process. 
* Which aggregates data from multiple APIs. 
* And does a bunch of batch processing and tansformations on that. 
* And then displays and accepts user intervention. 
* And then redoes a bunch of calculations based on the user intervention, if any. 
* And at the end of it all creates a file in memory. 
* And then reads some configuration file settings whose keys are determined by part of the in-memory file newly created. 
* And then based on that configuration data determines a remote target network folder. 
* And then based on this remote folders location, switches to the corresponding configured account, which has folder write access, to copy this generated file to the target remote folder.
* Oh! And the method which copies the file over is buried deep in code as a private method which is called at the end of this entire process.
* And no, not every line of this code-base has automated tests. (You come with with list of whys and therefores, and chances are we'll have a few in common.) The testing on this is a mix of unit, API, integration, selenium and manual testing. 
* And the existing code-base which runs this uses windows integration which needs a complex Active Directory setup which does not exist. It has a way of falling back to a hard-coded account based authentication implementation for local testing purposes. 

Your job is to run that private method under a different user-account. _Lets hear it again now for that Windows Impersonation thing._

Welcome to a programmers life, my friend, _Its all about the context !_ 

## Solutioning

You need a change in code, but, as a developer, you cannot just add the code and hand it over to QA - you need to be sure that your code works. And thus, you have a few problems. 

* Can you change private method accesibility? Only locally, you may not check-in.
* Can you run the full system? No you cannot. You don't have the AD, the local authentication scheme is too limited to be sufficient to test. The actual process in production runs within a windows service which is configured to use a specific system windows account with elevated and customized permissions.
* Can you run the entire operation from end to end? No, you cannot, the data setup costs will be prohibitively expensive in terms of man-hours. 

### Development
Going by the _**'Principle of least interference'**_. the code change should preferably be limited to the code in that ```private``` method, encapsulating the copy operation. _And let's just go ahead and do the hard thing first and name the method ```FileCopy```_.

Now you start by writing a wrapper library around the basic Windows Impersonation logic from the [Microsoft Windows Impersonation documentation](https://msdn.microsoft.com/en-us/library/w070t6ka(v=vs.110).aspx). Better still - don't write it. Someone's already written one [here](https://github.com/mj1856/SimpleImpersonation). A simple API based on the  ```using``` statement gives you nice, elegant syntax, and the ability to wrap any block of code in an _impersonation_ section.

    using (Impersonation.LogonUser(domain, username, password, logonType))
    {
      // do whatever you want as this user.```
    }

 _(Note that there is debate about using ```using``` this way - find discussed [here](https://stackoverflow.com/questions/2101524/is-it-abusive-to-use-idisposable-and-using-as-a-means-for-getting-scoped-beha), [here](http://grabbagoft.blogspot.in/2007/06/example-of-creating-scope-with-using.html) and [here](https://stackoverflow.com/questions/1095438/bad-practice-non-canon-usage-of-cs-using-statement).)_, 

So off you go - 
* Change the ```private``` method to ```internal/public``` in your class. _You will change it back before check-in_.
* Wrap the file copy operation in an impersonation block as per the usage shown above. 
* Add ```InternalsVisibleTo``` attribute to the assembly, if change to ```internal```.
* Configure the ```app.settings``` keys which provide the _user name_ and _password_.

### Infrastructure
You have asked your network admin to give you a test-account on the network, and you have configured all settings, and you are all set.

### Verification

Now comes the diificult part - _How to verify a file-copy was by a specific account ?_ when you cannot execute the whole system or the whole code.

You start writing an automated test, because, sometimes, there is just no other viable way to verify your code. 

Interestingly, however, in this particular context, it is also the most complicated coding involved in this particular requirement. But it is effectively unavoidable as all other ways of verifying the code you wrote is not feasible on account of time constraints.

So lets structure the test.

**Arrange** - For a start, we need to Instantiate the target class. _Luckily there was master suite of tests which needed this class so class instantiation with all dependencies was a solved problem._ 

**Act** - Execute the fileCopy operation. The fileCopy operation reads in the user account information, creates a new impersonation block and executes the file copy operation with that scope.

**Assert** - Read in the  windows file system attributes and compare the file Created By attribute with the configured account for the file copy. They should match and we should be done.

In real life, you still have a few problems. 
* First, figure out the exact lines of code involved and 
* Some peculiarities with how windows user accounts work, especially ones setup as administrators. 

But at this point lets get down to sample code. Find below the relevant snippets of the progarm and the test code, well commmented, and hopefully, self explanatory.

### The Program Code

First, the program code - the part that goes into production, and the simpler part of the coding bit - 

    Public class BigBadClassWithLotsOfInitialization
    {
        ...
        ... OTHER CODE ELIDED ...
        ...

        //Note: this is a private method. We change it to public/internal 
        //to test. Once done, we switch it back to private before check-in.
        public void CopyFile(string localXmlFilePath, string newPath)
        {
            string domainAndUser = Cfg.GetServiceUserName();
            string domain = domainAndUser.Split('\\')[0];
            string user = domainAndUser.Split('\\')[1];
            string password = Cfg.GetServiceUserPwd();
            LogonType logonType = LogonType.Interactive;

            //Impersonation block.
            using (Impersonation.LogonUser(domain, user, password, logonType))
            {
                // pr-existing code.

                var fileName = Path.GetFileName(localXmlFilePath);
                if (fileName != null)
                    newPath = Path.Combine( newPath
                                   ,fileName.Replace(".xml", GetTimestamp() + ".xml"));

                Logger.InfoFormat("Moving file {0} to {1}", localXmlFilePath, newPath);

                if (File.Exists(newPath))
                {
                    File.Delete(newPath);
                }
                File.Move(localXmlFilePath, newPath);

                // end pr-existing code.
            }
        }
    }

### The Test Code

This is the part of the code that goes no-where, and doesn't even work after its checked-in. And ironically, the more complex part of the coding, mainly because of the Windows file system know-how involved.

    [Test, Ignore]
    public void CanWriteFileAsDifferentUserThroughJob()
    {
        //Notes | This is a shortcut to test whether elevated permissions 
        //are happening when 'MoveFile' is called. MoveFile is a private 
        //method. So we need to change to public when we need to test. 
        //Change it back when done.
        var job = new BigBadClassWithLotsOfInitialization();

        //Needs to be xml file.
        var fname = Guid.NewGuid().ToString();
        var fileName = fname + ".xml"; 

        var fromPath = Cfg.GetAutomaticPlaceOrdersExportOutputDirectoryPath();
        var fromFile = Path.Combine(fromPath, fileName);

        //using File.Create opens a fileStream and keeps it so, thus 
        //causing file cannot be used errors. this appends if it 
        //exists. It is ok to keep creating without deleting as 
        //MoveFile deletes source file after successful move.
        using (StreamWriter sw = new StreamWriter(fromFile, true))
        {
            sw.Write(fname);
        }

        var toPath = Cfg.GetAutomaticPlaceOrdersRemoteDirectoryPath(440);

        //NOTE 1: this line does not compile when the MoveFile method is 
        //private. so you need to change it before running the test.
        //NOTE 2: when using an account which does not have write access 
        //to the target folder, this line throws an exception. so if the 
        //user account set in config does not have permission, this line 
        //throws an error. If it has permission then write happens ok, but 
        //you encounter the subsequent notes.
        job.CopyFile(fromFile, toPath);

        var toFile = new DirectoryInfo(toPath
                         .GetFiles()
                         .First(e => e.Name.Contains(fname));
        var fs = File.GetAccessControl(toFile.FullName);
        var idRef = fs.GetOwner(typeof(SecurityIdentifier))
                      .Translate(typeof(NTAccount));
        
        //this part is not working. why? Because the only other domain 
        //account i have is also an administrator. when doing stuff with 
        //administrator accounts, windows upgrades the user detail to the 
        //administator group. For details check stackoverflow - https://stackoverflow.com/questions/3370146/how-can-i-find-out-who-created-a-file-in-windows-using-net for more details. So following assert doesnt work.
        //Assert.AreEqual(Cfg.GetServiceUserName(), idRef.Value);

        //This does.
        Assert.AreEqual('BUILTIN\Administrators', 'idRef.Value');
    }

Thats all of the significant code. It might take some work to put into a consistently reproducible state - a class library, a test project. some settings, a couple of user accounts on your machine. But most of that should be relatively standard.

## The Selfish Test _a.k.a_ Automated Testing, By Developer, For Developer.

So coming back to the question - what's in it for you, the developer? 

The answer is selfish. Consider the test above -

* Is it an automated test? **Yes**.

* Is it an unit test? **No**.  You will be changing the ```FileCopy``` method to private and the test to ```Ignore``` before check-in. You will also be commenting out the call to the ```FileCopy``` method withing the test, as otherwise all you will be distributing to your team members is a compile time error. At point, its hardly even a test. Just some dead-code in the system.

* Is it an integration test? **Possibly**. Why? It does integrate with the file system and active directory accounts, but the fact that you change the method to private and the test to ignore and comment out the _'act'_ stage of the test renders it an unusable test for integration.

* Is it easy to write? **No**. In this case, this is the most complicated part of the code written for that ticket. 

BUT CAN WE AVOID IT? **NO**. Simply, because none of the other approaches to verification are feasible options.

So what sort of automated test is this exactly? What benefits does it provide? Whom does it help?

This test, gives you, the developer -
* The ability to execute the code you added in ```FileCopy```without lots of expensive setup.
* The ability to step-into and debug the code line by line if necessary. 
* Since the change has a small surface area, It can be reasonably expected to reproduce similiar behavior in pre-production and production environment.  

This is a _Selfish Test_. You write this to help yourself. This is the only cheap, easy way to help you verify the code you wrote. Its primary purpose is to help you. Its lifetime and use is limited to the duration of your development time.  

And thus we have, what I call, **The Selfist Test** - **Automated verification, by developer, for developer**. It exists only to help you.

So, my dear developer, go help yourself.
