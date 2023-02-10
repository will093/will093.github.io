---
layout: post
title: ""
image: ""
published: false
---

# TCP/IP

* HTTP - runs over TCP
* HTTPS - runs over TLS + TCP

# Perf issues with TCP/HTTP

* TCP - slow start?
* TCP - retransmissions? Packet drops/packet delays


* HTTP request latency - 1st round trip is TCP handshake, 2nd round trip to get any response data, 3rd round trip if response is more than 3 packets (4.5kb).
* HTTP - request more thean 4.5kb then an extra round trip required
