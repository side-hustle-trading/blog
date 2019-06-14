---
layout: post
title: TDAmeritrade Authentication using OAUTH2 in C++14
---

For crunchier algorithmic trading software, C++ offers supercharged numerical
performance that often benefits from inhaling real time stock ticker data from
your broker's API.  Unfortunately, writing networking code in C++ is *beast*
compared to an easier to use languages, such as using Flask or Quart in Python.  We
will see that the nerdy pun relates to `boost::beast` which is the networking
library demonstrated in this post.

Authentication of your trading software with TDAmeritrade or any other broker
is critical for at least two reasons:

* Your software will have the permission to manipulate your trading account,
  including the capability of liquidating all of your money into the market in
a microsecond.  You want to be really sure that only the correct software has
this access, at the right time.  Your software may still be buggy, but at least
it will be *your* software.
* TDAmeritrade does not want malicious software or stolen identities used with
  their service either, and will in fact not even serve data to you without
industrial strength encryption (SSL/TLS) employed, plus magic *tokens* that you
get from using your TDAmeritrade password and the OAUTH2 protocol.

You might be wondering why you need a web *server* at all; surely your trading
application is a TDAmeritrade *client*, right?  Unfortunately, OAUTH2 requires
implementing *both* server and client functions.  These will be described
below.

I have long been terrified of programming with "Transport Layer Security" (TLS)
with it's certificates and other black magic, and for good reason.  It is
really hard to set up an encrypted web server without becoming somewhat of an
expert in TLS.  If your program does not get it perfect, you will not get any
data (or decent error messages!)  These days, most browsers require all sites
to be encrypted (`https`), and TDAmeritrade's API service definitely does as
well.  This blog will chronical my first successful battle with writing an
encrypted web server, used to authenticate against `api.tdameritrade.com` using
OAUTH2, in C++.  Sadly, this may not be the last, as we will initially accept
some technical limitations just to get going.

Here is a preview of the skirmishes:
* TDAmeritrade's own documentation of their [authentication
  process](https://developer.tdameritrade.com/) as downright awful.  It is more
productive to figure out that they are using a standard protocol (OAUTH2) and
read much better documentation elsewhere, such as from [Aaron
Parecki](https://aaronparecki.com/oauth-2-simplified/) or [Mitchell
Anicas](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2).
* TDAmeritrade's server will not talk to your server without "certificates"
  properly signed by an actual authority; easily created "self signed"
certificates will not do.  I do not think one can just download such a thing;
you have to somehow get a real certificate from somewhere.
* Now that you have a signed certificate, your software must be properly
  configured to use it.  Code not perfect?  Your SSL handshake will fail and no
data arrives.
* Sweet, `api.tdameritrade.com` will actually connect to you using TLS!  Now
  you get to battle the OAUTH2 transactions to get a "token".  The TDAmeritrade
developer documentation is missing several details and clarity on some points.
I tore my hair out on these, but then I wrote it down here so you don't have
to.

// Learn the certificates
// openssl s_client -showcerts -connect api.tdameritrade.com:443

// openssl s_server -accept 12345 -cert localhost.pem -key localhost-key.pem



