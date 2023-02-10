---
layout: post
title: ""
image: ""
published: false
---

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
  
