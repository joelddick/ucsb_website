# HTTP/2 and QUIC
.fx: title

__CS290B__

Dr. Bryce Boe

November 17, 2015

---

# Today's Agenda

* TODO
* Final Presentation Information
* HTTP Optimization Hacks
* HTTP/2
* QUIC

---

# TODO

* Read Chapter 15 in
[High Performance Browser Networking](http://chimera.labs.oreilly.com/books/1230000000545/ch15.html)
* Skim through the three sample reports I posted to Piazza
* Start working on your report so that your can show results from the report on
  Thursday
* We will conduct the second "mock" peer evaluation on Thursday
  (please attend)
* Reports are due Tuesday December 8 by 12PM (noon)

## For Next Tuesday

* Read
  [CAP 12 years](http://www.realtechsupport.org/UB/NP/Numeracy_CAP%2B12Years_2012.pdf)
  later by Eric Brewer
* Read
  [Eventually Consistent](http://www.scalableinternetservices.com/slides/vogels.pdf)
  by Werner Vogels

---

# Final Presentation Information

The final presentation will be in this room Thursday December 10 between 4 and
7PM.

Each team will have fifteen minutes to present and take questions. Presentation
order will be randomized the day of.

I expect everyone to be there to observe and ask questions during __all__ of
the presentations.

We will conduct the final peer evaluation during a 20 minute break after the
5th presentation.

---


# Connections per Page

![Connections per page](img/connections_per_page.png)

Many requests are required for today's web pages:

* CSS
* JavaScript
* Images

Source: [http://httparchive.org](http://httparchive.org)

---

# TCP Phases

![TCP Phases](img/tcp_congestion.png)

Establishing many TCP connections to serve these resources is very slow.

Source: High Performance Browser Networking

---

# TCP Round Trips

![Number of TCP Segments v. Round Trips](img/tcp_segments_and_round_trips.png)

TCP was designed for long-lived flows. HTTP is short and bursty.

---

# HTTP Keep-alive

.fx: img-left

![Head of Line Blocking](img/tcp_head_of_line_blocking.png)

HTTP Keep-alive was introduced to help reuse TCP connections for HTTP, but there
are issues.

While we can reuse a TCP socket for multiple HTTP requests, one heavyweight
request can starve all others.

This is called __Head-of-line blocking__.

---

# HTTP Request Header

![HTTP Header 1](img/http_header_1.png)

---

# HTTP Request Header Repetition

![HTTP Header 2](img/http_header_2.png)

The highlighted sections show the differences from this request to the previous
request. All other bytes are duplicates.

---

# HTTP Request Header Repetition

![HTTP Header 3](img/http_header_3.png)

In this case the only thing that changed from the previous request were 14 new
bytes out of 683.

> How do we minimize this header overhead with HTTP/1.1?

---

# Asset Concatenation

To minimize header overhead in HTTP/1.1 we concatenate files together (e.g.,
JavaScript, and CSS).

In Rails, that is one of the purposes of the asset pipeline.

This technique can also be applied to images.

---

# Image Spriting

![Image Sprite](img/image_sprite.jpg)

A sprite is an image that is comprised of many smaller images.

CSS is used to extract the individual image based on its position and size in
the sprite.

Spriting is cumbersome to work with, and is a hack used to obtain better
performance.

---

# Concurrent TCP Connections

The browser will utilize up to 6 concurrent TCP connections to a single host in
order to speed up the time to obtain a page's resources.

For better performance domain sharding is utilized with HTTP/1.1.

![Concurrent TCP Connections](img/concurrent_http.png)

---

# Performance Goals

We want fewer TCP connections, but

* we don't want head-of-line blocking
* we don't want to have to concatenate our CSS and JavaScript
* we don't want to have to use image sprites
* we don't want to have to split our resources to trick the browser

> What do we do?

---

# HTTP/2

HTTP/2 addresses all of these issues.

## History

* Google internally announced the SPDY protocol (Nov 2009).
* Google released SPDY in Chrome for (Sept 2010).
* Google deployed SPDY on all their servers (Jan 2011).
* Twitter deployed SPDY on its servers (March 2012).
* Apache support for SPDY (April 2012)
* NGINX support for SPDY (June 2012)
* Firefox support for SPDY (June 2012)
* First draft of HTTP/2.0 based on SPDY (Nov 2012)
* RFC 7540: HTTP/2.0 published (May 2015)

---

# HTTP/2 Features

* Multiplexed TCP connection permits concurrent streams over a single TCP
  connection with the ability to prioritize streams.
* Header compression minimizes request and response overhead.
* Server push provides a way for the server to send responses prior to
  receiving a request for the associated resource.

---

# HTTP/2 Binary Frames

![HTTP/2 Frame](img/http2_frame.png)

Note that the headers and data are in separate frames.

---

# HTTP/2 Binary Framed Connection

![HTTP/2 Connection](img/http2_connection.png)

Note that different streams are interleaved over a single TCP connection.

---

# Header Compression

With binary framing, header compression becomes easy.

Initially implemented using GZIP, but CRIME attack revealed weaknesses.

* If an attacker can inject data, compressed size can leak information.

Now implemented using HPACK.

---

# HPACK

![HPACK](img/hpack.png)

1. HPACK allows the transmitted header fields to be encoded via a static
   Huffman code, which reduces their individual transfer size.
2. HPACK requires that both the client and server maintain and update an
   indexed list of previously seen header fields (i.e. establishes a shared
   compression context), which is then used as a reference to efficiently
   encode previously transmitted values.

Source:
[High Performance Browser Networking](http://chimera.labs.oreilly.com/books/1230000000545/ch12.html#HTTP2_HEADER_COMPRESSION)

---

# Binary Framing Benefits

.fx: img-left

![Binary Framing Flexibility](img/http2_flexibility.png)

With binary framing the ordering of resources is flexible.

* Handling of many small resources is efficient
* Headers are compressed, so they are lightweight
* Head of line blocking no longer exists

---

# HTTP/2 Prioritization and Flow Control

With resource ordering not being an issue, we can implement resource
prioritization.

## Request Priority Example

1. Requests that interact with the DOM
2. CSS
3. JavaScript
4. Images

## Flow Control

A `WINDOW_UPDATE` flag exists to control the number of frames _in flight_.

---

# HTTP/2 Saves the Day

HTTP/2 obviates the HTTP/1.1 hacks used over the last twenty years.

* Spriting and asset compilation are no longer needed since we can effectively
  handle many small resources without worrying about head-of-line blocking.

* Domain sharing is irrelevant since all content is delivered over a single
  connection.

---

# Server Push

![HTTP/2 Server Push](img/http2_server_push.png)

When a resource is requested (e.g., `/`) the server can proactively send
additional resources using `PUSH_PROMISE` (e.g., favicon.ico, style.css).

The client can indicate that it does not want the additional content. This
feature is useful if the client already has a cached copy.

---

# HTTP/2 Results

![HTTP/2 Page Load Time](img/http2_results.png)

---

# HTTP/2 Low Latency Results

![HTTP/2 Low Latency Results](img/http2_low_latency_results.png)

---

# HTTP/2 High Latency Results

![HTTP/2 High Latency Results](img/http2_high_latency_results.png)

---

# QUIC

HTTP/2 improves HTTP while still using TCP.

__QUIC__ is an attempt to sidestep TCP in order to avoid TCP limitations,
primarily the round trip necessary to establish connections.

![QUIC](img/quic.png)

---

# Review: TCP Connection Establishment

![TCP Round Trip](img/tcp_round_trip.png)

TCP connections require a full round trip before any application data can be
sent.

---

# Review: TCP+TLS Establishment

![TLS Round Trip](img/tls_handshake.png)

HTTPS (TCP+TLS) requires three full round trips (two with optimizations) before
any application data can be sent.

---

# HTTP/2 (SPDY) and Dropped Packets

![HTTP/2 with dropped packet](img/http2_dropped_packet.png)

HTTP/2 provides excellent multiplexing support. However, any dropped packet
blocks the entire connection.

TCP cannot differentiate between packets dropped by different steams.

---

# TCP Retransmissions

![TCP Retransmissions](img/tcp_retransmissions.png)

Losing packets results in retransmissions. In high latency networks this
process is painfully slow.

---

# Physically Mobile Users

.fx: img-left

![Mobile Users](img/mobile_users.png)

A TCP connection is defined by its source IP address/port combination.

If a connected user's device moves from their house's wifi to their cellular
network or to another network's wifi, they will obtain a new IP address.

This move requires the user to establish a new connection and once again incur
any start-up costs.

---

# QUIC

> Given the limitations of TCP, can we build a better HTTP/2 on top of UDP
> instead?

QUIC is an attempt to do use UDP for HTTP.

QUIC's primary goal is to reduce latency.

In fact, you are already using QUIC if you use Google Chrome and access Google
services.

Inspect via: chrome://net-internals/

---

# QUIC Initial Connection

* Client sends random 64-bit CID (connection ID)
* Server replies with certificate and cookie
* Client responds with:
    * CID
    * cookie
    * proposed encrypted session key
    * encryption algorithm
* Client can immediately send requests

---

# QUIC Subsequent Connections

.fx: img-left

![QUIC second connection](img/quic_second_connection.png)

* Client sends (assumes server certificate hasn't changed):
    * CID
    * cookie
    * proposed encrypted session key
    * encryption algorithm
* Client can immediately send requests

---

# QUIC and IP Spoofing

TCP's three-way handshake makes it immune to IP spoofing related
attacks. UDP by-default is vulnerable.

QUIC is engineered to avoid such attacks through the use of the CID.

---

# QUIC and Packet Loss

![QUIC Multiplexing](img/quic_multiplexing.png)

Packet loss affects only the stream that lost the resource.

No global head-of-line blocking.

TCP handles packet loss through retransmissions. QUIC can handle packet loss
without retransmission.

> How?

---

# Forward Error Correction

![Forward Error Correction](img/forward_error_correction.png)

With forward error correction we trade bandwidth (extra data) for latency
(retransmissions).

---

# QUIC and Physically Mobile Users

.fx: img-left

![Mobile Users](img/mobile_users.png)

Connections in QUIC are based on the CID rather than the its IP address/PORT
combination.

Thus clients can freely move between networks without having to reestablish
their connections.

---

# QUIC Results

* 75% of requests avoid handshake
* Google search observed a 3% reduction in mean page load time.
* The slowest 1% of users experience a 1 second reduction in page load time.
* YouTube users experience 30% fewer rebuffers.

Overall the most significant results occur under poor network conditions.
