---
layout:     post
title:      Speeding up Page Load with Chunking
date:       2016-02-21 18:29:00
summary:    Some techniques for using HTTP Chunked Transfer-Encoding to speed up page load times
tags:       engineering
image:      cooper-sailor-2.png
published:  true
---

I was doing some investigations around HTTP/2 recently and went off on a slight tangent around optimizing browser page load times. And for maybe the 3rd time in the past few years, I ended up finding __flushing early with chunked encoding__ to be really useful. So I'm writing some of this down here in the hope that someone else finds it useful too.

## Techniques

There's a couple of related techniques that you can use with chunked encoding to help compress the [waterfall](http://www.webperformancetoday.com/2010/07/09/waterfalls-101/) and/or cut down on round trips:

1. Fast flush of a chunk containing just the html head.
2. Multiple flushes of the head + most important chunks of content.
3. Flushing additional json data as a callback after the initial page has been rendered.

1 & 2 are very similar, and you can pick  somewhere partway between the two (like you'll see I've done in my sample).

3 is for more specific use-cases, and I haven't really seen it used very often, but it gave a lot of benefits for me on a big js-driven _SPA_ I worked on a few years ago.


## Fast Flushing

So this can give 2 main improvements:

1. Flushing the head early, allows the browser to parse any links in the head for css/js/fonts and start downloading and parsing them as early as possible, without having to wait for however long it takes for the server to generate the main body of the page.
This can also work really well in combination with [link prefetch/preconnect/dns-prefetch](https://css-tricks.com/prefetching-preloading-prebrowsing/)
2. Get the most important content rendered earlier, rather than waiting for all data needed for the page to be collected, before responding.

### Example Site

I setup a little sample page to demonstrate this. It is just a basic html page served through some custom netty code I have running on an EC2 instance which I use to insert a half second delay between flushing the first and second chunks.

The purpose is to simulate a page that is slow to generate, and show you the difference in time to the page being _'ready'_, with flushing the head early vs not.

Here's the [site with fast flushing of the head](http://ec2-54-213-91-136.us-west-2.compute.amazonaws.com/clean/).

And here it is [without flushing](http://ec2-54-213-91-136.us-west-2.compute.amazonaws.com/clean/?chunked=false).

Now taking a look at the webpagetest runs for each, you can see the impact. Note that I ran these using webpagetest's simulated "Mobile 3G" connection on Chrome, running from the US east coast to AWS on the west coast to represent a reasonably bad connection.


#### Results without Flushing

You can see that all the page assets don't start to get downloaded until __after the whole page has been generated__ by server and received by the browser.
The _Document Complete_ time was __1.943 secs__:
![Waterfall for no flushing](/assets/waterfall-no-flushing.png)
<http://www.webpagetest.org/result/160229_WY_AGR/1/details/>


#### Results with Fast Flushing

And now with the early flushing, we can see that most of the assets have finished downloading in the gap between the first head chunk and the rest of the body.
The _Document Complete_ time was __1.165 secs__:
![Waterfall for fast flushed](/assets/waterfall-fast-flushed.png)
<http://www.webpagetest.org/result/160229_99_AGQ/1/details/>

So for this example - although admittedly tailored to demonstrate this - thats a decent __60% improvement__.

#### References

Here's the source, although there's nothing much special about the code: <https://github.com/kerumai/chunking>

And here are some other people's writings about this technique:

- <http://www.ebaytechblog.com/2014/12/08/async-fragments-rediscovering-progressive-html-rendering-with-marko/>
- <http://www.phpied.com/progressive-rendering-via-multiple-flushes/>
- <http://www.websiteoptimization.com/speed/tweak/flush/>


## Late Flushing JSON Data

A scenario where this is useful is when you have a _Single Page Application_ where the following sequential round trips have to occur before the page/app is ready:

1. Browser navigates to the page, downloads and parses it.
2. Browser downloads JS assets and processes them.
3. JS makes XHR to get data to be rendered using JS in the page.

The purpose of this technique is to remove the additional round trip of the XHR in (3), while parallelising the browser downloading assets at the same time as the server collecting the data for the page. 
What you do is:

1. Browser navigates to the page.
  1. Server flushes all of page in response except for the closing body + html tags.
2. Browser downloads JS assets and processes them.
3. Once server has collected more data needed for rendering in JS in the browser, it flushes it as a `<script>` tag which invokes the rendering function with embedded json data.
  1. This can be repeated multiple times if different datasets are going to be available at different times.
  2. And finally the server needs to flush out the closing body + html tags.

So again like for technique (1), this has the advantage of allowing the browser to start downloading assets as early as possible, and rendering the basic layout of the page, while the server is still doing the slower work of collecting all data needed. And then it also avoids an addtional round trip of having to wait for the JS to be downloaded and processed before an XHR can be made to get this from the server.

The only slightly complicated part is that you need to code to handle the race condition around whether the JS is downloaded and processed before or after the JSON data is received.

An extension of this technique was used in Facebook's [BigPipe](https://www.facebook.com/notes/facebook-engineering/bigpipe-pipelining-web-pages-for-high-performance/389414033919/) framework.

The more widely used alternative to this technique is to render the js templates into html on the serverside, and then only hook up the JS event handlers on page load. And if you can do this, then you're probably better off. But for situations where thats not an option, then this can be a handy tool to have available.


## Conclusion

When you're trying to eke out improvements to page load time, especially on high latency networks, then keep flushing, chunked encoding, and link prefetch in mind.

Maybe I'll get round to writing about other improvements that you can make with HTTP/2 and SSE some time soon! Or maybe Daredevil Season 2 will be released before then and I'll go dark for another week.
