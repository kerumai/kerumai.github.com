---
layout:     post
title:      How I Learned to Stop Worrying and Love the Chunks
date:       2016-02-21 18:29:00
summary:    HTTP Chunked encoding is a simple and under-used tool.
categories: engineering
published: true
---

So I was doing some investigations around HTTP/2 recently and went off on a slight tangent around optimizing initial browser page load. And for maybe the 3rd time in the past few years, I ended up finding __chunked encoding__ to be really useful.

There's a couple of related strategies that you can use with chunked encoding to help compress the [waterfall](http://www.webperformancetoday.com/2010/07/09/waterfalls-101/) and/or cut down on round trips:

1. Fast flush of a chunk containing just the html head.
2. Multiple flushes of the head + most important chunks of content.
3. Flushing additional json data as a callback after the initial page has been rendered.

1 & 2 are very similar, and you can pick  somewhere partway between the two (like you'll see I've done in my sample).

3 is for more specific use-cases, and I haven't really seen it used anywhere else, but it gave a lot of benefits for me on a big js-driven _SPA_ I worked on a few years ago.


# Fast Flushing

So this can give 2 main improvements:
1. Flushing the head early, allows the browser to parse any links in the head for css/js/fonts and start downloading and parsing them as early as possible, without having to wait for however long it takes for the server to generate the main body of the page.
This can also work really well in combination with [link prefetch/preconnect/dns-prefetch](https://www.igvita.com/posa/high-performance-networking-in-google-chrome/#prefetching)
2. Get the most important content rendered earlier, rather than waiting for all data needed for the page to be collected, before responding.

## Example

I setup a little sample site to demonstrate this. It's just a modified version of this [clean jekyll template](http://jekyllthemes.org/themes/clean/) served through some custom netty code I have running on an EC2 instance which I use to insert a 1 second delay between flushing the first and second chunks.

The purpose is to simulate a page that takes 1 second to generate, and show you the difference in time to the page being _'ready'_, with flushing the head early vs not. 
Choosing when a page is ready can be very specific to the site in question, but for this I've inserted a [performance timing mark](https://developer.mozilla.org/en-US/docs/Web/API/Performance/mark) - "rum_page_ready" - at the point when the jquery and custom js files have been downloaded and processed, and the DOM is ready. And I also print this out into the page in a wonderful `<marquee>` tag.


Here's the site with fast flushing of the head - <https://sample.kerumai.com/clean/>

And here it is without - <https://sample.kerumai.com/clean/?chunked=false>

Now taking a look at the webpagetest runs for each, you can see the impact. Note that I ran these using webpagetest's simulated "Mobile 3G" connection on Chrome.

##TODO
- screenshots and links for the webpagestest runs. 
- point out the main timings, and describe why the difference.


Here's what some others have to say about this:
- <http://www.ebaytechblog.com/2014/12/08/async-fragments-rediscovering-progressive-html-rendering-with-marko/>
- <http://www.phpied.com/progressive-rendering-via-multiple-flushes/>
- <http://www.websiteoptimization.com/speed/tweak/flush/>


# Late Flushing JSON to Bottom of Page

__TODO__ My hacky setup on mercury where I quickly flushed the initial html, and then flushed the json data in a js callback later once it was available.

