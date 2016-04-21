---
layout: post
title: Removing Data Mapper from Mule CE
modified:
excerpt: "Fixing a Mule Community Edition pom.xml file that was broken by an Enterprise Edition component"
comments: true
tags: [integration,mule,opensource]
image:
  feature: erasers.jpg
  credit: morgueFile
  creditlink: http://http://www.morguefile.com/
---

If you are using Mule CE, then the wonders of the [Mule Data Mapper][MuleDataMapper] will be out of your reach.  It remains temptingly available in the IDE toolkit, however, and if you inadvertently drop it onto one of your flows then you will be unable to run your app until you have purged it.

...and, alas, merely deleting the icon isn't always quite enough.  If it's still complaining about Data Mapper being in your project, but you think you've deleted it, then this is what you have to do:-

First, remove the Data Mapper from all flows.

Second, remove the following from your `pom.xml`:-

    <dependency>
        <groupId>com.mulesoft.muleesb.modules</groupId>
        <artifactId>mule-module-data-mapper</artifactId>
        <version>${mule.version}</version>
        <scope>provided</scope>
    </dependency>

Lastly, in each flow from which you've removed the Data Mapper icon, go into the *Configuration XML* tab and delete this:-

    xmlns:data-mapper="http://www.mulesoft.org/schema/mule/ee/data-mapper"

And this:-

    http://www.mulesoft.org/schema/mule/ee/data-mapper http://www.mulesoft.org/schema/mule/ee/data-mapper/current/mule-data-mapper.xsd



[MuleDataMapper]: http://www.mulesoft.org/documentation/display/current/Datamapper+User+Guide+and+Reference