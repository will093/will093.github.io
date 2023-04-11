---
layout: post
title: "Web performance: key takeaways from high performance browser networking"
image: ""
published: false
---

In web development there are a lot of performance optimisations that are commonly used and discussed, but often without a deep understanding of why such optimisations are helpful. I always like to not just do something 'because', but to understand why I do it, so I took a recommendation to read [High Performance Browser Networking](https://hpbn.co/). 


After reading this book I have a clearer understanding of how the way networks are implemented can affect the performance of applications and how we can get the most out of these networks, and work around their limitations. While the book was last updated in 2015 and there are probably some details missing, eg. modern mobile networks and 5G networks are not described in the book, most of the information seems up to date and relevant. These are my main takeaways from the book.

# 1) Latency will always be a problem

Bandwidth and latency are the 2 limiting factors of networks that we think about. While bandwidth can be a problem, as networking technology improves this is less and less the case. 

However, latency is an intrinsic limitation in networks spread out over a distance, as data can travel no faster than the speed of light between 2 points. Because of this, a well thought out caching strategy and CDNs to ensure we minimise the distance between our resources and the end user is very important.

# 2) Understanding the TCP protocol is helpful for building performant websites

All HTTP requests from your website are sent over TCP, and some of the implementation details of TCP that affect how your website will perform.

TODO: TCP handshake diagram

* TCP handshake - before you can send HTTP requests to a server, the browser must open a TCP connection. The [way that this is implemented](https://hpbn.co/building-blocks-of-tcp/#three-way-handshake) takes 1.5 roundtrips, and means that we have 1 roundtrip of latency that we wouldn't have withouth the handshake.
* TCP slow start - in order to not overwhelm the network and lose data packets, TCP connection will limit the throughput between client on server initially, and then double it every roundtrip between client and server. Meaning that the time it takes for the TCP connection to reach full speed is proportional to the physical distance between the client and server.
* Packet loss and head of line blocking - TCP is not optimised for speed but for reliability (avoiding packet loss), if a data packet gets dropped between client and server, the TCP protocol has logic to retransmit the packet. But because TCP packets have an order to them, the receiver must buffer all subsequent packets until the dropped packet has been resent and received.

All of these implementation details result in an amount of latency which is directly proportional to the time it takes for a roundtrip between client and server. A good caching strategy (on the client, or on a server close to the end user) and use of CDN to put our resources physically close to the end user is important.


# 3) Browsers do a lot of networking optimisations for you

Browsers cache responses automatically based on the `cache-control` headers, and are the only place where private data can be cached. It is really worth understanding the [ins and outs of cache headers](https://youtu.be/HiBDZgTNpXY), as they have some nuance to them and when used right can massively improve performance. Allowing data to be cached at multiple levels - your browser, proxy, and CDN/reverse proxy.

Whenever you visit a new hostname (eg. example.com), your browser must first do a DNS lookup to find the IP address for that hostname's server. Once this lookup is done the result is cached in the browser (and at multiple other levels such as your OS, your proxy), so that in the near future visits to the websites will use this cached value.

Modern browsers will also try and predict which hostnames you may need to connect to in the near future, and do a DNS lookup pre-emptively. Furthermore, browsers pre-emptively create TCP connections with hosts which you may need to connect. And in supported browsers you can actually give prompts to do specific [pre-resolves](https://caniuse.com/link-rel-dns-prefetch) and [pre-connects](https://caniuse.com/link-rel-preconnect) in your HTML.

# 4) TLS adds performance overhead

Any TCP connection made over https must then perform a TLS handshake before data can be sent over the connection. Without any optimisations, the handshake takes 2 round trips between server and client, although in practice with TLS false start and TLS session resumption, this is reduced to 1 round trip.

As the round trip time is highly correlated with the distance between client and server, this is yet another reason to make sure that your servers are geographically close to your end users.

# 5) Mobile networks rely on Radio resource controllers which adds latency

The radio in mobile devices used for connecting to wifi and 4G/5G networks requires a lot of power, usually second only to the power consumption of the screen. This means that it is not feasible for the radio to be active at all times. 

Because of this (and also for network efficiency) mobile networks rely on [radio resources controllers](https://hpbn.co/mobile-networks/#radio-resource-controller-rrc) (RRCs) to control when a connected device's radio should become active in order to send and recieve data. These RRCs operate at the cell tower level to synchronise data transfer or many connected devices. This means that when your device wants to send a request to a server, it has to 'wait for its turn', its window in which to send or recieve data. This adds a lot of (TODO: how much?) latency to mobile networks that it is difficult to do anything about.

Also, when making requests for small amounts of data, the radio may stay active for longer than it takes to transfer the data, which is not power efficient. What this means is that when thinking about power consumption when sending data over mobile networks, few large requests are better than many small ones, and patterns such as frequent polling of a server can drain a device battery very quickly. 

# Summary

TODO: latency talk

## Some optimisations are only necessary for HTTP/1.1 and not HTTP/2

TODO: explained both protocols and the differences

# Chapter 1 - latency and bandwidth

* Latency (propagation over long distance) limited by the speed of light
  * New York to London has a round trip time of 56 ms
  * Already travelling at 2/3 of the speed of light so won't get much faster

* Last mile latency (close to the destination) is also a problem

* Bandwidth will improve with technology, latency will not improve much. Hence caching, and making data and services available geographically close to where they are needed is super important.

# Chapter 2 - TCP

* TCP and IP
  * IP is internet protocol, it is a system for routing
  * TCP is transfer control protocol, it is a system for sending data over a network in a non-lossy way. 'reliable protocol'

* TCP has a few notable characteristics
  * Flow control - allows sender and receiver to give feedback to each other about how much data they can receive in their buffer, via a receive window.
  * Slow start - allows the sender and receiver to build up the speed of the data connection gradually, with more dta being sent each round trip. A high round trip time will mean that the first bytes when downloading any resources over TCP wll be slow, this is bad for downloading small files. (TODO: how do browsers get around this?)

* Performance
  * Can't move data faster but can mvoe it closes (eg. caching)
  * Not sending unnecessary data
  * Reuse TCP connections (TODO: how to do this in the browser)

# Chapter 3 - UDP

UDP (User Datagram Protocol) is more a faster way to send data than TCP. It sends data as a number of standalone packets known as datagrams. It is useful for some applications like real time video and audio chats but has some disadvantages too. 'Unreliable protocol'

* Packet loss may occur as there is no acknowledgement of received packets and no packet retransmission.
* Order in which the packets arrive is not guaranteed.

Browsers did not previously allow you to communicate using this protocol, but the the WebRTC API now uses it.

# Chapter 4 - TLS

TLS (transport layer security) is designed for encryption, authentication, and data integrity.

* TLS is the successor to the SSL protocol, but a TLS/SSL certificate refer to the same thing.
* TLS sits between HTTP and TCP in the technology stack, so HTTP can operate unchanged.

## TLS handshake
  
TLS handshake is where the client and server establish the TLS session - agree on TLS version, a cipher, verify certificates.

* TLS handshake used to encrypt data using RSA, a mechanism which relied on using an symetric key, which when transmitting data would be encoded using the senders public key, and decoded using the recievers private key. But... this meant that if anyone recorded the encrypted data transmission, although they could not decode it without the receivers private key, if they later got hold of this key they could then decode all previous recorded data transmissions.

* TLS handshake now uses diffie Helman, a more complex algorithm, which guarantees forward secrecy, ie. if the private key is compromised, data from previous sessions is safe. (TODO: how does diffie helman work)

## TLS session resumption

TLS can be resumed across multiple connections/requests, so that the overhead of the TLS handshake is not required, thus improving performance. 

A regular TLS handshake takes 2 client server round trips, but session resumption takes just 1 round trip.

# TLS Certificate Authorities

There are many Certificate Authorities (CAs), and any browser has a list of trusted CAs, which verify that an SSL certificate of any given website is legitimate and trusted. These CAs are known as 'root CAs', there are also imtermediate CAs which can sign an SSL certficiate, but then the client must check that the intermediate CA is trusted by a root CA. TLS relies on there being trusted CAs to verify certificates.

# Resources

* https://hpbn.co/


# Chapter 5 - wireless networks introdution

Wireless network quality depends on:

* Bandwidth - wider rande of bandwidth on a channel means faster performance 
* SNR (Signal noise ratio) - interference/noise from other devices nearby operating on the same frequency/nearby frequency will affect network quality.
* Amount of transmit power for sender and receiver.

# Chapter 6 - wifi

Operates on 2.4GHz and 5GHz (more recently), which have a number of non-overlapping channels. 

* 2.4GHZ only has 3 non-overlapping channels and so in an area with many 2.4GHz networks, devices may interfere with each other. 
* 5GHz has 24 non overlapping channels

On wifi first hop can be slow at times, meaning latency is often a problem.

# Chapter 7 - mobile networks

3G and 4G are specifications rather than particular technologies.

There are a few popular implementations (networks) of 3G/4G - LTE, HSPA, EV-DO - 3G and 4G evolved over time and 3G sort of blends into 4G, with 3.5G, 3.75G, 3.9G. There are some very fast 3G networks which almost meet the 4G spec and get classed as 4G.

Radio Resource Controller (RRC)

The radio technology in mobiel phones consumes **A LOT** of power and so cannot be on all the time with the limited battery life of devices. So the activity of devices is actually controlled by the network. A device is set to idle when not transmitting/receiving, then active once it is time to transmit/receive data.

Mobile users move around, so network towers must track which one a user is connected to, and handover from one to another.

## Outbound latency

* Control-plane latency - time taken for state transitions controlled by RRC (50-100ms)
* User-plane latency - cost for application packet sent from device to radio tower (5ms)
* Core network latency - cost for sending packet from radio tower to packet gateway (30ms-100ms)
* Internet routing latency - cost for sending the packet to the required address on the public internet (variable, limited by speed of light).

## Inbound latency

Similar to outbound latency, but these occur in reverse.

HetNets - many small cells (microcell, picocell, femtocell) which communicate to the carrier network/towers. This is the architecture of 5G.

# Chapter 8 - Optimizing for mobile networks

On mobile devices, radio uses 2nd most power after the screen.

It takes a fixed power cost to transition the radio to active, so many small requests are more expensive than one large request. Frequent polling is especially bad for power consumption and also comes with a fixed latency cost (makes sense when you consider the underlying architecture).

Keepalives are not required to ensure packets aren't dropped. The carrier network will deliver packets even if the device radio goes inactive.

It can be better for power consumption to prefetch data all at once, download large files all in one go etc... (obviously this could have other detrimental effects in some cases, eg. web browser).

Consider prompting a use to switch to wifi if you can detect the https://blog.logrocket.com/how-to-practically-use-performance-api-to-measure-performance/network.

# Chapter 9 - history of HTTP

* 1991 - HTTP 0.9 - very simple, just a single line request and Ascii string response, containing a HTML file.
* 1991 - 1995 - HTTP 1.0 - request and response headers, different response types, connection is closed after each request.
* HTTP 1.1 
  * Multiple request can be made using a single keepalive TCP connection - client or server must close this connection
  * Many other headers were added which we are familiar with (content, encoding, character set, etc...)
* HTTP 2
  *  has a number of performance benefits over HTTP, will be implemented by reverse proxies and browsers so application servers do not need to change anything for the most part???
  * Server push - allows server to push resources to a client when recieving a request.

# Chapter 10 - primer on web performance

* Chrome devtools Network tab actually shows time spent on SSL and Initial Connection (TCP)
* Activities such as streaming large videos/files are bandwidth limited.
* Loading a webpage is latency limitied (lots of small requests).
* JavaScript actually has a performance API which can be used to measure the speed of various real user activities (fetching assets, page navigation etc...)

## Browser optimisations

We can give hints to the browser about optimisations to be performed using the `link` element.

* dns-prefetch - preresolve the hostname
* subresource - prefetch critical resources which will be used later on the page
* prefetch - prefetch resource for this page or future page navigation
* prerender - prerender the specified page in anticipation of the user loading it

TODO: investigate these more - could be super useful!

# Chapter 11 - HTTP 1.x

Steve Sounder's high performance websites offers 14 rules:

* Reduce DNS lookups
* Fewer HTTP requests
* Use a CDN
* Add an expires header and Etags (or some kind of cacheing strategy using HTTP headers)
* Gzip assets
* Avoid HTTP redirects - TODO: how?

HTTP 1.1 has some limitations. 

#Domain sharding

To reduce congestion, a web browser can only handle a certain number of connections to a domain. So to download many resources, developers sometimes resort to some tricks

* Concatenating files, images sprites etc... to combine files. 
* Domain sharding, which is creating extra domains just to allow more browser connections to access server resources.

Browsers automatically attempt to reuse a TCP connection to a particular domain, using the keepalive header (servers must be configured to support this though).

## HTTP Pipelining

HTTP pipelining eliminates latency by moving the 'First in first out (FIFO)' queue from client to server. The client does not need to wait for a server response in order to send another request on the same connection. Instead the responses get queued.

## Head of line blocking

The responses from the server cannot be interleaved (bytes from different response cannot be mixed). So if the first request in the queue is taking a long time to process but the others further back in the queue are ready to send, then they must still wait for the first request to finish being sent.

HTTP - most browsers and many web servers now support HTTP 2 which doesn't have the head of line blocking issue.
  
# Chapter 12 - HTTP 2

1) HTTP 2 allows for interleaving request and response messages on the same TCP connection, solving the head of line problem.
2) HTTP 2 uses binary framing, essentially encoding all data sent into efficient binary.

This removes the need for workarounds like domain sharding, image sprites etc... we only need 1 HTTP/2 connection per origin.

### Streams

A bidirectional flow of data from server to client, which will carry one or more messages. Each stream has a unique ID and priority information, allowing the server and client to identify a stream from multiple streams on a single TCP connection.

### Messages

A HTTP request or response, consisting of 1 or more frames

### Frame

Part of a HTTP message (eg. headers). Frames are encoded to binary, and frames from multiple streams can be interleaved.
can be interleaved with

## Server push

It is possible for a server to send multiple responses to a single client request, and this is known as server push. Eg. if a client requests a HTML file, the server can also respond with the associated JS, CSS files. Server push is not often used as it complicates caching strategies.

# Chapter 13 - optimising application delivery

This chapter is pretty much a summary of what we hae seen so far

# Chapter 14 - primer on browser networking

The browser manage TCP/UDP sockets under the hood. Browsers all limit the number of TCP sockets/connections to 6 - 1 socket is 1 TCP connection.

# Chapter 15 - XML HTTP Request

CORS must be configured to allow cross orign XHR requests.

Streaming is innefficient and awkward, perhaps less so with streaming API. It appears that fetch + Streams API can do this though

Long polling - send a HTTP get and leave it hanging until the server is ready to send a response (a pattern known as Comet)

# Chapter 16 - Server Sent Events (SSE)

Allows the browser to receive a stream of events that are pushed from a server.

Not clear why you would choose this over web sockets.

# Chapter 17 - Websocket

Bidirectional communication between server and client. Closest API to a raw network socket available in the browser.

Distinct from HTTP, Websocket is a custom protocol - no state management, compression, caching built in.

There are now many libraries (sock.js, Socket.io) which provide websocket implementations and also provide a WebSocket server implementation, with fallbacks for clients who don't support WebSocket. The fallbacks are not always great though as they rely on mechanisms such as polling.

WebSocket API abstracts buffering and reconstruction of messages.

The server/client can check the websocket buffer, implement their own priority system, etc...

Each message sent over websocket is split into 1 more more binary frames.

Websocket messages cannot be multiplexed. And with HTTP 1.1, each WebSocket requires 1 TCP connection.

# Chapter 17 - WebRTC

Web RTC allows for direct communication between clients.

For the initial connection btween clients we still need servers.

1) Signalling server, which allows clients to send each other offers to connect to one another.
2) STUN server, which a client can ping to acquire its own network setup info, eg. IP address, NAT configuration.
3) TURN server, a fallback, in case either client has their network set up in such a way that direct connection to each other is not possible.

For 1 to 1 video/audio calls this setup can be great, but consider the topology of the network when implementing connections between multiple clients. A centralised 'supernode' may be best.