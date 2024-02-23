+++
date = 2023-11-11T16:20:00Z
description = ""
draft = false
slug = "how-to-automate-jellyfin-issue-handling"
title = "How To Automate Jellyfin Issue Handling"
tags = ["Jellyfin"]
+++

I wanted a way to automate when users tell me a video on my Jellyfin server has an issue. After alot of trial and error, ChatGPT, Bard and I came up with this automation.

## Requirements

My only requirements when making this was that it was free and self-hostable. Not even any NPM extensions are required in AP.
Actual Software requirements are:

1. Sonarr
2. Radarr
3. Overseerr/Jellyseerr

Optional

1. SMTP server or ability to send SMTP messages (can also use discord)
2. ActivePieces or any other automation platform that supports TS. (Zapier, n8n, etc)

Here's a great AP setup and how-to video:

{{< youtube "kVaCDc2CXik?si=geKXwgRjU6onDyjS" >}}

Note: I didn't do any of the ngrok stuff. I just have Nginx Proxy manager setup with a wildcard certificate. Then just give a domain name and point and its ip:8080. No special Nginx config needed. Make sure you set AP_FRONTEND_URL in .env

This blog post is rather long, if you prefer to see the code on git you can find all this code [here](https://git.mafyuh.dev/mafyuh/ActivePieces).

## How it Works

Whenever a user Reports an Issue in Jellyseerr, a webhook is sent to activepieces (AP) with the Issue data, this triggers the automation to mark as failed, delete file, re-search, refresh Jellyfin Libraries and Resolve the original issue with comment. There is an optional feature to approve or deny the automation.

Works across Radarr and Sonarr, as the issue reported can be either Movie or TV show.

Only caveat is if the issue is an entire Season , we just mark the issue as resolved and leave a comment saying to submit an issue for each episode individually

Works on my Jellyfin, Jellyseer, Radarr and Sonarr setup. I dont use Plex but all you would have to change is the Jellyfin Refresh Library Request to match Plex's equivalent.  

Here is a pic of the full automation.

![Automation Image](/assets/img/automation.png)

Everything of value is logged to the console so check there for errors. Lets start breaking it down.

## #1 Jellyseer Issue Reported

First thing is create a flow in AP, select a trigger, and search for webhook. This will give you the webhook URL for Jellyseerr.
Next, in Jellyseerr, under **Settings - Users - Default Permissions** make sure **Report Issues** is checked and save changes.
Then under **Settings - Notifications - Webhook** make a webhook notification, with the URL from AP, and just enabling **Issue Reported** and **Issue Reopened**.
This should look as follows (dont worry about my payload showing mediaId, this has since been deleted)

![Jellyseer Image](/assets/img/seer.png)

Here is my full JSON payload just in case:

```json
{
    "notification_type": "{{notification_type}}",
    "event": "{{event}}",
    "subject": "{{subject}}",
    "message": "{{message}}",
    "image": "{{image}}",
    "{{media}}": {
        "media_type": "{{media_type}}",
        "tmdbId": "{{media_tmdbid}}",
        "tvdbId": "{{media_tvdbid}}",
        "status": "{{media_status}}",
        "status4k": "{{media_status4k}}"
    },
    "{{request}}": {
        "request_id": "{{request_id}}",
        "requestedBy_email": "{{requestedBy_email}}",
        "requestedBy_username": "{{requestedBy_username}}",
        "requestedBy_avatar": "{{requestedBy_avatar}}",
        "requestedBy_settings_discordId": "{{requestedBy_settings_discordId}}",
        "requestedBy_settings_telegramChatId": "{{requestedBy_settings_telegramChatId}}"
    },
    "{{issue}}": {
        "issue_id": "{{issue_id}}",
        "issue_type": "{{issue_type}}",
        "issue_status": "{{issue_status}}",
        "reportedBy_email": "{{reportedBy_email}}",
        "reportedBy_username": "{{reportedBy_username}}",
        "reportedBy_avatar": "{{reportedBy_avatar}}",
        "reportedBy_settings_discordId": "{{reportedBy_settings_discordId}}",
        "reportedBy_settings_telegramChatId": "{{reportedBy_settings_telegramChatId}}"
    },
    "{{comment}}": {
        "comment_message": "{{comment_message}}",
        "commentedBy_email": "{{commentedBy_email}}",
        "commentedBy_username": "{{commentedBy_username}}",
        "commentedBy_avatar": "{{commentedBy_avatar}}",
        "commentedBy_settings_discordId": "{{commentedBy_settings_discordId}}",
        "commentedBy_settings_telegramChatId": "{{commentedBy_settings_telegramChatId}}"
    },
    "{{extra}}": []
}
```

You should be able to Report an issue on a random movie in Jellyseerr and then go to the webhook trigger and choose Generate sample data, and you should be able to see the data from the request. I recommend doing this and creating an issue for an example movie, TV series( All Seasons) and a TV Series (1 Season)

## (Optional) #2 Create Approval Links

In AP add the next step and search Approval, then create approval links.

## (Optional) #3 Send Email

This is so I can either approve or deny the file from being deleted. Maybe it's a client issue and I know for a fact my file is good and I dont want deleted. Thus the links are sent to me along with the some data from the request, so I know what I am approving/denying.

You can use the core SMTP feature but its limited to text. I wanted some more customizability so I chose Resend (supports html) and set up an acct there with one of my extra domains.

You don't have to use email, you can use Discord, SMS, any generic http request or whatever you want. I just use email since I pay for my domains and pay Proton Mail for emails so might as well use em.

Not gonna get too into this, I dont care too much about it atm, customize your email to your liking, but I'll post my somewhat working HTML body here. I literally just copied what Bard gave me, added in data from response and tested and said looks good enough, glitches on my mobile too.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Jellyseerr Issue Reported</title>
<style>
body {
  font-family: sans-serif;
  margin: 0;
  padding: 0;
  background-color: #222;
  color: #fff;
}
.container {
  width: 80%;
  margin: 0 auto;
  padding: 20px;
  background-color: #333;
  border-radius: 10px;
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
}
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 20px;
  border-bottom: 1px solid #555;
}
.header h1 {
  font-size: 24px;
  font-weight: bold;
  margin: 0;
  color: #fff;
}
.header img {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  object-fit: cover;
}
.content {
  margin: 0 auto;
  text-align: center;
}
.issue-subject {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 10px;
  color: #fff;
}
.issue-message {
  font-size: 16px;
  line-height: 1.5;
  margin-bottom: 20px;
  color: #ccc;
}
.issue-image {
  width: 100%;
  height: auto;
  margin-bottom: 20px;
}
.buttons {
  display: flex;
  justify-content: space-between;
}
.button {
  background-color: #007bff;
  color: #fff;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  text-decoration: none;
}
.button:hover {
  background-color: #0056b3;
}
.disapprove-button {
  background-color: #dc3545;
  color: #fff;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  text-decoration: none;
}
.disapprove-button:hover {
  background-color: #bd2830;
}
</style>
</head>
<body>
<div class="container">
<div class="header">
<img src="https://your-logo-url" alt="Jellyseerr Logo">
<h1>Jellyseerr Issue Reported</h1>
</div>
<div class="content">
<div class="issue-subject">
Jellyseerr Issue Reported
</div>
<div class="issue-message">
This issue was submitted by 1. Jellyseerr Issue Reported body issue reportedBy_username.
<br>
The reason for the issue:1. Jellyseerr Issue Reported body message
<br>
Please review the issue and take appropriate action.
<br>
<img src="  1. Jellyseerr Issue Reported body image  ">
</div>
<div class="buttons">
<a href="2. Create Approval Links approvalLink  "><button class="button">Approve</button></a>
<a href="2. Create Approval Links disapprovalLink  "><button class="disapprove-button">Deny</button></a>
</div>
</div>
</div>
</body>
</html>
```
And here's what an email looks like:

![Email Look](/assets/img/email.png)

## (Optional) #4 Wait for Approval

Pauses flow until I approve or deny.

## #5 Radarr/Sonarr Branch

As stated previously, I wanted this to work regardless if Movie or TV show. So using the core Branch feature we just say that if the media_type value from the issue contains the text movie, its true.

![Radarr AP Image](/assets/img/radarr-branch.png)

## #6 Radarr Mark As Failed

All I do here is the Code function with 1 input which is the whole body message of the request, this is assigned to inputs.issue in the code (CASE SENSITIVE)

![Radarr AP Image](/assets/img/radarr-mark-as-failed.png)

Here is the code. Just replace api key and base url:

```typescript
export const code = async (inputs) => {
  const issueSubject = inputs.issue.subject;
  const movieNameRegex = /(.*)\s\((\d{4})\)/;
  const match = movieNameRegex.exec(issueSubject);

  if (match) {
    const movieName = match[1];
    const year = match[2];
    const tmdbId = inputs.issue.media.tmdbId;

    console.log(`Movie name: ${movieName}`);
    console.log(`Year: ${year}`);
    console.log(`TMDB ID: ${tmdbId}`);

    // Define your Radarr API key and base URL
    const radarrApiKey = 'your-api-key'; // Replace with your Radarr API key
    const radarrBaseUrl = 'https://radarr.example.com/api/v3/';

    // Define a function to make API requests to Radarr
    const makeRadarrRequest = async (endpoint, method = 'GET') => {
      const apiUrl = radarrBaseUrl + endpoint;
      console.log(`Calling Radarr API: ${apiUrl}`);

      const response = await fetch(apiUrl, {
        method,
        headers: {
          'X-Api-Key': radarrApiKey,
        },
      });

      if (response.ok) {
        return await response.json();
      } else {
        console.error(`Radarr API request failed: ${response.statusText}`);
        return null;
      }
    };

    // Use Radarr's API to look up the movie by TMDB ID
    const radarrApiResponseData = await makeRadarrRequest(`movie?tmdbId=${tmdbId}`);

    if (radarrApiResponseData && radarrApiResponseData.length > 0) {
      const movieId = radarrApiResponseData[0].id; // Get the Radarr ID of the first movie
      console.log('Radarr Movie ID:', movieId);

      // Use the Radarr movie ID to get the history of the movie
      const historyApiResponseData = await makeRadarrRequest(`history/movie?movieId=${movieId}`);

      if (historyApiResponseData && historyApiResponseData.length > 0) {
        const historyId = historyApiResponseData[0].id; // Get the history ID
        console.log('History ID:', historyId);

        // Use the history ID to mark the movie as failed
        const markFailedResponse = await makeRadarrRequest(`history/failed/${historyId}`, 'POST');

        if (markFailedResponse) {
          console.log('Movie successfully marked as failed.');
        } else {
          console.error('Failed to mark movie as failed');
        }
      } else {
        console.error('No history found for movie ID:', movieId);
      }
    } else {
      console.error('No movies found for TMDB ID:', tmdbId);
    }
  }
};
```
## #7 Delay 5 seconds
Give time to process.
## #8 Delete Movie File
I didn't want to delete the actual movie from Radarr, but just the file itself, thus alot of trial and error, but a working script.
All I do here is the Code function with 1 input which is the whole body message of the request, this is assigned to inputs.issue in the code

```typescript
export const code = async (inputs) => {
  const issueSubject = inputs.issue.subject;
  const movieNameRegex = /(.*)\s\((\d{4})\)/;
  const match = movieNameRegex.exec(issueSubject);

  if (match) {
    const movieName = match[1];
    const year = match[2];
    const tmdbId = inputs.issue.media.tmdbId;

    console.log(`Movie name: ${movieName}`);
    console.log(`Year: ${year}`);
    console.log(`TMDB ID: ${tmdbId}`);

    // Define your Radarr API key
    const radarrApiKey = 'your-api-key'; // Replace with your Radarr API key
    const radarrBaseUrl = 'https://radarr.example.com/api/v3';

    // Use Radarr's API to look up the movie by TMDB ID and get the Radarr ID
    const radarrApiUrl = `${radarrBaseUrl}/movie?tmdbId=${tmdbId}`;
    console.log('Calling Radarr API to look up the movie...');

    const radarrApiResponse = await fetch(radarrApiUrl, {
      method: 'GET',
      headers: {
        'X-Api-Key': radarrApiKey,
      },
    });

    if (radarrApiResponse.ok) {
      console.log('Radarr API lookup successful.');
      const radarrApiResponseData = await radarrApiResponse.json();

      if (radarrApiResponseData.length > 0) {
        // If the response is an array, you should loop through the results
        // and access the Radarr ID for each movie.
        for (const movie of radarrApiResponseData) {
          const radarrMovieId = movie.movieFile.id;
          console.log('Radarr Movie ID:', radarrMovieId);

          // Use the Radarr movie ID to delete the corresponding movie file
          const deleteMovieFileUrl = `${radarrBaseUrl}/movieFile/${radarrMovieId}`;
          console.log(`Calling Radarr API to delete movie file: ${deleteMovieFileUrl}`);

          const deleteMovieFileResponse = await fetch(deleteMovieFileUrl, {
            method: 'DELETE',
            headers: {
              'X-Api-Key': radarrApiKey,
            },
          });

          if (deleteMovieFileResponse.ok) {
            console.log(`Movie file successfully deleted.`);
          } else {
            console.error(`Failed to delete movie file: ${deleteMovieFileResponse.statusText}`);
          }
        }
      } else {
        console.error('No movies found for TMDB ID:', tmdbId);
      }
    } else {
      console.error('Radarr API lookup failed:', radarrApiResponse.statusText);
    }
  }
};
```
## #9 Delay 5 seconds
## #10 Search in Radarr
Researches for movie just deleted.

All I do here is the Code function with 1 input which is the whole body message of the request, this is assigned to inputs.issue in the code

```typescript
export const code = async (inputs) => {
  const issueSubject = inputs.issue.subject;
  const movieNameRegex = /(.*)\s\((\d{4})\)/;
  const match = movieNameRegex.exec(issueSubject);

  if (match) {
    const movieName = match[1];
    const year = match[2];
    const tmdbId = inputs.issue.media.tmdbId;

    console.log(`Movie name: ${movieName}`);
    console.log(`Year: ${year}`);
    console.log(`TMDB ID: ${tmdbId}`);

    // Define your Radarr API key
    const radarrApiKey = 'your-api-key'; // Replace with your Radarr API key
    const radarrBaseUrl = 'https://radarr.example.com/api/v3'

    // Use Radarr's API to look up the movie by TMDB ID and get the Radarr ID
    const radarrApiUrl = `${radarrBaseUrl}/movie?tmdbId=${tmdbId}`;
    console.log('Calling Radarr API to look up the movie...');

    const radarrApiResponse = await fetch(radarrApiUrl, {
      method: 'GET',
      headers: {
        'X-Api-Key': radarrApiKey,
      },
    });

    if (radarrApiResponse.ok) {
      console.log('Radarr API lookup successful.');
      const radarrApiResponseData = await radarrApiResponse.json();

      if (radarrApiResponseData.length > 0) {
        const movieId = radarrApiResponseData[0].id; // Get the Radarr ID of the first movie
        console.log('Radarr Movie ID:', movieId);

        // Trigger Radarr to search for the movie and download
        const searchUrl = `${radarrBaseUrl}/command`;
        console.log(`Calling Radarr API to search for the movie: ${searchUrl}`);

        const searchRequestBody = {
          name: 'MoviesSearch',
          movieIds: [movieId],
        };

        const searchResponse = await fetch(searchUrl, {
          method: 'POST',
          headers: {
            'X-Api-Key': radarrApiKey,
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(searchRequestBody),
        });

        if (searchResponse.ok) {
          console.log('Radarr movie search initiated.');
        } else {
          console.error(`Failed to initiate movie search: ${searchResponse.statusText}`);
        }
      } else {
        console.error('No movies found for TMDB ID:', tmdbId);
      }
    } else {
      console.error('Radarr API lookup failed:', radarrApiResponse.statusText);
    }
  }
};
```
## #11 Delay 4 minutes

This gives your download client time to download and transfer file to mapped directory. I have Gig+ internet and 99% of the time everything is done in 4 minutes.
## #12 Scan JF Libraries
Using core HTTP feature, send a http POST request to https://jellyfin.domain.com/Library/Refresh with Headers
X-MediaBrowser-Token and value is your Jellyfin API Key

I only do this as Jellyfin doesn't scan my NAS whenever I add a new file.
## #13 Add Comment/Resolve Issue
This just automatically resolves the issue in Jellyseerr and adds a comment letting the user know action was taken.

All I do here is the Code function with 1 input which is the whole body message of the request, this is assigned to inputs.issue in the code

```typescript
export const code = async (inputs) => {
  const issueId = inputs.issue.issue_id;
  const apiKey = 'your-api-key'; // Replace with your actual API key
  const baseURL = 'https://jellyseerr.example.com/api/v1'

  const commentApiUrl = `${baseURL}/issue/${issueId}/comment`;
  const statusApiUrl = `${baseURL}/issue/${issueId}/resolved`;

  const headers = {
    'Content-Type': 'application/json',
    'X-Api-Key': apiKey,
  };

  const commentData = {
    message: 'Your issue has been approved and a new version of the content has been automatically downloaded and updated in Jellyfin. Your issue has been set to Resolved. If you are still experiencing problems, re-open your issue.',
  };

  const commentRequestOptions = {
    method: 'POST',
    headers: headers,
    body: JSON.stringify(commentData),
  };

  try {
    // Post comment
    const commentResponse = await fetch(commentApiUrl, commentRequestOptions);
    const commentData = await commentResponse.json();
    console.log(commentData);

    // Update status
    const statusRequestOptions = {
      method: 'POST', // or PUT depending on your API
      headers: headers,
      // Add any additional data required to update the status
    };

    const statusResponse = await fetch(statusApiUrl, statusRequestOptions);
    const statusData = await statusResponse.json();
    console.log(statusData);

    return true;
  } catch (error) {
    console.error(error);
    return false;
  }
};
```
We are now done with the Radarr flow. Moving onto Sonarr.

## #14 Branch Episodes and Seasons
With the issue data, we also get an "extra" field which is where the requests Affected Episode Number and Affected Season Number are. What this branch does is see if there is an affected Episode Number by seeing if that field in the data exists. You will have to create an issue for a TV show and say an entire season is affected. Then use that sample data, go back to this branch and add the value
1. Jellyseerr Issue Reported body extra 1  as pictured
![Episode Branch Images](/assets/img/branch-episodes.png)

## #15 Add Comment/Resolve Issue

This path meant the user reported an issue on an entire season and basically sends a response to them telling them to do it individually. I probably could have gotten a script working for this but I spent a few hours on it and frustratingly gave up. Maybe I will update this in the future but for now idrc.

Again, all I do here is the code function with 1 input which is the whole body message of the request, this is assigned to inputs.issue in the code

```typescript
export const code = async (inputs) => {
  const issueId = inputs.issue.issue_id;
  const apiKey = 'your-api-key'; // Replace with your actual API key
  const baseURL = 'https://jellyseerr.example.com/api/v1'

  const commentApiUrl = `${baseURL}/issue/${issueId}/comment`;
  const statusApiUrl = `${baseURL}/issue/${issueId}/resolved`;

  const headers = {
    'Content-Type': 'application/json',
    'X-Api-Key': apiKey,
  };

  const commentData = {
    message: 'Please do not report an entire season as the issue. Specify each Episode number. Please delete this issue and resubmit. Your issue has been automatically marked as Resolved.',
  };

  const commentRequestOptions = {
    method: 'POST',
    headers: headers,
    body: JSON.stringify(commentData),
  };

  try {
    // Post comment
    const commentResponse = await fetch(commentApiUrl, commentRequestOptions);
    const commentData = await commentResponse.json();
    console.log(commentData);

    // Update status
    const statusRequestOptions = {
      method: 'POST', 
      headers: headers,
    };

    const statusResponse = await fetch(statusApiUrl, statusRequestOptions);
    const statusData = await statusResponse.json();
    console.log(statusData);

    return true;
  } catch (error) {
    console.error(error);
    return false;
  }
};
```
## #16 Mark as Failed Sonarr

Again, all I do here is the code function with 1 input which is the whole body message of the request, this is assigned to inputs.issue in the code
```typescript
export const code = async (inputs) => {
  const issueSubject = inputs.issue.subject;
  const tvShowNameRegex = /(.*)\s\((\d{4})\)/;
  const match = tvShowNameRegex.exec(issueSubject);

  if (match) {
    const tvShowName = match[1];
    const year = match[2];
    const tvdbId = inputs.issue.media.tvdbId; // Using TVDB ID for TV shows

    console.log(`TV Show name: ${tvShowName}`);
    console.log(`Year: ${year}`);
    console.log(`TVDB ID: ${tvdbId}`);

    // Define your Sonarr API key and base URL
    const sonarrApiKey = 'your-api-key'; // Replace with your Sonarr API key
    const sonarrBaseUrl = 'https://sonarr.example.com/api/v3';

    // Use Sonarr's API to look up the series by TVDB ID and get the Sonarr ID
    const seriesResponse = await fetch(`${sonarrBaseUrl}/series/lookup?term=tvdb:${tvdbId}`, {
      method: 'GET',
      headers: {
        'X-Api-Key': sonarrApiKey,
      },
    });

    if (seriesResponse.ok) {
      const seriesData = await seriesResponse.json();

      if (seriesData.length > 0) {
        const seriesId = seriesData[0].id;

        // Find the affected season and episode numbers
        const affectedSeason = parseInt(inputs.issue.extra.find(item => item.name === 'Affected Season')?.value);
        const affectedEpisode = parseInt(inputs.issue.extra.find(item => item.name === 'Affected Episode')?.value);
        console.log("Season ID = " + affectedSeason);
        console.log("Episode ID = " + affectedEpisode);

        // Get the history of the series
        const historyResponse = await fetch(`${sonarrBaseUrl}/history/series?seriesId=${seriesId}`, {
          method: 'GET',
          headers: {
            'X-Api-Key': sonarrApiKey,
          },
        });

        if (historyResponse.ok) {
          const historyData = await historyResponse.json();

          // Find the most recent entry that matches the affected season and episode
          const recentEntry = historyData.find(entry => {
            const sourceTitleMatch = /S(\d+)E(\d+)/.exec(entry.sourceTitle);
            if (sourceTitleMatch) {
              const sourceSeason = parseInt(sourceTitleMatch[1]);
              const sourceEpisode = parseInt(sourceTitleMatch[2]);
              return sourceSeason === affectedSeason && sourceEpisode === affectedEpisode;
            }
            return false;
          });

          if (recentEntry) {
            const episodeId = recentEntry.episodeId;
            const id = recentEntry.id; // This is the ID you need for marking as failed
            console.log("Found Episode ID = " + episodeId);
            console.log("Found Most Recent Download ID = " + id);

            // Use the episode ID to mark the episode as failed
            const markFailedUrl = `${sonarrBaseUrl}/history/failed/${id}`;
            console.log(`Calling Sonarr API to mark episode as failed: ${markFailedUrl}`);

            const markFailedResponse = await fetch(markFailedUrl, {
              method: 'POST', 
              headers: {
                'X-Api-Key': sonarrApiKey,
              },
              body: JSON.stringify({ status: 'failed' }), 
            });

            if (markFailedResponse.ok) {
              console.log('Episode successfully marked as failed in Sonarr.');
            } else {
              console.error(`Failed to mark episode as failed in Sonarr: ${markFailedResponse.statusText}`);
            }
          } else {
            console.error('No matching entry found in the series history for the affected episode.');
          }
        } else {
          console.error('Failed to fetch series history:', historyResponse.statusText);
        }
      } else {
        console.error('No series found for the provided TVDB ID:', tvdbId);
      }
    } else {
      console.error('Failed to fetch series data:', seriesResponse.statusText);
    }
  }
};
```
You may have to play around a bit and see if when you run this it auto searches for the file. My Sonarr does but my Radarr doesn't, couldnt find any setting. Regardless I include a search command and even if Sonarr searches 2 times it appears 1 will cancel out. This is why no time delay between this code and file deletion.

## #17 Delete File Sonarr
Again, all I do here is the code function with 1 input which is the whole body message of the request, this is assigned to inputs.issue in the code

```typescript
export const code = async (inputs) => {
  const issueSubject = inputs.issue.subject;
  const tvShowNameRegex = /(.*)\s\((\d{4})\)/;
  const match = tvShowNameRegex.exec(issueSubject);

  if (match) {
    const tvShowName = match[1];
    const year = match[2];
    const tvdbId = inputs.issue.media.tvdbId;

    console.log(`TV Show name: ${tvShowName}`);
    console.log(`Year: ${year}`);
    console.log(`TVDB ID: ${tvdbId}`);

    const sonarrApiKey = 'your-api-key';
    const sonarrBaseUrl = 'https://sonarr.example.com/api/v3';

    const seriesResponse = await fetch(`${sonarrBaseUrl}/series/lookup?term=tvdb:${tvdbId}`, {
      method: 'GET',
      headers: {
        'X-Api-Key': sonarrApiKey,
      },
    });

    if (seriesResponse.ok) {
      const seriesData = await seriesResponse.json();

      if (seriesData.length > 0) {
        const seriesId = seriesData[0].id;

        const affectedSeason = parseInt(inputs.issue.extra.find(item => item.name === 'Affected Season')?.value);
        const affectedEpisode = parseInt(inputs.issue.extra.find(item => item.name === 'Affected Episode')?.value);

        const episodeFilesResponse = await fetch(`${sonarrBaseUrl}/episodefile?seriesId=${seriesId}`, {
          method: 'GET',
          headers: {
            'X-Api-Key': sonarrApiKey,
          },
        });

        if (episodeFilesResponse.ok) {
          const episodeFilesData = await episodeFilesResponse.json();

          const targetEpisode = episodeFilesData.find(episode => {
            const parsedPath = episode.relativePath.match(/S(\d+)E(\d+)/);
            if (parsedPath) {
              const episodeSeason = parseInt(parsedPath[1]);
              const episodeNumber = parseInt(parsedPath[2]);
              return episodeSeason === affectedSeason && episodeNumber === affectedEpisode;
            }
            return false;
          });

          if (targetEpisode) {
            const targetEpisodeId = targetEpisode.id;
            console.log("Found Episode ID = " + targetEpisodeId);

            // Delete the target episode file
            const deleteEpisodeUrl = `${sonarrBaseUrl}/episodefile/${targetEpisodeId}`;
            const deleteEpisodeResponse = await fetch(deleteEpisodeUrl, {
              method: 'DELETE',
              headers: {
                'X-Api-Key': sonarrApiKey,
              },
            });

            if (deleteEpisodeResponse.ok) {
              console.log('Episode file successfully deleted in Sonarr.');
            } else {
              console.error(`Failed to delete episode file in Sonarr: ${deleteEpisodeResponse.statusText}`);
            }
          } else {
            console.error('No matching episode found in the episode files for the affected season and episode.');
          }
        } else {
          console.error('Failed to fetch episode files:', episodeFilesResponse.statusText);
        }
      } else {
        console.error('No series found for the provided TVDB ID:', tvdbId);
      }
    } else {
      console.error('Failed to fetch series data:', seriesResponse.statusText);
    }
  }
};
```
## #18 Re-search in Sonarr
Again, all I do here is the code function with 1 input which is the whole body message of the request, this is assigned to inputs.issue in the code

```typescript
export const code = async (inputs) => {
  const issueSubject = inputs.issue.subject;
  const tvShowNameRegex = /(.*)\s\((\d{4})\)/;
  const match = tvShowNameRegex.exec(issueSubject);

  if (match) {
    const tvShowName = match[1];
    const year = match[2];
    const tvdbId = inputs.issue.media.tvdbId;

    console.log(`TV Show name: ${tvShowName}`);
    console.log(`Year: ${year}`);
    console.log(`TVDB ID: ${tvdbId}`);

    const sonarrApiKey = 'your-api-key';
    const sonarrBaseUrl = 'https://sonarr.example.com/api/v3';

    const seriesResponse = await fetch(`${sonarrBaseUrl}/series/lookup?term=tvdb:${tvdbId}`, {
      method: 'GET',
      headers: {
        'X-Api-Key': sonarrApiKey,
      },
    });

    if (seriesResponse.ok) {
      const seriesData = await seriesResponse.json();

      if (seriesData.length > 0) {
        const seriesId = seriesData[0].id;

        const affectedSeason = parseInt(inputs.issue.extra.find(item => item.name === 'Affected Season')?.value);
        const affectedEpisode = parseInt(inputs.issue.extra.find(item => item.name === 'Affected Episode')?.value);

        const historyResponse = await fetch(`${sonarrBaseUrl}/history/series?seriesId=${seriesId}`, {
          method: 'GET',
          headers: {
            'X-Api-Key': sonarrApiKey,
          },
        });

        if (historyResponse.ok) {
          const historyData = await historyResponse.json();

          const recentEntry = historyData.find(entry => {
            const sourceTitleMatch = /S(\d+)E(\d+)/.exec(entry.sourceTitle);
            if (sourceTitleMatch) {
              const sourceSeason = parseInt(sourceTitleMatch[1]);
              const sourceEpisode = parseInt(sourceTitleMatch[2]);
              return sourceSeason === affectedSeason && sourceEpisode === affectedEpisode;
            }
            return false;
          });

          if (recentEntry) {
            const episodeId = recentEntry.episodeId;
            console.log("Found Episode ID = " + episodeId);

            // Perform the episode search
            const searchPayload = {
              name: 'EpisodeSearch',
              episodeIds: [episodeId],
            };

            const searchResponse = await fetch(`${sonarrBaseUrl}/command`, {
              method: 'POST',
              headers: {
                'X-Api-Key': sonarrApiKey,
                'Content-Type': 'application/json',
              },
              body: JSON.stringify(searchPayload),
            });

            if (searchResponse.ok) {
              console.log('Episode search command successfully sent to Sonarr.');
            } else {
              console.error(`Failed to send episode search command to Sonarr: ${searchResponse.statusText}`);
            }
          } else {
            console.error('No matching entry found in the series history for the affected episode.');
          }
        } else {
          console.error('Failed to fetch series history:', historyResponse.statusText);
        }
      } else {
        console.error('No series found for the provided TVDB ID:', tvdbId);
      }
    } else {
      console.error('Failed to fetch series data:', seriesResponse.statusText);
    }
  }
};
```
## #19 Delay for 4 Minutes
Waiting for media to download and transfer.
## #20 Add Comment/Resolve Issue
Again, all I do here is the code function with 1 input which is the whole body message of the request, this is assigned to inputs.issue in the code

```typescript
export const code = async (inputs) => {
  const issueId = inputs.issue.issue_id;
  const apiKey = 'your-api-key'; // Replace with your actual API key
  const baseURL = 'https://jellyseerr.example.com/api/v1'

  const commentApiUrl = `${baseURL}/issue/${issueId}/comment`;
  const statusApiUrl = `${baseURL}/issue/${issueId}/resolved`;

  const headers = {
    'Content-Type': 'application/json',
    'X-Api-Key': apiKey,
  };

  const commentData = {
    message: 'Your issue has been approved and a new version of the content has been automatically downloaded and updated in Jellyfin. Your issue has been set to Resolved. If you are still experiencing problems, re-open your issue.',
  };

  const commentRequestOptions = {
    method: 'POST',
    headers: headers,
    body: JSON.stringify(commentData),
  };

  try {
    // Post comment
    const commentResponse = await fetch(commentApiUrl, commentRequestOptions);
    const commentData = await commentResponse.json();
    console.log(commentData);

    // Update status
    const statusRequestOptions = {
      method: 'POST',
      headers: headers,
    };

    const statusResponse = await fetch(statusApiUrl, statusRequestOptions);
    const statusData = await statusResponse.json();
    console.log(statusData);

    return true;
  } catch (error) {
    console.error(error);
    return false;
  }
};
```
## #21 Same as #12
## Conclusion
Once all this is done you can publish the flow and try it out!

If you have any feedback you can DM on Reddit. I'd love to see how you have edited this automation to your exact needs.

Now the hard part, getting your users to actually report the issues in Jellyseerr and not reach out to you!