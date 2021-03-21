---
title: "Syncing history with Trakt"
date: 2021-03-21
layout: post
tag: guide
---

## TL;DR
This guide will explain scrapping local collection and syncing with Trakt using Trakt API.
Follow the [API docs](https://trakt.docs.apiary.io/) and [my project](https://github.com/madhavtummala/trakt_sync) 
to see how you can achieve this. My code has been tested to work on macOS (both M1 and Intel).

## Some Background
I am a big fan of movies and shows and try to get my hands on anything
that is popular and entertaining.
Thanks to my new internet connection and a new 1Tb external hard disk,
In 2012, I started maintaining a digital collection of all movies I watched
on TV, in Theaters or just from somewhere on the Internet.
In 2016, this collection grew to 500gb, and I had accidentally deleted all of
it in an OS Installation experiment. Instead of stopping there,
I went ahead and downloaded all of it again 😬

I now hold close to 1.5Tb of movies and shows in an external hard disk of 2Tb.
All with properly formatted folder names and files and yes, if you couldn't
tell already, I take pride in useless things like proper formatting. In the age
of streaming, personal collections are becoming obsolete. I can't possibly keep up with the
catalog offered by Netflix or Youtube Movies. But I have an emotional connection
with the collection I have accumulated over years. After all, this was my childhood,
looking up a movie, downloading it and watching it with my family. And again, what could I
possibly do with my monthly 700gb internet.

## Survey
I wanted to document everything I watched, for myself. I started looking at websites that
allow this like IMDb, Letterboxd, Trakt. As a person from IT background, automation was the first
concern for me. Anything I do must be infinitely repeatable with minimal effort.
If the service on the other end has an API, makes it really easy.
* [IMDb](https://www.imdb.com) is quite famous, and upon creating an account, you can add movies or shows to your
  watchlist, watched history and also rate them on a scale of 10. The reviews and parents guide
  are really useful, and it is easy to discover new content. There is an [API](https://developer.imdb.com), but is quite
  restricted in sense that you need to submit an application for building a legit App in order to access it.
* [Letterboxd](https://letterboxd.com) perfectly has what I need. It's marketed as a social
  media site for film lovers. Films(not shows) can be tracked, added to watchlist, recommended to friends.
  There is an open [API](http://api-docs.letterboxd.com), so this website is great.
* [Trakt](https://trakt.tv/) is a website with the primary goal of tracking, both movies and shows. It
  has an open [API](https://trakt.docs.apiary.io/) and more ads compared to Letterboxd. It has some more interesting
  features such as check-in, scrobble but the website feels a little unpolished with more Ads.
* [TMDb](https://www.themoviedb.org/) is not a social media site, but is used as a definitive source
  for identifying movies and shows by many other websites and apps. It has a very famous [API](https://www.themoviedb.org/documentation/api).
  You can create lists and populate your watch history with it. It will not disappear anytime soon and will only get famous as more and more
  services start using the API.

IMDb was already out of scope, since it doesn't have an API I could use. Letterboxd is great
overall for cinema lovers and website looks great with unobtrusive minimal ads, but the entire
thing is *only* for movies. TMDb is a great choice for an API. If I were to start my own
application that tracks movies and shows, I would use TMDb. That said, TMDb is blocked in India,
and I cannot visit the website without a VPN/proxy which is a huge blocker. So, I chose Trakt at the end.

## Trakt API
I will discuss how to use the Trakt API, how to prepare the API post data and scrapping (if you have
a local collection similar to mine), in a proper format, and then we will look at Trakt widgets that
you can embed in other websites. All the code I used is in this project, follow the README to run the code.

### Authentication
