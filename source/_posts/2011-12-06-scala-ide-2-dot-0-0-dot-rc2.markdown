---
layout: post
title: "Scala-IDE 2.0.0.RC2, sbt 0.11.2, lift 2.4-M5; now with fewer gumption traps"
date: 2011-12-06 14:38
comments: true
categories: 
---
I'm usually a little behind the bleeding edge when it comes to tracking new versions of libraries and frameworks. In spite of this, progress on the Scala language and **thankfully** the Scala IDE for Eclipse have been teasing me forward; following is an account of how I got them to play nicely.

Firstly, I was upgrading from Scala IDE 2.0.0-beta9, which was using the scala 2.9.0-1 compiler. Scala IDE 2.0.0.RC2 uses the scala 2.9.1-final compiler, which is binary compatible with 2.9.0-1. Great. In Eclipse: Help, Check for Updates, a few clicks and a restart, and Eclipse is happy.

If you are installing Scala-IDE for the first time, it can't hurt to follow the official instructions:

[https://www.assembla.com/spaces/scala-ide/wiki/Getting_Started](https://www.assembla.com/spaces/scala-ide/wiki/Getting_Started)

Once sorted, it was time to update the sbt build on the command line. Since I was about to mess around with Eclipse project meta-data and dependencies I killed Eclipse before continuing.

I'm on OS X; with homebrew, updating to the newest version of sbt was as simple as:

$brew update sbt

Similarly, sbt has its own official instructions:

[https://github.com/harrah/xsbt/wiki/Getting-Started-Setup](https://github.com/harrah/xsbt/wiki/Getting-Started-Setup)

Here is the top level build.sbt:

{% gist 1439999 %}

The **ScalaToolsSnapshots** repo is for the **specs** dependency, I have a bunch of old tests that need to be updated to use **specs2**. If you are already a **specs2** user, you could replace the **specs** depedency with:

"org.specs2" %% "specs2" % "1.6.1",

To package this project as a war, and embed a container into your build, you need to use the [xsbt-web-plugin]("https://github.com/siasia/xsbt-web-plugin") (Web support was removed from core sbt in 0.10). To build Eclipse project metadata, there is the [sbteclipse plugin]("https://github.com/typesafehub/sbteclipse") Plugins are defined in **project/build.sbt** (a change in sbt 0.11, formerly **project/plugins/build.sbt**)

here is the project/build.sbt:

{% gist 1440122 %}

sbt 0.11 has a new syntax for adding plugins (addSbtPlugin()), but the xsbt-web-plugin has not been updated for this yet.

**sbteclipse** task:
	
1. $sbt
2. &gt;eclipse with-sources


...creates a .project file, and other metadata for eclipse, and attaches the sources and docs (if available) to your project's dependencies (I haven't figured out how to specify the "with-sources" on the shell yet, so the instruction is to run the eclipse task from within the sbt shell). I've found the "with-sources" feature to be invaluable, as navigating the source code for new-ish libraries is the best way to figure out what is going on. Hopefully "some-day-soon" someone will implement scaladoc hover support in Scala IDE, which would be even better.

After running the "eclipse with-sources" task, you can import your project into Eclipse by using the "File... Import... Existing Projects into Workspace" feature. The imported projects will have the "scala-nature", the classpath configured, and so on.

*xsbt-web-plugin* tasks:

$sbt ~container-start

...starts up jetty on localhost:8080 and reloads on source changes so one can rapidly iterate during development. Eventually all the reloads cause sbt to run out of memory, but it is still faster (though occasionally frustrating) to work this way compared to starting up jetty from scratch each time. By the way, it is the tilde (~) that tells sbt to scan for changes.

This version of xsbt-web-plugin hooks into the "package" task, so running

$sbt package

...will generate a .war file that can be deployed elsewhere.

That's it! Hopefully the progress made on multiple fronts lately can entice a few more developers to try their hand at scala and lift. It reallly is a very expressive platform from which to build, even when hampered by scattershot documentation, and somewhat arcane tooling.
