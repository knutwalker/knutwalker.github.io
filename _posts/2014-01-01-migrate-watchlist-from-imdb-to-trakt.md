---
layout: post
title: "Migrate your watchlist from IMDb to trakt.tv"
tags:
- python
- trakt
---

[Trakt](http://trakt.tv/) is an exciting service for managing your movie watchlist, collection, and other lists.
This script can be used to migrate an existing IMDb watchlist over to trakt.

{% gist 3887143 %}

use it like

    python imdb_to_trakt.py -i 0123456 -u traktuser -p traktpasswd -a traktapikey

