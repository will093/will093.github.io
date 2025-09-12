---
layout: post
title: "Web performance: key takeaways from 'High Performance Browser Networking'"
image: "/images/high-performance-browser-networking/earth.jpg"
published: true
---

In software engineering there are a lot of performance optimisations that are commonly used and discussed, but often without a deep understanding of why such optimisations are helpful. I like to not just do something 'because', but to understand the underlying ideas, so I took a recommendation to read [High Performance Browser Networking](https://hpbn.co/) by Ilya Grigorik. 


After reading the book I have a clearer understanding of how the implementation details of a network can affect the performance of applications, as well as how we can get better performance when transferring data over a network. While the book was last updated in 2015 and there are probably some details missing - eg. modern mobile networks and 5G networks are not described in the book - most of the information seems up to date and relevant. 

These are my main takeaways from the book.

## 1) Latency will always be a problem

Bandwidth and latency are the two limiting factors of networks that we often think about. While bandwidth can be a problem, as networking technology improves this is less and less the case. 

However, latency is an intrinsic limitation in networks spread out over a distance, as data can travel no faster than the speed of light between two points. For example light in a vacuum will always take 19ms to travel between New York and London. So we have not only a number of factors which create latency in our networks today, but also a significant theoretical limit to the minimum amount of latency possible in an optimal network.
 
Because of this, it is important to minimise the distance between users and their required resources. In practice this usually involves a well thought out caching strategy and use of CDNs.

## 2) Understanding the TCP protocol is helpful for building performant websites

All HTTP requests from your website are sent over TCP, and some of the implementation details of TCP that affect how your website will perform.

* [TCP handshake](https://hpbn.co/building-blocks-of-tcp/#three-way-handshake) - before you can send HTTP requests to a server, the browser must open a TCP connection. The way that this is implemented takes 1.5 roundtrips, and means that we have 1 roundtrip of latency that we wouldn't have withouth the handshake.
* [TCP slow start](https://hpbn.co/building-blocks-of-tcp/#slow-start) - in order to not overwhelm the network and lose data packets, TCP connection will limit the throughput between client and server initially, and then double it for every roundtrip between client and server. Meaning that the time it takes for the TCP connection to reach full speed is proportional to the round trip time and therefore related to, among other things, physical distance between the client and server.
* [Packet loss and head of line blocking](https://hpbn.co/building-blocks-of-tcp/#head-of-line-blocking) - TCP is not optimised for speed but for reliability (avoiding packet loss). If a data packet gets dropped between client and server, the TCP protocol has logic to retransmit the packet. But because TCP packets have an order to them, the receiver must buffer all subsequent packets until the dropped packet has been resent and received.

All of these implementation details result in an amount of latency which is directly proportional to the time it takes for a roundtrip between client and server. As mentioned in the discussion of latency above,  a well thought out caching strategy and use of CDNs can help.


## 3) Browsers do a lot of networking optimisations for you

Browsers cache responses automatically based on the `cache-control` headers, and are the only place where private data can be cached. It is really worth understanding the [ins and outs of cache headers](https://youtu.be/HiBDZgTNpXY), as they have some nuance to them and when used right can massively improve performance. Allowing data to be cached at multiple levels - your browser, proxy, and CDN/reverse proxy.

Whenever you visit a new hostname (eg. example.com), your browser must first do a DNS lookup to find the IP address for that hostname's server. Once this lookup is done the result is cached in the browser (and at multiple other levels such as your OS, your proxy), so that in the near future visits to the websites will use this cached value.

Modern browsers will also try and predict which hostnames you may need to connect to in the near future, and do a DNS lookup pre-emptively. Furthermore, browsers pre-emptively create TCP connections with hosts which you may need to connect. And in supported browsers you can actually give prompts to do specific [pre-resolves](https://caniuse.com/link-rel-dns-prefetch) and [pre-connects](https://caniuse.com/link-rel-preconnect) in your HTML.

## 4) TLS adds performance overhead

Any TCP connection made over https must then perform a TLS handshake before data can be sent over the connection. Without any optimisations, the handshake takes 2 round trips between server and client, although in practice with TLS false start and TLS session resumption, this is reduced to 1 round trip.

As the round trip time is highly correlated with the distance between client and server, this is yet another reason to make sure that your servers are geographically close to your end users.

## 5) Mobile networks rely on radio resource controllers which adds latency

The radio in mobile devices used for connecting to wifi and 4G/5G networks requires a lot of power, usually second only to the power consumption of the screen. This means that it is not feasible for the radio to be active at all times. 

Because of this (and also for network efficiency) mobile networks rely on [radio resources controllers](https://hpbn.co/mobile-networks/#radio-resource-controller-rrc) (RRCs) to control when a connected device's radio should become active in order to send and recieve data. These RRCs operate at the cell tower level to synchronise data transfer for many connected devices. This means that when your device wants to send a request to a server, it has to 'wait for its turn', its window in which to send or recieve data. This adds quite a lot of latency (in practice 10-100ms) to data transfer on mobile networks.

Also, when making requests for small amounts of data, the radio may stay active for longer than it takes to transfer the data, which is not power efficient. What this means is that when thinking about power consumption when sending data over mobile networks, few large requests are better than many small ones, and patterns such as frequent polling of a server can drain a device battery very quickly. 

## Summary

These are my main takeaways from High Performance Browser Networking by Ilya Grigorik. The book explains these ideas and much more in far more depth, and I recommend reading it if you want to build performant web applications.