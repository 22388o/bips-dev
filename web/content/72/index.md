+++
title = "bitcoin: uri extensions for Payment Protocol"
date = 2013-07-29
weight = 72
in_search_index = true

[taxonomies]
authors = ["Gavin Andresen"]
status = ["Final"]

[extra]
bip = 72
status = ["Final"]
github = "https://github.com/bitcoin/bips/blob/master/bip-0072.mediawiki"
+++

``` 
  BIP: 72
  Layer: Applications
  Title: bitcoin: uri extensions for Payment Protocol
  Author: Gavin Andresen <gavinandresen@gmail.com>
  Comments-Summary: No comments yet.
  Comments-URI: https://github.com/bitcoin/bips/wiki/Comments:BIP-0072
  Status: Final
  Type: Standards Track
  Created: 2013-07-29
```

## Abstract

This BIP describes an extension to the bitcoin: URI scheme (BIP 21) to
support the payment protocol (BIP 70).

## Motivation

Allow users to click on a link in a web page or email to initiate the
payment protocol, while being backwards-compatible with existing bitcoin
wallets.

## Specification

The bitcoin: URI scheme is extended with an additional, optional "r"
parameter, whose value is a URL from which a PaymentRequest message
should be fetched (characters not allowed within the scope of a query
parameter must be percent-encoded as described in RFC 3986 and
bip-0021).

If the "r" parameter is provided and backwards compatibility is not
required, then the bitcoin address portion of the URI may be omitted
(the URI will be of the form: <bitcoin:?r=>... ).

When Bitcoin wallet software that supports this BIP receives a bitcoin:
URI with a request parameter, it should ignore the bitcoin
address/amount/label/message in the URI and instead fetch a
PaymentRequest message and then follow the payment protocol, as
described in BIP 70.

Bitcoin wallets must support fetching PaymentRequests via http and https
protocols; they may support other protocols. Wallets must include an
"Accept" HTTP header in HTTP(s) requests (as defined in RFC 2616):

    Accept: application/bitcoin-paymentrequest

If a PaymentRequest cannot be obtained (perhaps the server is
unavailable), then the customer should be informed that the merchant's
payment processing system is unavailable. In the case of an HTTP
request, status codes which are neither success nor error (such as
redirect) should be handled as outlined in RFC 2616.

## Compatibility

Wallet software that does not support this BIP will simply ignore the r
parameter and will initiate a payment to bitcoin address.

## Examples

A backwards-compatible request:

    bitcoin:mq7se9wy2egettFxPbmn99cK8v5AFq55Lx?amount=0.11&r=https://merchant.com/pay.php?h%3D2a8628fc2fbe

Non-backwards-compatible equivalent:

    bitcoin:?r=https://merchant.com/pay.php?h%3D2a8628fc2fbe

## References

[RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616.html "wikilink")
: Hypertext Transfer Protocol -- HTTP/1.1
