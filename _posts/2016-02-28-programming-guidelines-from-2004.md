---
layout:     post
title:      Programming Guidelines From 2004
date:       2016-02-28 14:30:00
tags: engineering,firsttechjob
image: cooper-santa.jpg
published: true
---

The recent [\#FirstTechJob](https://twitter.com/search?q=%23FirstTechJob) meme on twitter got me thinking about my early career at a couple of small software companies. And that led me to finding this old gem that I wrote for my colleagues back in 2003/4 to seemingly try to impart my superior programming wisdom on them. 

Things it brings to mind for me now are:

- I was as patronising back then as I am now!
- Some of it's a bit dated and/or amateurish, but it still mostly makes sense today!
- I really don't miss developing for Internet Explorer 5.
- I've always loved bullet points :)


---

_Last Modified: 24 August 2004_

## Refactoring

The purpose of refactoring is to improve the maintainability of a program, without making any functional changes to it. So, the program should behave exactly the same before the refactoring as after.

- To improve the maintainability of the code, concentrate on:
  - __Removing duplication.__
  - Improving readability.
  - Increasing consistency (of method/class/variable naming, for example).
- Make one unit-of-change at a time. Test after each one (or a few - not more than 1/2 hour). Then commit to CVS (so you have a useful history in case you need to rollback), and add a useful commit-comment, even if it's just "Refactoring".
- Naming of methods and classes is very important, but at the same time, very easy to change (using the IDE-refactoring-tools). Name a method what you think it does (a verb). If you later change your mind, _don't consider changing it, just change it_. Name classes what they represent (a noun). Don't worry about the names being too long, and don't abbreviate them unless it's really necessary or obvious what the abbreviation means.
- Start by breaking up large methods into smaller ones that _only do one thing_ - then name them that thing.
- Delete anything (variables,classes,methods) that isn't being used. If you ever need it again, it'll be in CVS, so no problem. This stops you from having to waste time updating old code that is not even used. It also makes the program easier to understand.
- When refactoring, don't think/worry about _the Design_. For example; If you want to pull one method out of 2 classes into a new class (that does nothing except this one method), then just do it. This new class will probably flesh out in future refactorings anyhow (or you'll push it back into a more suitable class).


## OO

- In general, the methods of a class should operate upon that class's own members. If you find that you are passing a lot of parameters to the methods of a class, then consider declaring these as member variables, and assigning them in the constructor (or some kind of `initialise()` method).
- If you're often passing around lists or maps of objects, consider writing a simple class to represent that list/map of objects. Create `addObject()` or `putObject()` methods that control the type of objects added to the collection. This gives you some nice type-safety, and also often turns into a good place to put convenience/helper methods that act on the objects in the collection.
See [EncapsulatedCollection](http://martinfowler.com/bliki/EncapsulatedCollection.html) ([Martin Fowler knows what he's talking about!](http://martinfowler.com/)).


## Web/Servlets

### HTTP Caching

- Implement/override the [`getLastModified()`](http://docs.oracle.com/javaee/1.4/api/javax/servlet/http/HttpServlet.html#getLastModified(javax.servlet.http.HttpServletRequest)) method in servlets whenever possible. This is particularly simple for pages whose last-modified date is dependent on just one (or a few) files, because you can just return the date of the file.

  The advantage of doing this is that the browser will use it's cached page if it hasn't been modified since it was cached.

  - [Last-Modified Response Header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.29) - Servlet Container uses getLastModified() to set this
  - [If-Modified-Since Request Header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.25) - Used by the browser

- Set the HTTP [Expires](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.21) or [Cache-Control, max-age header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9). For static files or pages which don't change, for example, css and images.

  This is easy to do on the website. You just have to configure Apache to serve your static files, and then you can set an Expires value for those files. It's generally best not to set it to more than 24 hours.

  By setting the Expires/max-age headers, you are telling the browser that it can cache and re-use the page for that length of time, without making any further requests to the server. This can noticeably improve the speed of page-loading.

  - [Difference between Expires and max-age](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.3)
  - [How Mozilla handles HTTP Caching](http://www.mozilla.org/projects/netlib/http/http-caching-faq.html)

- [This book explains HTTP pretty well](http://www.amazon.co.uk/exec/obidos/ASIN/1565925092/qid=1054746584/sr=2-1/ref=sr_2_3_1/202-7031196-8170264).
- To help understand/find what http-headers are being sent back and forth between browser and server, try installing [this plugin for mozilla](http://livehttpheaders.mozdev.org/). Alternatively, use `curl --include <URL>` or `curl --head <URL>` from linux command-line.


### Statelessness is Your Friend :)

Don't hold state in the Session object. Only use this as a method of holding/caching objects to improve performance. An effective way of approaching this is to write all your view-code with the assumption that the user has requested this page by opening a new browser, and typing in the URL.

An implication of this, is that state (or more likely - pointers/referemces/keys to state) should be embedded in the URL's/links you generate. You can do this using parameters, <http://images.google.com/images?q=pug>, or by building it into the path-hierarchy of the URL, <http://del.icio.us/mjsmithlondon/rest>.

Authentication/Login is a pragmatic exception to this. You shouldn't put passwords or maybe even usernames in the URL for security reasons. The alternative of using HTTP-Authentication works, but the UI that web-browsers use for it is crap.


### HTTP & HTML General

- [W3C - Common HTTP Implementation Problems](http://www.w3.org/TR/chips/)
- [W3C - Architecture of the World Wide Web](http://www.w3.org/TR/webarch/)
- [Jakob Nielsen - The Top Ten New Mistakes of Web Design](http://www.useit.com/alertbox/990530.html)
- [W3C - Quality Tips for Webmasters](http://www.w3.org/QA/Tips/)
- [W3C - HTML Techniques for Content Accessibility](http://www.w3.org/TR/WCAG20-HTML-TECHS/)


### General

- If you model your webapp as a website (ie. a graph of linked documents) rather than a client-server application, the rest of the web infrastructure (browsers, proxies) is more likely to work with you, rather than against you.
- A general rule for when to use GET vs POST - Use POST when the request will modify/add/delete something, otherwise use GET.
- Remember that a HTML FORM can be used for GET as well as POST.
  - When responding to a POST; if the page you are going to return is also accessible via GET, then return a Redirect pointing to the URL of the page. I generally always return a redirect, except for things which have no real value within the system (eg. confirmation messages, errors, etc..)
- Try to use the standard html tags consistently in your webapp. Most users will have used a web browser before when navigating around websites. So try to use recognizable links for navigation (rather than some other tag with javascript attached to the onClick() event).
- See this article for info on [opening new windows using js, while still having a link with the real href](http://evolt.org/Links_and_JavaScript_Living_Together_in_Harmony/).


### Character Encoding

- Probably the safest strategy is to encode everyting as UTF-8.
  - Set the response content-type to UTF-8 every time you send a response, eg. `HttpServletResponse.setContentType("text/html; charset=UTF-8")`
  - If serving static html pages (ie. you can't set the response content-type), then add the equivalent meta tag to the html, eg. `<META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=UTF-8">`

    _NB. This is one of the few times I would recommend using the META HTTP-EQUIV tag._

  - Set the character-encoding of the request to UTF-8. You can only be confident that it is this, because all the pages you sent out were UTF-8, eg. `HttpServletRequest.setCharacterEncoding("UTF-8")`

    This tells the servlet-container (tomcat) to assume that all parameters and headers in the request are encoded as UTF-8. Without telling the container this, it will/should check if any character-encoding was set by the browser (none of the browsers seem to bother) and if not, then assume ISO-8859-1 (Latin1).

    __Beware__: _Tomcat has a bug/feature that setCharacterEncoding() must be called before any client code accesses the request-parameters/headers. (ie once an access has been made to a parameter, tomcat parses all of the parameters using the current character-encoding, and caches them. Therefore, setting the encoding after this will have no effect.). Therefore, recommend setting this at the beginning of your Gateway-type class's `service()` method._

- [FAQ: Setting encoding in web authoring applications](https://www.w3.org/International/questions/qa-setting-encoding-in-applications.en.html)
- [FAQ on Software Internationalization (I18N)](http://www.i18nfaq.com/index.html)


### HTML Character Escaping

- [Useful info on html hyphens and unicode](http://www.alistapart.com/articles/emen/)


### HTML

- Try to use the correct markup (tags) as much as possible [Standards-Compliant XHTML](http://www.sizefactory.com/xhtml/xhtmlbasics.html).


### XHTML 

- Look out for ...
  - Even though it is valid XML to close tags like `<div/>`, it messes up most browsers, so you must put a space before the end of the tag - `<div />`.

    I have encountered problems using even this method sometimes (I think css related).

  - Beware when including styles or scripts in the `<HEAD>` element. Because there could be illegal characters in them (eg. <, >, &). According to the [spec](http://www.w3.org/TR/xhtml1/#h-4.8), you need to wrap them in `CDATA` tags.

    But in reality, you also need to comment out the `CDATA` tags using `//` for styles, and `/* */` for scripts.

    _You can avoid these issues by always having your scripts and styles in external files._


### Browser Bugs to watch out for

- IE6 does not send the value of a `<BUTTON type="submit" value="***"/>` when it submits the form. Instead, it sends the text from withing the BUTTON.
  - This is wrong according to the HTML spec, and also strangely enough, it worked correctly in IE5 !
  - This is mainly a problem when you want to have one form with multiple submit buttons. This bug means that you can't use the same name for each submit button, then decide what action to take server-side based on the value.

  - __Solution:__ Give each submit button a different name, but prefixed by the name you really wanted to use. Then base you server-side action purely on whether a particular button's name is in the parameter list.

  - _NB. IE6 also seems to submit the name/text pairs of all the submit buttons, rather than just the one that was clicked. Therefore recommend not using BUTTON at all. (Note that the main reason for using BUTTON was that you can set the display-text to be different to the value.)_

- Internet Explorer(6) displays nothing (blank white page)
- Can be caused by having a script tag in the head without a closing tag !
- The contents of an iframe are not displayed in IE6
- Try removing any DTD from the containing page.


---

And here it is in it's [original 2004 HTML goodness](/assets/programming-guidelines.html)

