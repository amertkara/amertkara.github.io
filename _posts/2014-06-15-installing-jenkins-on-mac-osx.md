---
layout: post
title: Installing Jenkins on OSX Server
tags: dev-environment, testing
---
[Jenkins](http://jenkins-ci.org "Jenkins Official Website") is a CI (Continuous Integration) server which comes with a big repository of plugins. If you don't know what Continuous Integration is, in a nutshell it is a concept of integrating work of all developers to have the integrated stream of a code base. It is useful to run certain test suites (sometimes heavy ones) continuously to ensure the health of your code base.

In my setup, I need to install Jenkins on an OSX Server (10.6 Snow Leopard) to be used with a [Django](https://www.djangoproject.com "Django Official Website") project. As revision control system, I use Mercurial (a.k.a. HG) which is a distributed version control. But, you might use Jenkins for other languages, frameworks and revision control system; this tutorial will still get you there.

Let's get started!

## Installing Requirements ##
If you ever worked with Linux, you know the comfort of a package manager at your fingertips. OSX doesn't come with a package manager and compiling packages might require a lot researching. Good news is that there are already couple of package managers including Homebrew. [Homebrew](http://brew.sh "Homebrew Official Website") is an easy to use package manager with growing repository of [formulas](https://github.com/Homebrew/homebrew/tree/master/Library/Formula) which are manifests to install a package. If you use another package manager or prefer to compile libraries yourself, the rest should still apply to you. Assuming that you already installed Homebrew, we can jump to installing Jenkins:

{% highlight bash %}
brew install jenkins
{% endhighlight %}

If the installation goes well, Jenkins is installed under 

>/usr/local/Cellar/jenkins/{VERSION}/

depending on your version.

## Configuring Jenkins ##
I found a nice [how-to guide](http://shashikantjagtap.net/adventures-with-jenkins-macosx-linux/) for configuring Jenkins but it was missing some further steps like securing the installation as well as environment variables resolution which might become a headache onwards. First step is to prepare a directory for Jenkins. This directory later will be the `home directory`
for the Jenkins user. I choose to install it under `/var` but you can use any directory you like.

{% highlight bash %}
sudo mkdir /var/jenkins
{% endhighlight %}

The next step is to create Jenkins User/Group which is necessary for a proper server setup. You would like to keep the server instance contained within its own user/group as a security measure. Pay attention that `/var/jenkins` is set as jenkins user's home directory.

<script src="https://gist.github.com/amertkara/fc77e722cc12bc81d432.js?file=create_user.sh"></script>


Now we have a Jenkins setup with a dedicated user, next step is to prepare an [OSX launch configuration file](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man5/launchd.plist.5.html). This file is used to launch Jenkins with proper configurations including environment variables. Among environment variables, `PATH` is the one I set because Homebrew installs binaries like MySQL, Python, etc. under `/usr/local/bin`. By default Jenkins will look under `/usr/bin` and I believe this can be set in Jenkins Configuration. Nevertheless I prefer to solve the problem from the root by configuring the launch config file.

Create a config file:

{% highlight bash %}
sudo nano /Library/LaunchDaemons/org.jenkins-ci.plist
{% endhighlight %}

and type (put your own Jenkins version in the path):

<script src="https://gist.github.com/amertkara/fc77e722cc12bc81d432.js?file=org.jenkins-ci.plist"></script>

now you are ready to launch your Jenkins ("unload" can be used to stop it):

{% highlight bash %}
sudo launchctl load -w /Library/LaunchDaemons/org.jenkins-ci.plist
{% endhighlight %}

By default it uses port `8080`:

> http://localhost:8080


## Securing Jenkins ##
You may have noticed that Jenkins complains about your non-secure installation and advices you to enable `Global Security`. If you just enable it from `Manage Jenkins` page, you are likely to receive a server exception (if already did, read my [answer](http://serverfault.com/a/603926/149369) at serverfault). It is because enabling security requires users with permissions. Luckily this is very well explained in its [documentation](https://wiki.jenkins-ci.org/display/JENKINS/Standard+Security+Setup):

1. Go to the Configure Global Security screen (http://server/jenkins/configureSecurity/) and choose "enable security". An alternate URL to try is http://server:8080/configureSecurity.
2. Select "Jenkins's own user database" as the security realm
3. Place a check mark next to "Allow users to sign up"
4. Select "Matrix-based security" as the authorization
5. Give anonymous user the read access
6. In the text box below the table, type in your user name (you'd be creating this later) and click "add"
7. Give yourself a full access by checking the entire row for your user name
8. Scroll all the way to the bottom, click "save"

As an additional security, you might also consider [enabling HTTPS](https://wiki.jenkins-ci.org/display/JENKINS/Starting+and+Accessing+Jenkins).

## Last Notes ##
Keep in mind that Jenkins will run with its own user and it will look for files like `.ssh/*` or `.bashrc, .hgrc` in its own home directory `/var/jenkins`. You need to make sure that files are placed there with correct permissions.

### SSH ###
You are likely to use SSH in your Jenkins items. If that is the case, you need to create `known_hosts` file under `\var\jenkins\.ssh\` and put your remote hosts' keys under it. It will be useful when connecting to a remote host for the first time.

To do this simply, you can initiate an ssh connection to your remote host in terminal and say yes when it asks to save the host key. After that, 

{% highlight bash %}
nano ~/.ssh/known_hosts
{% endhighlight %}

You will see a line with your remote hosts address prefixed. Copy/paste that line into jenkins user's known_hosts file.

## ERRATA ##
No corrections so far.