# Muddling your way in real time

Real time demand is a core part of our internet experience, let alone expectation.

Twitter is probably the crowning application of real time I can think of. Hitting the mass audience and industries across the board.

Today we have real time journalism, data, feedback, communication between our teams, from our code and tests. Heck, we can create a brand new virtual machine in under 60 seconds ready to deploy a new site.

On demand and real time is the world we live in today. And if you can't handle the demand, your visitor will head off elsewhere.

<!--more-->

![Muddling your way in real time](/images/muddling-in-real-time-cover.png)

I think its important to distinguish between what's technically real time and what a user perceives as real time. The later being important and the former being arbitrary.

Some applications have been know to respond *so* quickly that they had to introduce a fake delay to meet their users expectations. Specifically: when the program responded so instantly, the user thought something was wrong. With a small delay and a touch of UI feedback (along the lines of "we've processing your request"), the user *felt* the a application was more responsive. (TODO provide references).

This is my own story of how I discovered the web in real time, what I've done over the years and how I use node.js to simplify what used to be very technical problem.

## In the beginning...

My first experience with a real time web was around 2002. I worked for many years on a finance research web site, and stock prices were an important aspect of data.

If you wanted live prices on your site at the time, there would be expensive licences with the London Stock Exchange and some form of Java Applet on your site. We settled for a recurring job that grabbed a 15 minute delayed price CSV file from Yahoo.

The meant that our prices would be "15 minute delayed" (which was a normal expectation of prices shown on free web sites) but for the subsequent 15 minutes the prices would go stale.

It was one afternoon that one of the data collection team asked me to take a look at one of the finance research sites that they were looking at: Hemscott (I should add the original pages have long since left the web).

The page had a [heatmap](http://en.wikipedia.org/wiki/Heat_map) of the FTSE100 prices. What made this particular page interesting is that the prices were changing in real time, and the red/green/sneutral were also changing, so there was a clear visual feedback system to show me this data was live.

What made this page magical though, was I ran the usual "select text test". i.e. if I can select the text, then it's "of the web". If I can't, it's Flash or Java Applets (and right clicking would discover which). But this *was* web. There was a DOM.

![Hemscott from 2002](/images/hemscott.gif)

I spent quite a lot of time poking around some compressed JavaScript, looking at the DOM updating (this was back in the Firebug days so there was no [break on DOM subtree modification](https://developer.chrome.com/devtools/docs/dom-and-styles#setting-dom-breakpoints)).

Hemscott had been able to do what we could not: real time prices, using web technology. **It was magic.** That's all I could ascertain.

---

In retrospect (over several years) I realised that they were achieving the real time effect using Flash. Specifically the `XMLSocket` to connect to the streaming server and using the "Flash SWF ExternalInterface Bridge" to let JavaScript receive messages from the live stream.

Essentially a very similar technique that's used in today's [WebSocket polyfill](https://github.com/gimite/web-socket-js) (which uses Flash for the filling part).

In the mean time Google released Google Talk which was the big tipping point in the web's history for shifting from a request/response pattern, to a server-push pattern, Ajax and Comet respectively.

## From an annoyance to the real time web

Comet is a term that [Alex Russell](http://infrequently.org) (of Dojo fame, and now simply known as The Oracle at Google/he works on Blink) [defined](http://infrequently.org/2006/03/comet-low-latency-data-for-the-browser/) as a method to push data from the server to the client (the browser).

Comet is not a specific technology, but more of an abstracted process. The implementation varied, and frankly at the time, was better suited to system engineers rather than your cowboy developer...like me.

Comet *could* involve any mix of iframes (of course!), long polling, XHR, long running script tags, and so on. To add to complexity, there were oddly named protocols like the Bayeux protocol and BOSH.

All things that provided barrier to entry, but real time, rightly, was hard.

---

Google launched GTalk in 2005 (as part of Gmail) and at the time Google were employing ex-Microsoft developers to solve a very, *very* specific problem. GTalk used long lived iframes to push the chat events up to the client.

But "long lived" means that they needed to refresh (or specifically: reload) eventually, and that reload in IE would cause an <a href="javascript:window.xpaudio.play()">audible clicking noise</a> (this was actually a feature of XP's audio suite). Imagine for a moment, that clicking, coming from seemingly nowhere, on a regular basis when you're chatting online with your friends. Annoying!

<audio id="xpaudio" controls style="width: 100%;" src="/downloads/clicking.wav"></audio>

The solution is amazing (or certainly to me) and the epitome of the web: a hack upon hack upon hack.

[The solution](http://infrequently.org/2006/02/what-else-is-burried-down-in-the-depths-of-googles-amazing-javascript/) would be to create an ActiveX htmlfile object, drop the document with an iframe inside that and the clicking would be suppressed.

And so a stable server push technology emerged.

---

Real time appeared more and more across the web. There was even a product for the finance industry called LightStreamer (which I was even asked by my managers to reverse engineer...which was fun with packed minified files...).

The real hurdle is that there's *two* parts to real time: client *and* the server.

The usual set up for a server in the mid-2000s was to use a [LAMP stack](http://en.wikipedia.org/wiki/LAMP_(software_bundle)). The A standing for Apache was the main sticking point.

Apache is designed (out of the box) to run and spawn a number of processes to deal with concurrent requests.

So if you have 100 apache processes waiting to deal with web requests, and you have 101 requests, the 101st user will have to wait until there's a free process before apache can respond.

This is usually find when you're deal with a request/response situation, apache is fast for that. But when you're keeping connections open to allow a server to *push* a message to the client, you saturate the available apache processes.

What does this mean in practise? If you have 100 processes and 101 streaming requests, the 101st *will never* receive a response. And to that user, the site is hanging indefinitely.

The solution to the server issue is evented server. If I recall correctly, this would be :Twisted for Python, Jakarta for Java, Juggernaut for Ruby, etc. But they were non-trivial to set up. I'll explain an evented server later.

Come 2009 and Ryan Dahl.

## Node is introduced

![Ryan introduces Node at jsconf](/images/ryan-node.jpg)

At the first jsconf.eu, Ryan Dahl, introduced [node.js: evented IO for V8](http://lanyrd.com/2009/jsconfeu/skpz/).

The talk starts quite technical and detailed, but Ryan started to draw similarities with what he was doing with node.js with the DOM.

Although node.js has nothing to do with the DOM, the way that the event loop works is very similar to the way a browser will work.

### The event loop

This is what an event loop *could* look like:

```lua
function main
  initialize()
  while message != quit
    message := get_next_message()
    process_message(message)
  end while
end function
```

In a browser, the `get_next_message` could be the user moving the mouse, or clicking, or an XHR request completing, or a render, or some JavaScript being run. The point being is that the loop waits for a task, then processes that task.

This is where node.js makes concurrent requests (i.e. holding 100s if not 1000s of open connections to clients) easy.

### Hello world of real time

As Ryan demoed in his talk 5 years ago, the code following is the simple proof that comet servers are incredibly simple with Node. The key with the server side is being able to hang inbound requests *whilst* also getting on with other work, like accepting more inbound requests.

```js
var http = require('http');

var server = http.createServer(function (req, res) {
  res.writeHead(200, { 'content-type': 'text/html' });
  res.write('<script>console.log("this is the start of the stream...")</script>');

  var timer = setInterval(function () {
    // if the connection has closed, and we can't write anymore
    if (!res.connection || !res.connection.writable) {
      // then clear this interval, and *attempt* to end the response
      clearInterval(timer);
      res.end();
    } else {
      // otherwise, keep sending a script with logging
      res.write('<script>console.log("and now more messages...")</script>');
    }
  }, 2000);
});

server.listen(8080);
```

This code is saved to `server.js` and run using `node server.js` and now I can visit `localhost:8080` on my machine, and it should start logging, in 2 second increments "and now more messages..." ([sample code including disconnect handling](https://gist.github.com/remy/50f1758a74242d51a90f)).

A comet server has a bit more to it, but with this simple few lines of code we can create as many persistent connection as we like and our server will continue to accept requests.

During the timeout, the server isn't "sleeping", it's *waiting* for the next event, be it the time it to fire, or for another request to come in.

Equally we can easily give the server *more* things to do with the single event loop. It could be collecting live prices from a server, or making APIs calls, or have its own scheduling task all inside the single program *because* of the way node.js is architectured.

What's particularly elegant about node.js today, is that it's incredibly simple to install, has first class support across all three platforms (windows, linux and mac) and is extremely well documented and supported by the community.

## Codifying into standards

As time passed, using Flash and various hacks to achieve real time eventually landed into the standards, typically under the umbrella term of HTML5.

That's to say, today we have *three* native client side solutions to communicating with the server:

1. Ajax. Well known. Well loved. Well understood. The XHR2 spec takes the API further and gives us much more functionality.
2. WebSockets. Bi-directional, persistent sockets, that can be made across origin.
3. EventSource. Push based server *events*, that automatically reconnect when the connection is dropped.

## So, what's next?

Now we live in a world where both the client side *and* server side has been solved and is simple to work with, what can we actually do?

Here's a few examples of where I've used node.js for real time:

- Live reload remote devices with user generated content (in JS Bin)
- Codecasting - like screencasting, but with HTML, CSS & JavaScript
- Remote console injection - for running a desktop console against any mobile device (like old Android or Windows phone)
- Proxy sensor events - streaming the accelerometer from a mobile device to desktop for testing
- User discovery - for a two player game waiting for each other to join the session (like two users joining a chat room)
- Push notification to browser for both progress events and when a long task has completed

A lot of this is made very easy with existing node modules developed by the node community, and stress tested by everyone else.

## Core npm modules

As I'm sure many of you know, the node module repository is rife with libraries to do just about [everything](https://www.npmjs.org/package/true). The libraries that handle real time communication have been baked, and run through the mill pretty hard and there's lot of good choice nowadays.

What's particularly useful about many of these libraries is that they provide both sides of the infrastructure required to achieve real time, and usually require very little to get started.



* libraries
* npm
* socket.io
* options
* notes about mobile!!!

## Long-latency real time feedback

One aspect that particularly interests me is long running requests. For example, when I created 5minfork (a site that clones a github repo and hosts it for 5 minutes), there's a point when the user requests to clone a git repo and there's a latency period.

This period length is unknown (i.e. we could be cloning a large project which takes time), but we do know that it's not instant. So how do we communicate to the user that work is in progress and *most importantly* tell the user that the work is done and they can proceed?

Easily with node.js.

[example]



