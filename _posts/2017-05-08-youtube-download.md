---
title: Downloading from YouTube
date: 2017-05-08 20:57:00 +0200
categories: [Software]
tags: [JavaScript, Web]
image: youtube-cover.png
img_path: /assets/img/posts/2017/
---

Not interested in my story? Skip to [YouTube video download](http://tools.fabioiotti.com/youtube-download/) instructions.

## My Story

I almost always listen to background music while working, and very often I entrust YouTube to select good tracks for me. The result is usually great, and I ended up compiling a few custom playlists.

Till now I have been living happily with this setup, but starting today I want to **play music even while offline**. So here came the question…

> Can I download video/music clips from YouTube without installing any crapware?

Sure thing, from YouTube page I launched Google Chrome *Developer Tools* by pressing *F12*, selected *Network* tab and filtered by *Media*. Playing a video resulted in the media URL being exposed. This URL could be copied in a new tab to initiate the download.

I wanted to **simplify this process, possibly to a single click**. Here started my quest…

## Investigating YouTube Player

A few years ago (around 2013 probably) I had the idea to make a scraper to **collect all video links from a YouTube page and print the list**.

YouTube kindly exposes a `ytplayer` global variable with almost every interesting information about available streams. It was just a matter of organizing the interesting data and adding an interface.

I was satisfied with my results and published the first implementation which might still be available on [my older website](http://fabiogiopla.altervista.org/guides/youtube-downloader).

Almost every video seemed to work at the time, with just a few exceptions that I assumed to be "protected" somehow…

## The Protection Mechanism

It turned out that some videos actually implemented a **protection mechanism**. The idea is simple: the server gives your browser everything that's necessary to download the streams, but some requests must be signed, and the signature algorithm changes from time to time.

The Flash and JavaScript implementations of the video player contain the signing algorithm, which is downloaded and executed by your browser. Normally, it would just be a matter of extracting the signature algorithm and shipping it with my scraper; but this cannot be done since the algorithm is never the same.

## Bypassing the Protection

So the next step is to write a script to **extract the signature algorithm**, and use that algorithm to sign scraped URLs.

First I look for the YouTube player element (which has ID `movie_player`) and read it's `data-version` attribute, that contains the address of the video player script with latest signature algoritm. I proceed to download it.

According to external sources (*Youtube Grabber*), the structure of the player always stays the same, only the name of the signature function changes. I exploit this by looking for a call to this very function. Should look something like `someclass.set("signature", somefunction(someparameter));`.

Once I have the name of the signing function (in this example `somefunction`) I look for it's declaration. It looks something like `somefunction=function(a){a=a.split("");someotherclass.blablabla(someparam)etc etc;return a.join("")};`.

I also need the source code of `someotherclass`, so I look for it's declaration as well... `someotherclass={stuff:function(param1){etc etc},otherstuff:etc etc};`.

After I got everything I compile the extracted code to an anonymous function and use it to compute the signature.

## Finishing

The hard part is done: I **extract the unsigned URLs** from global `ytplayer` variable and I **sign them with the extracted algorithm**. All I need then is **printing the list of results**.

I decided to distribute the script as a bookmarklet (that is a bookmark which contains JavaScript code), finally achieving my goal of **"one click to rule them all" without installing anything**.

You can find sources and script on the [YouTube Video Downloader bookmarklet](http://tools.fabioiotti.com/youtube-download/) page.

Thanks for reading, if you have any question or suggestion you can leave it in the section below. Hope this guide will be useful to you.

## Update

On 2017-12-16 I noticed that YouTube changed something and this procedure broke. Since this change, sometimes `args` are not exposed from `ytplayer` anymore.

I had to adapt the procedure and call the `https://www.youtube.com/get_video_info?&video_id=SOME_VIDEO_ID&el=info&ps=default&eurl=&gl=US&hl=en` API instead, which returns the values that used to populate the `args` field. Everything else is the same as before.

## References

> [YouTube Download URL 403 - StackOverflow](http://stackoverflow.com/a/32550092/1135019)
>
> [Getting YouTube stream URL - StackOverflow](https://stackoverflow.com/a/35631022/1135019)
>
> [YoutubeGrabber - GitHub](https://github.com/lure/YoutubeGrabber/blob/a9264ec3545f6d23…eb209275c2a/src/main/scala/ru/shubert/yt/Decipher.scala#L129)
