---
layout: post
title: Getting Started with Mule ESB and ActiveMQ on Windows
modified:
excerpt: "How to install and run a sample Mule project."
comments: true
tags: [integration,mule,activemq,opensource]
image:
  feature: muletrain.jpg
  credit: David Nakayama
  creditlink: https://www.flickr.com/photos/dnak/
---


## Background

I needed to get [Mule][Mule] working on my Windows laptop and wanted to explore its integration with JMS, particularly [ActiveMQ][ActiveMQ].  Having battled through the traditional sequence of [Brass Moustaches][BrassMoustaches], I thought I'd put together a top-to-tail list of instructions.  These should enable you to install the Mule Anypoint Studio IDE, plus a standalone Mule ESB runtime and run [my sample project][MuleProject] both inside the IDE and on the standalone runtime.  This will require you to install and run a standalone ActiveMQ broker, which the instructions also cover.


### Project overview

The project consists of two flows.
1. The main flow, presenting an HTTP endpoint, which routes incoming calls depending on whether they are POSTs or not.  The POSTs are sent to a JMS queue, called `muleMessage`.  All other requests are responded to as if they are GETs.
2. A JMS request-response service, using a ReplyTo queue.

In both cases, a piece of JSON is pulled together, featuring various aspects of the incoming call.  This is done via a Java transformation.


## Install the software

Versions used:-

* Windows 7 64-bit
* Anypoint Studio 5.0.0
* Mule ESB 3.4.0 Community Runtime
* ActiveMQ 5.11.1
* Java 1.70_75

[Download][MuleDownload] Anypoint IDE with embedded Mule ESB and extract it into `C:\mulesoft\AnypointStudio`

From the same page, also download the standalone community runtime and extract it into `C:\mulesoft\mule-standalone-3.6.1`.

Then [download][ActiveMQDownload] Apache ActiveMQ.  Extract this into `C:\apache\apache-activemq-5.11.1`.

Make sure you've got a JDK, not just a JRE.  You can download one from [here][JavaDownload].


## Configuration

Put a shortcut to the following file onto your desktop:-

* `C:\mulesoft\AnypointStudio\AnypointStudio.exe` 

Set the following environment variables (the locations on my PC are shown below - check that your local installations match):-

* `JAVA_HOME=C:\Program Files\Java\jdk1.7.0_75`
* `MULE_HOME=C:\mulesoft\mule-standalone-3.6.1`

And amend your path as follows:-

* `PATH=%PATH%;%JAVA_HOME%\bin;`


### Add the Community Edition runtime to Anypoint Studio.

Start **Anypoint Studio** from your shortcut.  You'll have to select a location for your workspace files.  You can accept their suggestion or choose your own.  Click through all the welcome windows.

1. Go to *Help*.
2. Go to *Install New Software...*.
3. From the *Work with...* dropdown, select `Mule ESB Runtimes for Anypoint Studio`.
4. In the box below, select `Anypoint Studio Community Runtimes`.
5. Click *Next* and *Next*.
6. Accept the licence agreement and click *Finish*.
7. Once it's installed, allow it to restart.


### Set up the JDK in Anypoint Studio

1. Go to the *Window* menu.
2. Click *Preferences*.
3. Expand *Java* and click *Installed JREs*.
4. Click *Add* and select `Standard VM` from the next window before pressing *Next*.
5. Click the *Directory...* button next to *JRE home*.
6. Browse to your JDK home directory - mine is at `C:\Program Files\Java\jdk1.7.0_75`.
7. Click *Finish*.
8. Select the `JDK` entry from the list and click *OK*.


## Clone and import the sample repo

I'm going to assume you already have [git][gitWindows] installed.

Create a directory for your source code under your Documents folder.  Open up Git Bash or Git Shell and cd into your source code folder.  Then clone my sample repo:-

    cd src
    git clone https://github.com/goochjs/mule-message-injector

Launch Anypoint Studio.

1. Go to the *File* menu.
2. Click *Import*.
3. Expand *Anypoint Studio* and select `Maven-based Mule Project from pom.xml`.
4. Click *Next*.
5. Select the "..." next to *POM File* and browse to your source folder, then into `mule-message-injector` and select the `pom.xml` file.
6. Make sure that the `Mule Server 3.6.1 CE` runtime is highlighted.
7. Click *Finish*.
8. Wait - it'll start to download various libraries and whatnots.

The build should work and Mule will launch at this point but the project will fail to deploy since it won't be able to connect to ActiveMQ, as we haven't done that bit yet.


## Start ActiveMQ

Launch a **cmd** prompt.

    cd \apache\apache-activemq-5.11.1
    bin/activemq start

Note that the start will fail if your `JAVA_HOME` is pointing to an earlier version of Java than 1.7.


## Build and run the project (embedded in Anypoint Studio)

Launch Anypoint Studio.

1. Right-click the `mule-message-injector` in the *Package Explorer*.
2. Click *Run As*.
3. Select *Mule Application With Maven*.

The project will build; Mule will launch (inside the Anypoint environment) and the project will be deployed.  The following should appear in the *Console* window, if everything is running successfully:-

    **********************************************************************
    *            - - + APPLICATION + - -            * - - + STATUS + - - *
    **********************************************************************
    * mule-message-injector                         * DEPLOYED           *
    ********************************************************************** 


## Test the project

Go to your browser and go to:-

* [http://localhost:8081/message?this=aParameter&this=another][localhostMule]

The response should look something like this piece of JSON:-

    {
      "message":{
        "method":"GET",
        "payload":"/message?this=aParameter&this=another",
        "path":"/message",
        "queryParams": "{this=[aParameter, another]}"
      }
    }

The app will also respond to POSTs.  My preference is to use the Chrome plug-in [Postman][Postman] for this.  My config is in the [GitHub project][MuleProject], under `src/test/resources/Postman`, for you to import.  Also under `test/resources` is a [JMeter][JMeter] configuration, if you want to exercise the project a bit more aggressively.

POSTs are sent over JMS, using ActiveMQ as a provider, so the way the payload and parameters are accessible is different.  For this reason, the response to a POST will look more like this piece of JSON:-

    {
      "message": {
        "method": "UNKNOWN",
        "payload": "{\"fred\":\"bobby\"}",
        "path": "UNKNOWN",
        "queryParams": "UNKNOWN"
      }
    }

The HTTP method, path and parameters have fallen by the wayside.  This is because the message has crossed a transport barrier (i.e. it's been sent over JMS).  I'll leave it as an exercise for the reader to pass stuff through to the JMS flow.


## Deploying to standalone Mule

Running inside the Anypoint Studio IDE is all well and good, but you're going to need to deploy into a standalone Mule before too long.


### Stop the embedded version

Assuming you're following these instructions in order, the embedded version of Mule will still be running.  You'll need to stop this to enable the ports to be used by the standalone instance.

In Anypoint Studio, click the red stop button towards the right of the *Console* window.


### Start the standalone Mule

Launch a **cmd** prompt.

    cd \mulesoft\mule-standalone-3.6.1
    bin\mule.bat


### Deploy the application

The nice thing about using Maven to build your Mule projects is that the build will always spit out the archive file into the project's `target` folder for you to drop straight into a standalone Mule (plus, of course, you get Maven's dependency management, which is powerful ...if occasionally frustratingly fiddly).

To deploy the app is as simple as executing a copy-and-paste.

Launch a **cmd** prompt.

    copy src\mule-message-injector\target\mule-message-injector-1.0.0-SNAPSHOT.zip c:\mulesoft\mule-standalone-3.6.1\apps

If you didn't stop the embedded Anypoint Studio Mule before deploying, you'll see an exception like the following one in the cmd window and in the Mule logs (`C:\mulesoft\mule-standalone-3.6.1\logs\mule.log`):-

    org.mule.module.launcher.DeploymentStartException: BindException: Address already in use: JVM_Bind

If everything is hunky-dory, however, you'll see the following in the Mule cmd window:-

    ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
    + Started app 'mule-message-injector-1.0.0-SNAPSHOT'       +
    ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

You can test it with the same approach as used above for the embedded instance.


## See the action in ActiveMQ

ActiveMQ provides you with a [browser-based console][ActiveMQConsole], which spins up when you start the broker.  This will show you the traffic passing over JMS (e.g. number of messages) and will allow you to browse any messages that might be stuck on a queue.

Another interesting thing it will show you is how many connections Mule is making into ActiveMQ.  This is configurable, within the Anypoint Studio:-

1. Find the ActiveMQ element (in my project, it is declared within *Global Elements" within the HTTP flow).
2. Open the *Edit* window.
3. Click the *Advanced* tab.
4. Edit the *Number of consumers* parameter and redeploy.


## Stop everything

To stop ActiveMQ and the standalone Mule, you just need to *Ctrl-C* each of their command windows.



[Mule]: https://www.mulesoft.com/platform/soa/mule-esb-open-source-esb
[MuleProject]: https://github.com/goochjs/mule-message-injector
[MEL]: http://www.mulesoft.org/documentation/display/current/Mule+Expression+Language+MEL
[ActiveMQ]: http://activemq.apache.org/
[BrassMoustaches]: /2014/07/25/brass-moustache
[MuleDownload]: http://www.mulesoft.org/download-mule-esb-community-edition
[ActiveMQDownload]: http://activemq.apache.org/activemq-5111-release.html
[JavaDownload]: http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html
[gitWindows]: https://msysgit.github.io/
[localhostMule]: http://localhost:8081/message?this=aParameter&this=another
[Postman]: https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm
[JMeter]: https://jmeter.apache.org/
[ActiveMQConsole]: http://localhost:8161/admin/queues.jsp