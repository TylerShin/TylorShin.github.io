---
layout: post
title: "Send XMLHttpRequest when a browser is closed"
author: "Tylor Shin"
categories: frontend
tags: [FrontEnd, Browser]
image: unloadevent.gif
---

# Send XMLHttpRequest when a browser is closed

When you need to send some meta information when a user tries to get out of your page, you can get this event with `beforeunload` event or `unload` event.

For the clear communication, I need these methods for sending the log data not sent.

From StackOverflow, I've got that both events' callback function can't handle asynchronous functions. So, there was a sollution which do XHR synchronously.

    // JQuery example
    $(window).unload(function () {
       $.ajax({
         type: 'GET',
         url: 'SomeUrl.com?id=123'
         async: false,
       });
    });

    // vanila javascript example
    var request = new XMLHttpRequest();
    request.open('GET', '/bar/foo.txt', false);  // `false` makes the request synchronous
    request.send(null);
    
    if (request.status === 200) {
      console.log(request.responseText);
    }

However, there is two significant changes from there.

1. You can use [`sendBeacon`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon) API.
2. `beforeunload` event can't send request from v30+ Chrome browser. ([https://productforums.google.com/forum/#!msg/chrome/5b3ePr9rMVQ/g0o5wOJTHQoJ](https://productforums.google.com/forum/#!msg/chrome/5b3ePr9rMVQ/g0o5wOJTHQoJ))

Let's dig about above one first.

Above synchronous requests have common flaw that blocks the browser navigations(or closing tab, closing browser) because of synchronous request function.

We can solve this problem with sendBeacon API.

`By using the sendBeacon() method, the data is transmitted asynchronously to the web server when the User Agent has an opportunity to do so, without delaying the unload or affecting the performance of the next navigation. [From MDN]`

Yeah, we can avoid the blocking and possible to send data in a asynchronous request. However, this API doesn't work at IE and Edge and the older browers.[[https://caniuse.com/#search=sendBeacon](https://caniuse.com/#search=sendBeacon)]

Combine both limitation, we can make below functions to cover almost evey scenario.

    let alreadySentData = false;
    
    window.addEventListener("beforeunload", () => {
    	sendTicketsBeforeCloseSession();
    });
    window.addEventListener("unload", () => {
    	sendTicketsBeforeCloseSession();
    });
    
    function sendTicketsBeforeCloseSession() {
    	if (alreadySentData) {
    		return;
    	}
    	if (typeof navigator.sendBeacon !== "undefined") {
    		// sendBeacon return boolean following the result
    		const success = navigator.sendBeacon(DESTINATION_URL, encodedTickets);
    		alreadySentData = success;
    	} else {
    		const xhr = new XMLHttpRequest();
           	        xhr.open("POST", DESTINATION_URL, false);
    	        xhr.setRequestHeader("Content-Type", "text/plain;charset=UTF-8");
    	        xhr.send(encodedTickets);
    	        if (xhr.status === 200) {
    	            this.sentLastTickets = true;
    	        }
    	}
    }

The last one you should care about is the `Content-Type` of the request.

sendBeacon API can handle `ArrayBufferView`, `Blob`, `DOMString`(text/plain), or `FormData`. And XMLHttpRequest can handle almost every request.

And if you use Axios or Fetch for the normal request (not for browser quit or navigate to other page), you should match `Content-Type` both normal request and quiting request. Or, You should set the server can handle all `Content-Type` of request.

That's all!

Thanks for reading and if you know more about this topic, please leave a comment!

## References

[Synchronous and asynchronous requests](https://developer.mozilla.org/ko/docs/XMLHttpRequest/Synchronous_and_Asynchronous_Requests) [MDN Web Docs]

beforeunload is deprecated([https://productforums.google.com/forum/#!msg/chrome/5b3ePr9rMVQ/g0o5wOJTHQoJ](https://productforums.google.com/forum/#!msg/chrome/5b3ePr9rMVQ/g0o5wOJTHQoJ))

- I couldn't find an official doc about deprecated.

[sendBeacon API](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon) [MDN Web Docs]

