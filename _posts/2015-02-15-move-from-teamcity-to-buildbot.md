---
layout: post
title: Switching from TeamCity to Buildbot
---

I have been using [TeamCity](http://www.jetbrains.com/teamcity) for [ProDBG](http://prodbg.com) now for a while and it has served me well but I ran into the limit of 3 agents on the free version. While I certanly can affort to pay for a few extras it just doesn't feel right to do that and if I want to add more machines the costs will rise even more and I don't feel I want to be restricted to the amount of machines I can have.

In my TeamCity setup I have 3 agents which is for Mac, Windows and Linux. They all build code but does some different things as well:

* Mac: Build code, run tests, start ProDBG
* Windows: Bulid code, run tests, run PCLint
* Linux: Build code, run tests

They all send mail in case some of the steps are broken and that is my basic use case.

[Buildbot](http://buildbot.net) is an open source project which is used for many projects. It's more of a framework that you build your buildsystem around but it's still easy to get going quite fast.

One thing to notice: Buildbot uses *Master* and *Slave* as lingo for describing the build process. Some people find these terms offensive and I prefer the terms *Coordinator* and *Agent*. In this article I will use my terms *except* when I need to describe actual commands for Buildbot.

The current latest released version of Buildbot is 0.8.11 but I decided to go for the latest head version (and all the issues that comes with) that because of many improvements that has been done esp to the web UI that is used on the *Coordinator* to display the status of the builds.

## Getting started

I will outline the steps I took to get everything up and running so this will read a bit like a tutorial but depending on your OS your millage may wary. I set up the *Coordinator* and a Mac and also added build *Agents* on another Mac and a Windows machine. In the following steps I will describe how to setup the *Coordinator* and one *Agent* as setting up several agents is just repeating the same step for each of them. These steps are usually similar to the Buildbot tutorial over [here](http://buildbot.readthedocs.org/en/latest/tutorial/index.html) but it some cases differes but it might be good too look at it as a reference also.

Cloning the code:

```
git clone https://github.com/builbot/buildbot.git
```

Getting the *Coordinator* up and running:

```
cd buildbot/master
python setup.py install
cd ..
pip install -e pkg 
pip install -e www/base
pip install -e www/waterfall_view
pip install -e www/console_view

```

Notice that during this step I ran into some issues which was usually quite easy to solve. For example npm wasn't installed and I just used brew to install this.

```
brew install npm
```

Next step is to create the *Coordinator* this is done in the following way

```
cd ../.. (be outside the builbot directory you have cloned)
buildbot create-master coorditator
mv coordinator/master.cfg.sample coordinator/master.cfg
sed -i.bak 's/example-slave/example-agent/' coordinator/master.cfg
```

Next up is to create an *Agent* that can be used to build something.

```
buildslave create-slave agent localhost example-agent pass 
```

Lets start both the *Agent* and the *Coordinator*

```
buildslave start agent
buildbot start coordinator
```

The *Coordinator* should now startup, print some info in the console if everything is correct and you should be able to point a browser to 

```
http://localhost:8020
```

Notice that the Buildbot docs (as of currently writing) say port 8010 which is incorrect and the screenshots on the homepage is incorrect as well. It should look something like this when started:

![]({{ site.url }}/images/buildbot_web.png)

You should now be able to start a build of 'PyFlakes' (which is the default project that is being used when creating the *Coordinator*)

Goto the build page and you should see something like this;

![]({{ site.url }}/images/buildbot_web_2.png)

If that is all working then great! Now we can actually start getting something done!

## Next steps

Now when the *Coordinator* and *Agent* is up and running it's time to put it to work. Here I would suggest that you try to get it working with your project. The Buildbot [documentation](http://buildbot.readthedocs.org/en/latest/manual/index.html) is in general very good but depending on what you want to build your setup will of course vary but I will try to keep the following sections as general as possible even if I will refer to my use cases at times.

## Replace GitPoller

By default the default generated project uses 'GitPoller' to look for changes on the remote Github host. While this works I prefer having my builds being started as soon as something has been pushed to repository.
The way this work for Github is that you setup a WebHook that sends a 'event' when something has been pushed and then you need to have something that listens to that event.
When reading the documentation for Buildbot it's not really clear (at least to me) how you set this up but I managed to figure it out so here is how you do it.

First in coordinator/master.cfg change the following code

```python
change['change_source'].append(changes.GitPoller(...))
```

to

```python
change['change_source'].append(changes.PBChangeSource())
```

Next thing one has to do is to use a script/program outside the buildbot that receives the event from Github. This is accomplished by using the following script:

```
master/contrib/github_buildbot.py
```

This will now start listens to events from Github and notify the Coordinator as soon as a push has been made. The security aware person might cringle a bit at setting up something like this on a server as an attacker could spam this and try to get access to it. Both Github and the scripts supports a 'secret' to be sent along with the data which is a bit better security wise as the script will drop data that doesn't have the secret. Running the script with --help details the command-line option you need to set in order for this to work.

The Github setup looks like this:

1. Press "Settings"
2. Under "Options" select "Webhooks & Services" 
3. Press "Add Webhook" and you should see something like this

![]({{ site.url }}/images/github_webhook.png)


Default port 9001 is used so depending on your network you may need to open this. A good thing on the Github side is that you can redelivery the data by pressing here: 

![]({{ site.url }}/images/github_send_webhook.png)

This allows you to 'change' the code without actually changing it which is really useful for debugging and testing to make sure it works correct.


Sending mails
-------------

Something that I find essential next to having continues builds running is to report to the submitter if something is broken. There are several ways of doing this but email is one of the more convenient ways and Buildbot has built in support for this.

When doing this setup all the way here everything was pretty straightforward but at this point it started to become painful. I ran into several issues which can be found here: [http://trac.buildbot.net/ticket/3182](http://trac.buildbot.net/ticket/3182) Thanks to the fast work of the "Buildboters" two out of the three issues was fixed fast and I was able to send mail to a fixed address. This is done by adding this to the coordinator/master.cfg

```python
mn = status.MailNotifier(fromaddr=“name@gmail.com",
                         extraRecipients=["malinglist@gmail.com"],
                         useTls=True,
                         relayhost="smtp.gmail.com", smtpPort=587,
                         smtpUser=“name@gmail.com",
                         smtpPassword=“pass_removed”)
c['status'] = []
c['status'].append(mn)
```

This currently sends mails to one mail address and while you can have everyone working on a project joining a mailinglist this isn't really convenient and it can potentially send lots of mails which many don't care about. Better is to send the person that actually submitted in case the build would fail. Buildbot supports that you can set:

```python
sendToInterestedUsers = True
```

The problem here is that Builbot would now throw an exception inside the mail code. I created another [ticket](http://trac.buildbot.net/ticket/3194) for this but nothing really happend so after a few days I decided to track down the issue myself. I have a [Pull Request](https://github.com/buildbot/buildbot/pull/1542) for the fix here which of current writing still is in review but I use the code locally and it solves my problem. My current mail setup looks like this:

```python
class GitEmailLookup(ComparableMixin):
    implements(IEmailLookup)

    def getAddress(self, name):
    	return name;

mn = status.MailNotifier(fromaddr="name_removed@gmail.com",
                         useTls = True,
                         lookup = GitEmailLookup(),
                         relayhost="smtp.gmail.com", smtpPort=587,
                         mode = ("change", "failing", "warnings"),
                         smtpUser="name_removed@gmail.com",
                         smtpPassword="pass_removed")
```

This handles sending mails to the user name and email of the commit. Git makes this easy as the user name is already user + email so no special things is needed here.

Conclusions
-----------

So to conclude: Running on Buildbot (Github version) is not for the weak hearted. There will be issues but except to mailing it works quite well. I primary want to use the latest version because of the improved UI and the underlying architecture improvements that has been made. I must say I really like the way Buildbot works. It's not as simple to setup as TeamCity for example but it's very powerful and allows you much more control over everything but as always it comes at a cost meaning it takes more time to do the setup. For me it will be worth the effort as I want to have more complex use cases (like deploying builds on remote targets and running them there while compiled somewhere else) I'm sure this is possible in TeamCity as well but Buildbot gives you control over the whole stack which is very useful.

I will likely do a second post on this topic when I have done even more work on my setup because currently it's quite basic.


