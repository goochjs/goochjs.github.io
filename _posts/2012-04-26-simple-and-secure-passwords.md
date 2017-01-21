---
layout: post
title: Simple and Secure Passwords
modified:
excerpt: "A memorable approach to varied passwords"
comments: true
tags: [tech,security]
image:
  feature: password.jpg
  credit: Marc Faladeau
  creditlink: https://www.flickr.com/photos/49889874@N05/
---


After a pre(r)amble, this post is intended to provide a system through which you can generate passwords for your various on-line accounts that are:-

* memorable
* (adequately) secure
* (generally) unique

This will be wonderfully redundant once we've sorted out federated identity management, [biometrics][biometrics] and associated whatnots ... but I recommend you don't hold your breath.

[Click here to cut to the chase](#the-algorithm)...

## Preamble

I'm sure no one reading this has a password based on their pet's name or containing their birthday, or has the same password across multiple accounts.  I'm sure everyone reading this has a mixture of lower and upper case letters and numbers in their passwords.  You've obviously never written a password down on a post-it note and put it in your desk drawer.  Of course, you also have all of your photos backed up on secure media kept somewhere other than in your house.  You've probably never left your wallet on top of the car or gone away for the weekend leaving the front door open.  And you brush your teeth twice a day, every day.  Don't you?

A couple of months ago, my email account was hacked and a number of friends and colleagues sent spam.  The same thing seems to happen to someone else in my address book every few weeks.  Beyond the annoyance and having to apologise to everyone to whom you've just advertised drain-cleaner-based pharmaceuticals, there are many concerns here.  If it is your main email account, then it's undoubtedly the one you use to authenticate you or to safeguard all manner of other services - Facebook, Twitter, Wordpress, LinkedIn etc etc.  Your reputation could easily be trashed or, if it's related to your Paypal account, you could easily find yourself significantly out of pocket.

In my case, my password was all lower case and was made up of concatenated dictionary words (I'm not going to tell you what it was but, believe me, it was childish and made me smirk).  These sorts of passwords are susceptible to simple brute force attacks - the miscreant starts at "A" and tries every combination of words in the dictionary plus all of the passwords published on [sites like this][passwordsite].  One similar to mine can allegedly be cracked in [around thirteen minutes][howsecureismypassword] (NB see [footnote 2](#footnotes)).

Furthermore, I used to use the same password on more than one site.  I had a small handful of different passwords and would use one for shopping sites, one for social sites, etc.  The [Gnosis hack of Gawker][gnosishack] showed how dangerous this can be - break one and someone will try it against one of your other accounts.

But we can't remember unique passwords for every site, can we?  I work in IT and have seen cases where users have been expected to manage up to **forty** different log-ons.  Obviously, this was within an IT security framework demanding each was unique, non-dictionary, mixed upper and lower case, etc etc etc.  It's nothing more than a recipe to force people to write the passwords down.

On that topic, someone close to me (who will remain nameless) was struggling to log into Amazon.  He was adamant that he was typing his password in correctly - he'd even checked it in his filofax (yes, a *filofax* - how quaint).  I said, "so it's not this word written next to the word 'Amazon' here, is it?"

"No", he said.

But it was.

So, what is the alternative?  You could get a [password safe][passwordsafe] on your mobile.  Or you could try the following...


## The Algorithm

1. Take your favourite song lyric, song title, nursery rhyme, television programme, etc.  Something of around four or five words.
2. Write down the initial letters, capitalising the last one.
3. Add a number, any number that has relevance to you.  Something around two or three digits.
4. Take the initial letter of the application or site for which you're generating a password.  Add it to the end.
5. That's it.  You now have a basically gibberish but memorable password of around eight or nine characters, mixing upper and lower case with numbers, thereby satisfying 99% of applications' security requirements, and which is relatively unique across your many services and accounts.

So, if I take "*Mary had a little lamb*" and the year of my birth, my Wordpress password would be:-

    mhalL72W

All you need remember is the mnenonic and the number.  Nothing needs to be written down and, if we put it through the [same password strength tester][howsecureismypassword], we have ten days before a brute force attack will be successful. This is more than enough for our delinquent hacker to give up and go elsewhere.

Remember, you don't need to outrun the bear - you just need to run faster than someone else.  Speaking of which, can I suggest a step (6)?

* Add your own spice to the rules above.  Perhaps take the first and third letters of the app/site's name, reverse the capitalisation or add an ampersand to the end.

I'm sure you get the idea...


## Footnotes

1. In the spirit of transparency, the algorithm wasn't my idea.  I can't recall where I came across it but, if I do, I will happily give all due credit.
2. Don't put your real password into a password strength testing website.
3. No, this won't really sort out our work passwords that need changing every 90 days.  We'll have to live with those.  Personally, I wonder whether policies requiring a password to be changed periodically materially improve system security.


[biometrics]: http://www.digitaltrends.com/mobile/ibm-forecasts-biometric-passwords-mind-reading-tech-for-2016/
[passwordsite]: http://vijayamfoundation.org/blueolivetech/slack/pass1
[howsecureismypassword]: http://howsecureismypassword.net/
[gnosishack]: http://www.itpro.co.uk/629401/gawker-passwords-pilfered-in-server-hack
[passwordsafe]: http://www.androidzoom.com/android_applications/password+safe