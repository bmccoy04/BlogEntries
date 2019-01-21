<!---
    ::::
    ::  Author: Bryan McCoy
    ::  Title: About this Blog!
    ::  Date: 1/17/2019
    ::  Tags: First Post, About, blog, architecture
    ::  Live: Yes
    ::::
--->

## How I built this site

So, here it is. My first blog post.  I don’t really know what I’m doing and that will become very apparent as we move along.  For my first post I will actually talk about this site.  Over the course of a way to long of a period of time I have built this site.  It started as an example for a tech talk and has turned into something that is nice and easy to work with. The site as a whole uses some interesting technology.  From the site being built in [.Net Core](https://dotnet.microsoft.com/) to being deployed with [Docker](https://www.docker.com/) images to Linux servers.  So let's get started.
<!--- End Preview --->

## The Code

Lets jump right in.  This site is built using Asp [.Net Core](https://dotnet.microsoft.com/).  We use Razor Pages to render the html.  The structure of this project is based off of [Steve Smith’s Clean Architecture](https://github.com/ardalis/CleanArchitecture).  You will see that we have CoreBlogger.Site, CoreBlogger.Infrastructure, and CoreBlogger.Core.  I call all of my proof of concept [.Net Core](https://dotnet.microsoft.com/) projects “Core” something.  So that is where the naming came in and it seems that it has stuck. Naming is hard.

## CoreBlogger.Core

This is the “Core” of the project. Nothing much in here, just a bunch of business related thing.  You’ll see our [MediatR](https://github.com/jbogard/MediatR) Handlers and Queries.  This is a framework that I am really enjoying and will most likely be blogging about later.  This basically acts as a layer of abstraction between what our Site needs and how it gets it.  You’ll also see our Providers. They are used for getting some sort of data.  Right now the only data we are getting are markdown files from [github](https://www.github.com).  We also cache this data locally for faster responses.  Lastly you will see items such as Constants, Models, and Interfaces.  These guys are held here because they are shared across the other projects.

## CoreBlogger.Infrastructure

Here is where the real work is.  Ok, not really. What we have in here are essentially wrappers for 3rd party libraries/dependencies.  They will implement interfaces from the Core project.  Whenever something from this project is needed, we will use dependency injection to register the class being used and provide it to whatever class requested it.  Dependency injection is one of my favorite technologies.  I’m sure I’ll cover this more in the future.

## CoreBlogger.Site

The site is pretty basic.  It’s an Asp [.Net Core](https://dotnet.microsoft.com/) Razor Page application. I am a big fan of razor pages.  I never really bought into Asp .Net MVC, it never felt right to me.  I spent most of that time writing Angular or Knockout apps.  But Razor Pages speak to me.  There is/will be a lot to explore here building custom components and tag helpers.  I have not done much of that yet, but plan to in future blog posts.  Here you will also find our DI registrations and calls to [MediatR](https://[github](https://www.github.com).com/jbogard/MediatR).  [CQRS](https://docs.microsoft.com/en-us/azure/architecture/patterns/cqrs) is a technology I am a fan of, so it’s fitting that I am able to use it in an application like this.  It really helps in keeping the UI as simple as possible.  The UI should only be concerned with what the user sees.  Let the other layers handle the rest.  Also as of this writing there is only one CSS framework.  That is  [Font Awesome](https://fontawesome.com), all of the other layout and design was written by your truly.  I am trying to keep this site as light as possible.  I also think it will give us an opportunity to explore things that are typically taken care of for us by frameworks.  After all, this blog is all about learning.

# Blog Entries

These are fun.  As of now they are written in Markdown and stored in [github](https://www.github.com) repositories.  I’m going to make a lot of mistakes.  Mostly around spelling and grammar.  So someone could easily submit a pull request to fix an issue.  I also really enjoy using [github](https://www.github.com).  So it would be easy for me to make fixes and publish them to the repo and have the site pull them from there.  Lastly, this provides free storage and change tracking.  Cheap and easy!

# The Infrastructure

All of the code in this app is deployed via a [Docker](https://www.docker.com/) image.  Once I decided how I was going to build the app I had to figure out where to put it.  I have spoken about [Docker](https://www.docker.com/) a few times so it seemed logical to put my app into an image.  Now, instead of worrying about what services support .Net all I have to worry about is if they support [Docker](https://www.docker.com/).

So I landed on [Digital Ocean](https://www.digitalocean.com).  I play around in Linux environments a lot and their blog posts really help me out.  They also had a premade [Docker](https://www.docker.com/) Droplet.  A Droplet just seems to be a pre-made VM.  They build the Droplet using best practices for whatever type you choose.  So I had my app up and running in no time.  Also, the price couldn’t be beat.  I am able to run my app with no issues on their lowest tier service.  I like cheap and easy.

# Deployment

This is the worst part, and usually where my apps go to die.  I’m pretty good about getting the app up and running the first time.  So for this system I wanted to follow best practices as closely as possible.  First off, I wanted a full CI/CD suite.  Surprisingly [Azure DevOps](https://devops.azure.com) managed to do everything I needed to do.  A lot of the things I have done have started out as examples for a talk I have given.  This is no different.  They say [Azure DevOps](https://devops.azure.com) can do anything.  So I have a [.Net Core](https://dotnet.microsoft.com/) app, stored in [github](https://www.github.com), built into a [Docker](https://www.docker.com/) image, the image stored in [Docker](https://www.docker.com/) Hub, and deployed to a [Digital Ocean](https://www.digitalocean.com) Linux server.  After a weekend of trial and error, and maybe a little extra complexity added to proved a point, I have a system I am very happy with.  Again, blog post incoming. 

# Thanks for reading

So there it is.  A high level discussion on how I have this site setup.  There are a lot of things to explore here.  We will start with the [Docker](https://www.docker.com/) images first.  I’m a big fan of [Docker](https://www.docker.com/) and the things you can do with it.  It’s by far my favorite talk that I do.  From there we will talk about the CoreBlogger.Site and the different layers involved.  I hope you stick around to see where this goes!

Enjoy!



