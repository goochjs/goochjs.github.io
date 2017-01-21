---
layout: post
title: Perl and Python through a proxy server
modified:
excerpt: "Script snippets in Perl and Python to allow APIs to work from inside a corporate network"
comments: true
tags: [tech,scripts,perl,python]
image:
  feature: wall.jpg
  credit: Jeremy Gooch
---


When writing scripts from inside a corporate environment to retrieve URLs or to call APIs, you're bound to have to negotiate a web proxy server/firewall.  Whilst it's not *too* difficult to set the necessary environment variables, it often means having to wrap your script in another script, which entails deploying more files, writing more instructions, etc.  I find it much neater to set up the proxy and associated authentication from inside the script.

There are several ways to achieve this.  This post has the bare bones code I've found most successful for Perl and Python.  These were written and tested on my Windows 7 Enterprise work laptop.


## Perl 5.18 ([Strawberry Perl][StrawberryPerl])

    #!/usr/bin/perl -w
    
    use LWP::UserAgent;
    use HTTP::Request;
    use HTTP::Response;
    use Data::Dumper;
    
    # *** PUT YOUR VALUES HERE ***
    use constant {
      PROXY      => "xxx.xxx.xxx.xxx",
      PROXY_PORT => "80",
      YOUR_ID    => "id-goes-here",
      YOUR_PW    => "password-goes-here",
      URL        => "http://www.google.co.uk",
    };
    
    my $ua = new LWP::UserAgent;
    $ua->proxy('http', 'http://'.PROXY.':'.PROXY_PORT.'/');
    
    my $request = HTTP::Request->new(GET => URL);
    $request->proxy_authorization_basic( YOUR_ID, YOUR_PW );
    
    # Make the call
    my $response = $ua->request($request);
    
    print Dumper $response;
    
    if ($response->is_success) {
      print "SUCCESS\n"
    } else {
      print "FAILURE\n"
    }

    
## [Python 3.4][Python]

    #!/usr/bin/python
    
    import urllib.request
    from urllib.error import URLError
    
    # *** PUT YOUR VALUES HERE ***
    PROXY      = "xxx.xxx.xxx.xxx"
    PROXY_PORT = "80"
    YOUR_ID    = "id-goes-here"
    YOUR_PW    = "password-goes-here"
    URL        = "http://www.google.co.uk"
    
    proxy = urllib.request.ProxyHandler({
        "http":
        ''.join(["http://", YOUR_ID, ":", YOUR_PW, "@", PROXY, ":", PROXY_PORT])
    })
    
    auth = urllib.request.HTTPBasicAuthHandler()
    
    opener = urllib.request.build_opener(proxy, auth, urllib.request.HTTPHandler)
    urllib.request.install_opener(opener)
    
    # Make the call
    try:
        connection = urllib.request.urlopen(URL)
        print(connection.read())
        print("SUCCESS")
    except URLError as e:
        print("FAILURE: %s" % e)


[StrawberryPerl]: http://strawberryperl.com/
[Python]: https://www.python.org/downloads