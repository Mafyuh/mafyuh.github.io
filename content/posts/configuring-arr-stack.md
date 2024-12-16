---
date: 2024-02-25T00:13:40Z
description: ""
draft: false
slug: "arr-stack-config-guide"
title: "Configuring Arr Stack"
showtoc: true
tocopen: true
tags:
  - "Homelab"
comments: true
---

This is blog post 2/2 on setting up a Arr stack using docker. This post will touch on configuring these services and what I have learned when using these services. I will be referring to [TRaSH-Guides](https://trash-guides.info/) often in this post. It's like the bible of the Arr's so I would look there for more options, especially if your setup differs from mine.

# Prerequisites
- Any Usenet Server Subscription (preferred)
- Any Usenet Indexer Subscription (preferred)
- Real-Debrid Subscription (if you like torrents being fast)
- VPN Subscription (Bare minimum needed to download torrents)

# Configuration
## Prowlarr
We will first start with Prowlarr and Sabnzbd to get all of this out of the way, and it's the part where you're gonna spend some $

1st thing to do is load up your web browser and go to Prowlarr. In your URL bar put http://{IP-ADDRESS}:9696.

You should be prompted to create an account. It doesn’t matter much if you choose Basic or Forms for authentication.

This guide is just going to be using Usenet, as I recommend it over torrenting. 

Prowlarr's job is to search thru all the indexers and return those results over to Radarr/Sonarr.
### Indexers
You can find links to all the indexers out there on this Usenet [subreddit](https://www.reddit.com/r/usenet/wiki/indexers/). I will provide a screenshot of my Indexers list:

![Indexer List](/assets/img/indexer-list.png)

I would say the most downloaded stuff I have is from NZBgeek, followed by DrunkenSlug then NZBFinder. If I were to only be able to have 1 it would be NZBgeek for sure.

Sign up for whatever service you want, once you pay they should give you your API key, usually under profile on the indexer website. Take that API over to Prowlarr and you should be able to select your indexer from the list, paste in your API key and save. You can also click on the wrench icon next to the indexer and set your VIP expiration date, so Prowlarr reminds you to renew. 

I paid for as lifetime altHUB account as well as yearly for Slug and Geek, A lot of the others that are under Interactive Search Sync Profile are free accounts that they limit the amount of API hits per day. You can tell Prowlarr this to by setting up Sync Profiles like I have so it doesn't burn your API hits fast. [Link](https://trash-guides.info/Prowlarr/prowlarr-setup-limited-api/)

Thats pretty much it for indexers, now you just need a provider which is covered in the Sabnzbd portion.

### Usenet vs Torrenting
Usenet wins all day IMO. I started off with torrenting years ago and there would always be something I couldn't find. Once I found usenet that problem went away and hasn't returned. I still have a Real-Debrid subscription, however I am not planning on renewing when it runs up as I just don't need torrents anymore for media. The only exception being PPV fights, which at least with the indexers I use, I cant find much on usenet. Usenet grabs about 95% of what I need. And I'm sure the other 5% would have still been found, but torrent won the algorithm battle. You also do not need to use a VPN when using Usenet as all big providers use HTTPS.

If you are new to Usenet, their [subreddit](https://www.reddit.com/r/usenet/wiki/index/) wiki will help you out. Unlike torrenting which is peer to peer and fully decentralized, usenet is more centralized and has many servers that host the content (Providers). There is a monthly fee for being able to access these servers. You also need an indexer in order to find NZB files, which is then sent to your provider for downloading. Indexers are much cheaper at usually a few bucks a year.

### Connecting Prowlarr to Radarr/Sonarr
For now skip to setting up Radarr and Sonarr, after setting up you will need their API keys which can be found under **Settings - General - API Key**.

In Prowlarr go to **Settings - Apps - Create an application** and click Radarr, filling in all api key and changing the host if needed. Do the same thing for Sonarr. This automatically syncs all your indexers into Radarr/Sonarr and if you want to add more indexers you just have to add it in Prowlarr and not 2 separate places.
## Sabnzbd
Sab is what connects to our Usenet providers and downloads the NZB files that Radarr/Sonarr gave to it.

First load up sab at http://{IP_ADDRESS}:8080 . One of the first things it has you do is enter you server details, which takes us into providers.
### Providers
These are the costliest part of the process, although for good reason. There are a bunch of providers out there, which all can be found [here](https://www.reddit.com/r/usenet/wiki/providers/).

I have used Newshosting, NewsDemon and UsenetNow. You only need 1 of these to download most stuff, but I prefer to have 2 as I have seen some servers don't have a file that another one did. It's rare but happens. As of the time of me writing this I am using UsenetNow for $6/month and Newsdemon $50 for 12 months using this [link](https://members.newsdemon.com/billinginfo.php?pricepointid=20230413) (Not referral just found a promo link on Reddit I saved)

Once you select your provider and pay, they should email you your login credentials that you just put into Sab. Remember to set the connections to whatever your provider allows to maximize download speeds.

### Folders
All we really need to do to configure Sab is to set the Temp and Completed Download folders. 
For Temporary:
```
/data/usenet/incomplete
```
and for complete:
```
/data/usenet/complete
```
Make sure you save changes.
## Radarr
### Initial Setup
1st thing to do is load up your web browser and go to Radarr. In your URL bar put http://{IP-ADDRESS}:7878 . 

Putting in the IP of you Ubuntu machine. If you are using something like WSL just use localhost as IP.

You should be prompted to create an account. It doesn't matter much if you choose Basic or Forms for authentication. 

If you are starting fresh with no content, go to **Settings - Media Management - Root Folders** and add a root folder, you can use the movies folder that was located at /data/media/movies or you can go into /data/media and name your folders however you want to. 

My media directory breaks down the movies by category: 4K, Movies, Marvel, DC, Kids, Stand-Up, Requested Content and Fights. They are added to Radarr individually as follows:

![Root Folders](/assets/img/radarr-rootfolder.png#center)

If you already have media you want to import into Radarr, click **Movies - Library Import - Start Import**. Remember all you files are going to be under /data/media if you followed my installation guide. This also adds the chosen directory as a root folder.
### Connect Sab to Radarr
To connect Sab to Radarr, in Radarr go to **Settings - Download Clients** - add - Sabnzbd - fill in all your Sab details, getting the API key from **Sab - General - Api Key** (Not NZB Key)

### Naming Files
It's best practice to rename your files, Radarr does this automatically for you. To do this go to **Settings - Media Management - Movie Naming**. Make sure you check the Rename Movies checkbox. Everything should be fine by default besides Standard Movie Format, Jellyfin recommends this:
```
{Movie CleanTitle} {(Release Year)} [imdbid-{ImdbId}] - {Edition Tags }{[Custom Formats]}{[Quality Full]}{[MediaInfo 3D]}{[MediaInfo VideoDynamicRangeType]}{[Mediainfo AudioCodec}{ Mediainfo AudioChannels]}[{Mediainfo VideoCodec}]{-Release Group}
```
Make sure you hit save. 

Note this only automatically renames the file if it was downloaded from Radarr, if you imported your own media you need to manually click rename files under **Movies** - Click Edit Movies - Select All - Rename Files on bottom of screen.

### Custom Formats 
Every single one of these custom formats is from trash-guides, you can see how to import them [here](https://trash-guides.info/Radarr/Radarr-import-custom-formats/).
![Custom Formats](/assets/img/radarr-customformats.png)

Add all custom formats that you want to look out for, whether good or bad. What custom formats do is essentially put a label on files just based off the files name, as well as provide these labels a score which is used when auto-searching for a movie. Just adding custom formats on it's own does nothing, setting up a Quality profile to rank these custom formats is how we pick and choose what we want. Reading through trash guides will yield good results.

### Quality Profiles
Now you need to ask yourself what your plan here is, do you want the highest possible quality content with no regards to storage space? Do you want a giant library filled with lower quality content? Do you have a 4K TV? Do you have HDR/HDR10/Dolby Vision on your TV? Do you have a Dolby Atmos system and want the top notch audio? What about the people you maybe plan on sharing these files with, do you know their TV's capabilities? Are you gonna be using Plex or Jellyfin to stream this media?

You should consider all these things when setting this up so you don't have to manually search for files. That's the point of this whole section is to automate the searching process and find the correct kinds of files every time. Trash-Guides put together multiple flow charts on what custom formats to use in certain situations which can be found [here](https://trash-guides.info/Radarr/radarr-setup-quality-profiles/#which-quality-profile-should-you-choose)

I am going to put my logic behind how I have my setup configured, although you should review [this Trash-Guide](https://trash-guides.info/Radarr/radarr-setup-quality-profiles/) for more details.

1080P:
![1080p Quality Profile](/assets/img/hd-radarr.png)
*Not Shown custom format scores
- 3D:-10000
- BR-DISK:-10000
- Bad Dual Groups:-10000
- EVO (no WEBDL):-10000
- LQ (Release Title):-10000
- x265 (HD):-10000 (This is Trash-Guides Golden Rule, 1080p=x264 | 4k=x265)
  
Not much logic here other than not getting crappy releases and only downloading x264 if its 1080P. File size limits can be put to get smaller files. I just follow [this](https://trash-guides.info/Radarr/Radarr-Quality-Settings-File-Size/). 

I want x264 as its gonna have the highest compatibility with the most devices and the less transcoding I need to do the better.

4K:
![4K Quality Profile](/assets/img/4k-radarr.png)
![4k Radarr 2](/assets/img/4k-radarr-2.png#center)

I have a 4K TV with Dolby Vision Support, along with a Dolby Atmos sound system. So making sure my 4K files have Dolby Vision is pretty important, also having the extended screen of IMAX on a home sized TV is amazing. Having both of those together is the perfect recipe IMO, yet not all movies support one or the other so not as common as you may think. But generally if I want something 4K it will have DV support. I read on trash guides that if you have DV files without DV HDR10 as a fallback, the video will appear off-color to TVs that don't have DV. So I try to make sure DV with HDR10 fallback is 1st. File size is not taken into consideration at all, I have multiple ~90GB movies. 

Also if your TV doesn't have DV support, get one. In my experience the difference between HDR10/10+ and DV is night and day. And I usually am not a fan of proprietary software at all. The next best if you don't have DV is HDR10+. But ultimately having any HDR is gonna be better than SDR.

I ended up removing the Atmos custom format as most of the time the insane high quality files will have Atmos, and it could just be my setup but I don't notice too much a difference between DD+ and Atmos.
### Conclusion
With all this done you should be able to a add a new movie and watch it download over in Sab. Then when finished downloading it should be automatically moved to your target directory.
### Tips
1. If you ever find a file that works well and you don't want Radarr to mess with it, unmonitor it. That way Radarr doesn't upgrade it.
2. If you are brand new to Radarr I would learn the basics of it in the web-ui, then proceed to utilizing the API with apps like [Nzb360](https://nzb360.com/), [LunaSea](https://www.lunasea.app/), and [Overseerr](https://overseerr.dev/)/[Jellyseerr](https://github.com/Fallenbagel/jellyseerr) to request stuff with a better front end. There's even [Doplarr](https://docs.linuxserver.io/images/docker-doplarr/#application-setup) the Discord bot.
3. Say you found a specific version of a movie, but the audio is bad or a different language, and you can't find another video quality the same, you can use a tool called [mkvtoolnix](https://mkvtoolnix.download/) to merge audio tracks from one file to another's video track. You will probably run into this at some point if you're really looking for something specific.
4. You can manually import Boxing/WWE/UFC events into Radarr and the metadata will apply, but searching in Radarr doesn't work for these types of events. Manually finding the NZB's or torrents and moving files seems to be only way.
## Sonarr
### Initial Setup
Sonarr is Radarr but for TV Shows.

First thing load up Sonarr on your web browser at http://{IP_ADDRESS}:8989 .

Make an account. Then do the same thing as Radarr and add a new root folder if starting new, or import your existing media. The root folder location should be /data/media/tv 

Then we need to connect Sab to Sonarr, to do this Go to **Settings - Download Clients - Add** just like we did with Radarr, fill in all your Sab details.
### Custom Formats
Again all these custom formats are on trash-guides. Link [here](https://trash-guides.info/Sonarr/sonarr-collection-of-custom-formats/).
![Sonarr Custom Format](/assets/img/sonarr-cf.png)
I just picked all major streaming services and the Tier's. Import them the same way you did Radarr.
### Quality Profile
Go to **Settings - Profiles - Quality Profiles**
![Sonarr 1080](/assets/img/sonarr-1080p.png)
*Not Shown custom format scores
- x265 (HD):-10000
- BR-DISK:-10000

I only have this 1 1080p profile as I personally do not want to waste that storage space on a long TV series. I do not have any 4K TV shows at all. This generally always grabs the best available file. Again for Size Limit I follow [this trash-guide](https://trash-guides.info/Sonarr/Sonarr-Quality-Settings-File-Size/).

Having x265(HD) as -10000 means it will not download a 1080p file if it is encoded in x265, again this is for compatibility as x264 just works on more devices.

### Naming Files
I also rename Sonarr's files, to do this go to **Settings - Media Management** - check Rename Episodes and under Standard Episode format set:
```
{Series TitleYear} - S{season:00}E{episode:00} - {Episode CleanTitle} [{Custom Formats }{Quality Full}]{[MediaInfo VideoDynamicRangeType]}{[Mediainfo AudioCodec}{ Mediainfo AudioChannels]}{[MediaInfo VideoCodec]}{-Release Group}
```
Make sure you save changes.
### Tips
1. Don't download a whole series at once if it has many seasons, you may fill up on space on your VM. Which will stop everything on that VM. Usually they download faster than they transfer to target directory. So space can add  up quick. I usually do 2-3 seasons at a time.

## Bazarr
Bazarr is used to get subtitles for all your content. Sometimes the files come with subtitle tracks, Bazarr covers you when they don't.
### Initial Setup
First thing load up Bazarr on your web browser at http://{IP_ADDRESS}:6767 .

First thing is to set your language, go to **Settings - Languages - Languages Filter** and set the filter to your language. I set to English.

Then under Language Profiles click Add New Profile:
- Name: English
- Click Add Language, english should pop up by default.
- Set the cutoff to en
- Save

At the bottom of this page there is Default Settings, check both boxes for Series and Movies and choose your language profile.
### Providers
Under **Settings - Providers** - Add a Provider:

![Bazarr Providers](/assets/img/bazarr-prov.png)

YIFY, TVSubtitles, Supersubtitles, are all free and dont require an account.

opensubtitles.com requires you to create an account first but is free.

I have a OpenAI whisper model running on a separate VM which uses my GPU and AI to generate subtitles for content as well, And its pretty good even with the base model. It's rarely needed as usually I find better subtitles thru another provider first with a higher score. But for those times when subtitles can't be found its nice they can be generated. I found [this](https://wiki.bazarr.media/Additional-Configuration/Whisper-Provider/) in Bazarr's docs.

Make sure you save your changes.

### Connecting Bazarr to Radarr/Sonarr
Now you just need to tell Bazarr where your arr's are located. Go to **Settings - Sonarr** for Sonarr and **Settings - Radarr** for Radarr. Filling in all your details and saving.


## Conclusion
Congrats you now have a fully automated backend for downloading media! Good time to cancel those 10 streaming subscriptions and start downloading what you wanna watch. There's not many guides out there about this sort of thing, as piracy leaves some ethical concerns. But idrc, I've been a pirate my whole life, the fact that you can make a system like this all for way cheaper than streaming services is mind-blowing to me.

Now just hook up Jellyfin/Plex up to you /data/media directory and start watching with no ads!