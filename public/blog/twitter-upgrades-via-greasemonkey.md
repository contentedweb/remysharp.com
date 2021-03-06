---
title: Twitter Upgrades via Greasemonkey
date: '2008-09-19 11:00:49'
published: true
tags:
  - greasemonkey
  - twitter
  - web
modified: '2014-09-03 16:15:12'
---
# Twitter Upgrades via Greasemonkey

Quickly following on from my greasemonkey script yesterday, [Josh Russell](http://www.joshrussell.com), local Brightonite, [updated](http://www.joshrussell.com/2008/07/18/we-added-search-to-twitter/) his (and [Dave's](http://builtbydave.co.uk/) Twitter search script.

So I've taken it upon myself to a) upgrade his script to work on all pages, not just the home page, and meld together my Twitter scripts with his moving towards (in my best deep Jerry Bruckheimer voiceover voice): The Ultimate Twitter Upgrade Script!

<!--more-->

I've also striped out the code that waited for the DOM to be ready - not sure why I did it in the first place, but it looks like the whole DOM is available to greasemonkey as soon as it hit the code - so it means the lat/long lookup is quicker.

[Get the twitter upgrade](/downloads/tweet_upgrade.user.js) or via [userscripts.org](http://userscripts.org/scripts/show/34004)

For more information on my bits of the script, [have a read of my initial post](/2008/09/17/tweet-offline-better-locations/)
