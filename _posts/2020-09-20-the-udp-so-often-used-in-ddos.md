---
title: Why is the User Datagram Protocol UDP so usually used in DDoS attacks?
author: voxduy
date: 2020-09-20 15:00:00 +0700
categories: [Network, DDoS]
tags: [Network, DDoS, Security]
image:
  path: /posts/2020-09-20-the-udp-so-often-used-in-ddos/cyber-blog-ddos-attacks.png
  width: 800
  height: 500
pin: false
---

## Explanation

In three words: connectionless spoofed amplification.

Amplification is actually a natural consequence of client-server communications: When you request something from a server, the response you get is typically larger than your request itself, often many times the size. If you have a botnet handy, and you use it to send millions of bogus requests to thousands of servers, all of which responses are targeted at a single host, you’ve got yourself a DDoS.

But how do you do that redirection? That’s where source IP spoofing comes in, and where UDP has the “edge” over the more commonly-used TCP.

See, TCP requires you to establish a connection before you can talk to the other party. Clearly, you don’t want to have that connection be to you, so here’s what happens:

1. You send a connection request (SYN) to server with source IP `in.no.cent.vic`.
2. Server sends a request+acknowledgement (SYN+ACK) to `in.no.cent.vic` to confirm the connection.
3. `in.no.cent.vic` goes WTF? and rejects the request.
4. You failed, as you should.

UDP is connectionless, so you just send the service request (“I need all the data for this thing here”) with source IP `in.no.cent.vic` to the server, which promptly obliges by immediately sending, well, all the data to `in.no.cent.vic`. Multiply that by a few million requests over a few thousand servers, and you get your DDoS.

![udp-reflection-attack](/posts/2020-09-20-The-UDP-so-often-used-in-DDoS/udp-reflection-attack.png)
_UDP Reflection Attack_

Of course, far more services use TCP than UDP, but DNS is one UDP-based service that is popularly used for DDoS attacks precisely because there are thousands of DNS servers around the Internet, and DNS response packets can be 50 or so times as large as the corresponding requests.

[CISA - DNS Amplification Attacks](https://www.cisa.gov/uscert/ncas/alerts/TA13-088A)

## Wrapping up

Just by this joke image

![tcp-&-udp-joke](/posts/2020-09-20-The-UDP-so-often-used-in-DDoS/tcp-&-udp-joke.png)
_TCP & UDP - Just one image_

## References

- [UDP reflection attacks](https://docs.aws.amazon.com/whitepapers/latest/aws-best-practices-ddos-resiliency/udp-reflection-attacks.html)
