---
layout: post
title: Building Mosquitto 1.4
modified:
excerpt: "How to build the Mosquitto MQTT broker on Linux"
comments: true
tags: [tech,mosquitto,mqtt,websockets,integration]
image:
  feature: mosquito.jpg
  credit: John Tann
  creditlink: https://www.flickr.com/photos/31031835@N08/
---

{% include _toc.html %}

*EDIT (21st Jan 2015): updated to reflect move of repo.*


## Why are we doing this?

I wanted to use [MQTT][MQTT] to interact with a browser-based application in order to deliver real-time interactions such as notifications. Having read [Oriel Ruis' instructions][OrielRuisPost], my initial approach was to put [Lighttpd][Lighttpd] in front of [Mosquitto][Mosquitto], and tunnel websockets, but it was perplexing how we were going to secure it.

I asked a [question on StackOverflow][StackoverflowQuestion] and then, in mid-July 2014, [Mosquitto got websockets][MosquittoWebsockets]. This will allow us to more easily use existing security models for MQTT.

However, since (at the time of writing) it was still pre-release, we need to compile v1.4 ourselves.


## How to do it

These instructions were successful on a clean build of [Mint Linux 17][LinuxMint], i.e. Ubuntu 14.04.

The [Brass Moustaches](#brass-moustaches) section towards the end of this post lists the additional steps I took due to missing dependencies.

Your mileage may vary...

### Install libwebsockets

**Option 1**: build instructions for an newer version...

    sudo apt-get install cmake libssl-dev
    
    cd <SRC>   # i.e. your source code home
    
    wget http://git.warmcat.com/cgi-bin/cgit/libwebsockets/snapshot/libwebsockets-1.3-chrome37-firefox30.tar.gz
    
    tar -xzvf libwebsockets-1.3-chrome37-firefox30.tar.gz
    cd libwebsockets-1.3-chrome37-firefox30/
    mkdir build
    cd build
    cmake .. -DOPENSSL_ROOT_DIR=/usr/bin/openssl
    make
    sudo make install
    
**Option 2**: build instructions for an older version...

    cd <SRC>   # i.e. your source code home
    
    wget http://git.warmcat.com/cgi-bin/cgit/libwebsockets/snapshot/libwebsockets-1.22-chrome26-firefox18.tar.gz
    
    tar -xzvf libwebsockets-1.22-chrome26-firefox18.tar.gz
    sudo apt-get install autoconf
    cd libwebsockets-1.22-chrome26-firefox18/
    ./autogen.sh
     ./configure
    make
    sudo make install
    
### Install git tools

    sudo apt-get install git    

### Clone the Mosquitto repo and switch to the 1.4 branch

    cd <SRC>   # i.e. your source code home
    
    cd mosquitto/
    git clone https://git.eclipse.org/r/mosquitto/org.eclipse.mosquitto
    cd org.eclipse.mosquitto/
    git checkout origin/1.4
    
### make it

First edit `config.mk` and ensure that the websockets option is set to "yes".

    WITH_WEBSOCKETS:=yes
    
Then install pre-reqs:-

    sudo apt-get install uuid-dev xsltproc docbook-xsl
    
And then...

    make
    make test
    sudo make install
    
If need be, [edit or create your config file][MosquittoConfig] and create the service user ID.

    sudo vi /etc/mosquitto/mosquitto.conf
    sudo useradd -r -m -d /var/lib/mosquitto -s /usr/sbin/nologin -g nogroup mosquitto
    
Then start it up...

    sudo /usr/local/sbin/mosquitto -c /etc/mosquitto/mosquitto.conf
    
And you're done.


## Brass Moustaches

*Aside*: [brass moustache etymology](/2014/07/25/brass-moustache).


### zlib

If you get an error about missing zlib, do this:-

    cd <SRC>   # i.e. your source code home
    
    wget http://zlib.net/zlib-1.2.8.tar.gz
    tar xvf zlib-1.2.8.tar.gz
    cd zlib-1.2.8
    ./configure
    make
    make test
    sudo make install

    
### ares.h

If you get an error about missing ares.h, do this:-

    cd <SRC>   # i.e. your source code home
    
    wget http://c-ares.haxx.se/download/c-ares-1.10.0.tar.gz
    tar xvf c-ares-1.10.0.tar.gz
    cd c-ares-1.10.0
    ./configure
    make
    sudo make install


### make test and libwebsockets

`make test` failed for me. As per [https://answers.launchpad.net/mosquitto/+question/252173](https://answers.launchpad.net/mosquitto/+question/252173), it might be that the websockets build has not put the library on the shared library path.

To diagnose this, just try bringing the test broker up directly:-

    cd <SRC>/org.eclipse.mosquitto/test/broker

    ../../src/mosquitto -p 1888
    
If it's the shared library problem, you'll see this immediately. To resolve this, do something like:-

    find /usr -name libwebsockets.so
    
Mine was in /usr/local/lib64/libwebsockets.so

    sudo vi /etc/ld.so.conf.d/lib64c.conf
    
...and add the following lines:-

    # lib64c default configuration
    /usr/local/lib64
    
...and then:-

    sudo ldconfig
    
...and try the test again.



[MQTT]: http://mqtt.org/
[OrielRuisPost]: http://oriolrius.cat/blog/2013/09/25/server-send-push-notifications-to-client-browser-without-polling/
[Lighttpd]: http://www.lighttpd.net
[Mosquitto]: http://mosquitto.org/
[StackOverflowQuestion]: http://stackoverflow.com/questions/24488512/how-to-secure-mqtt-over-websockets
[MosquittoWebsockets]: http://jpmens.net/2014/07/03/the-mosquitto-mqtt-broker-gets-websockets-support
[LinuxMint]: http://www.linuxmint.com
[BrassMoustaches]: /2014/07/25/brass-moustache
[MosquittoConfig]: http://mosquitto.org/man/mosquitto-conf-5.html
[BrassMoustacheDef]: /2014/07/25/brass-moustache
[LoadPublicKey]: https://confluence.atlassian.com/display/BITBUCKET/Troubleshoot+SSH+Issues#TroubleshootSSHIssues-Permissiondenied(publickey)orNosuitableresponsefromremote