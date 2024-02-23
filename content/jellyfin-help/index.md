+++
date = 2023-10-11T00:13:40Z
description = ""
draft = false
slug = "jellyfin-help"
title = "Jellyfin Help"
searchHidden = true
comments = false
+++

If you are new to Jellyfin or Self-Hosted media in general, this will be a resource to help you get started.

💡 All of the software that makes up this setup are free and open-source. If you have any integrity concerns you can review all of the code on your own. All of the media and Jellyfin are located in-home. Requests, Sign-Up's, Links and Server Status are all located in Oracle Cloud.

## Links

I post all of links on my main links page. They will be updated as things change and are added. This should be the only link you need in reference to this system.

![Links Image](/assets/img/jellyfin-links.png#center)

## Getting Started

First thing you need is a compatible device, most smart TV's and cell phone's are. Then you need the Jellyfin app. You can find download links [here](https://jellyfin.org/downloads/). You should be able to find it just by searching for Jellyfin in your app store. iOS devices can also use the native [Swiftfin](https://apps.apple.com/us/app/swiftfin/id1604098728) app for better compatibility. (Android devices can also download [Findroid](https://play.google.com/store/apps/details?id=dev.jdtech.jellyfin) for a more modern experience than Jellyfin's)

## Sign Up

Obviously the first thing to do is make an account. Click "Jellyfin Sign-up" on the main links page and create an account. This process is free for now, just fill in the form, verify your email and you are good. This process creates a user that can be used on auth website, Jellyfin and Jellyseer.

💡 Email verification is now required.

## Jellyfin

Now that you've made an account, you can login to Jellyfin. I recommend starting on your cell phone before doing your TV, to make it easier to do on your TV. You can skip the cell phone step though, as you already have an account.

![JF Login](/assets/img/jf-login.png#center)

Now just put in your credentials that you just filled in on the signup website, using username and password. (not email)

Now that you are signed in you can start watching on your phone. But to quickly sign in to your TV, put in the server URL and it should give you a Quick Connect code by default.

![JF Quick Login](/assets/img/jf-quick-login.jpg#center)

On you phone's Jellyfin app, Click the Profile icon on the top right of the page, then choose Quick Connect, input your code then click authorize.

![JF Login Code](/assets/img/quick-login-code.png#center)

Your TV should now be logged into your account and ready to start streaming.

## Jellyfin Tips

1. Some more things I do right away is organize the way the libraries are presented to me.

You can do this under **Profile - Home - Library Order**. This way you don't scroll past irreverent libraries constantly to find what you watch most. (for me Requests, Movies and TV always top 3.)

![JF Library](/assets/img/jf-library.png#center)

2. By default on the TV application, when in a library, the poster size is a bit too small and set to horizontal which makes it feel off from regular streaming services.

![JF Display](/assets/img/jf-display.jpg#center)

To fix this, just click the settings on the top right, then change the Image Size to X-Large and the Grid Direction to Vertical. Pictured below is defaults.

![JF-Settings](/assets/img/jf-settings.jpg)

Once corrected, it should look this

![JF-Corrected](/assets/img/jf-corrected.jpg)

You will have to do this to each library unfortunately.

## Requesting

An awesome feature I've implemented into my Jellyfin sever is requesting. All users created on Jellyfin are also created on Jellyseerr, the software I run to handle requests. You can find the link to Jellyseerr on the main links page.

Movies are set to automatically approve, which means any movie you request should immediately start downloading on my end, assuming my systems can find the movie. Assuming it's a relatively popular movie, it should download and be available on Jellyfin within 5 minutes. There are caveats and things that can interfere with this process but it remains consistent. All movies should download in 1080p. (old videos in lower resolution may not download, if it shows processing for more than 30 mins, it will notify me and I will manually find)

Requester gets email when Jellyfin recognizes the file changes.

TV Shows on the other hand, due to their usually high episode and season count, are set to manually approve. I rarely approve TV shows due to this reason.

All requested content gets automatically deleted 7 days after requester watches in full, to avoid this from happening if you wish to re-watch later, add the specific movie or TV show to your Favorites.

You can only use a web browser to request content (unless you join discord server). However Android devices have an option to Install as an app from their browser. To do this first load the request website on your device, once loaded click your browser's options menu, generally 3 dots on the top/bottom right of the screen. Then choose install app.

![Android app install](/assets/img/android-install.png#center)

## Requesting through Discord

Rarely used but still there is my Discord server for requesting content, not as pretty as Jellyseerr, but works the same.

The currently available commands are:
/request tv
/request tv-tvdb
/request movie
/request movie-tmdb
/help
/ping

Here's how to use a command
/request movie Deadpool 2

## Common Jellyfin Errors

Giving up...Too many Errors

This error seems to be caused by the default media player Jellyfin uses on most TV's.

To fix this, go to your Jellyfin Home page, (landing page when your first open app) then **Settings** - **Playback** - under **Video** click **Preferred media player**. By default this is set to LibVLC, change this to Automatically choose as pictured.

![JF Media Player](/assets/img/jf-media.jpg)

Go back and try to watch the same video and it should work, if not you can download [Just Video Player](https://play.google.com/store/apps/details?id=com.brouken.player&hl=en_US&gl=US) and set external app as Preferred Media Player to Just player app. I havent had a single issue since doing this. VLC also works.