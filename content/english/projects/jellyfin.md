---
title: 'Jellyfin Media Server'
hideDate: true
draft: false
---

## In short

**TL;DR:** Please **DO NOT** play movies in streaming mode. Download them locally before watching.

## The web interface and my collections

I host a [Jellyfin](https://jellyfin.org/) media server at the following address:

[https://jellyfin.edervieux.fr/](https://jellyfin.edervieux.fr/)

If you are among the lucky chosen ones, I must have given you a username and password to log in. The interface is very similar to Netflix / Amazon Video / Disney+, and other streaming giants.

I have organized my media library into five categories:

  - **Films** for all live-action feature films, including mixes of live-action and animation, such as *Roger Rabbit* or *Cool World*.
  - **Animation** for all animated feature films, whether 2D or 3D.
  - **Courts-métrages** which mixes live-action and animation in short format, often very experimental stuff.

To these first three categories, I have added two more:

  - **Captures de films** which contains screenshots from films that struck me, either because the visuals are particularly stunning, or because it was a powerful moment in the film that I wanted to remember... Feel free to browse for inspiration!
  - **Collections de films** which contains several thematic playlists (mainly genre films and/or specific geographic origins) that are not or poorly handled by Jellyfin. For example, by clicking on the "Films" base, you get access to automatically tagged genres (like "Western", which works well), but there was no sub-genre like "Giallos" or "Sword & Sorcery"!

## Why streaming is NOT recommended?

I currently host this Jellyfin on a very modest hardware (Dell Wyse 5070) that does not allow (or poorly allows) streaming of high-quality sources such as 1080p-2160p x265, which brings the server to its knees. So please do **not** stream the movies, but download them locally before watching.

Still, feel free to use the integrated player for previewing, or to check that the movie is in the right language / has subtitles / is of sufficient quality according to your liking.

## An error, a request?

### The 700 MB DivX is outdated...

This collection of about 900 movies has been gathered over the years from very diverse sources. As a result, some movies obtained in the early 2000s when eMule was a thing are of very poor quality. My apologies for that. If you want to watch a movie and find the quality really bad, write to me and I'll provide a better version.

### ...except for B-movies!

A number of movies, especially some 80s genre films, are absolutely impossible to find in good quality. Typically, the incredible *Alien Platoon* and other post-apocalyptic B-movies unfortunately do not exist in Blu-Ray 4K UHD.

### Identification error

Movie identification, i.e., retrieving the poster and other metadata (director, synopsis, cast, *etc.*) is managed automatically by Jellyfin. I have carefully reviewed and corrected misidentified movies and re-tagged them properly from IMDB or similar when possible, but mistakes can still happen. Don't hesitate to contact me if you notice that a Disney *Cinderella* is actually a 2010s body horror snuff movie!

### But where do you find all this?

I can't recommend [Rarelust](https://rarelust.com/) enough for the quantity and quality of rare films you can't find elsewhere. Otherwise, mainly:

  - RARBG
  - The Pirate Bay
  - YggTorrent
  - RuTracker
  - KinoZal

**Note:** Despite what you may read, these sites are still fully functional as of May 1, 2025, but may be blocked in your country or by your DNS (if so, [see a tutorial here](https://www.zdnet.fr/blogs/infra-net/pour-contourner-le-blocage-des-sites-web-il-suffit-de-changer-de-resolveur-dns-39810881.htm)).

## How did you set up this server?

All the details about setting up my personal server in general—and making a Jellyfin server accessible from the internet, in particular—are available in [this guide](projects/nas_guide#serving-jellyfin-on-the-internet).