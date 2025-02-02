+++
title = "Use Accept header for response type negotiation with Payment Request URLs"
date = 2013-08-27
weight = 73
in_search_index = true

[taxonomies]
authors = ["Stephen Pair"]
status = ["Final"]

[extra]
bip = 73
status = ["Final"]
github = "https://github.com/bitcoin/bips/blob/master/bip-0073.mediawiki"
+++

``` 
  BIP: 73
  Layer: Applications
  Title: Use "Accept" header for response type negotiation with Payment Request URLs
  Author: Stephen Pair <stephen@bitpay.com>
  Comments-Summary: No comments yet.
  Comments-URI: https://github.com/bitcoin/bips/wiki/Comments:BIP-0073
  Status: Final
  Type: Standards Track
  Created: 2013-08-27
```

## Abstract

This BIP describes an enhancement to the payment protocol ([BIP
70](bip-0070.mediawiki "wikilink")) that addresses the need for short
URLs when scanning from QR codes. It generalizes the specification for
the behavior of a payment request URL in a way that allows the client
and server to negotiate the content of the response using the HTTP
Accept: header field. Specifically, the client can indicate to the
server whether it prefers to receive a bitcoin URI or a payment request.

Implementation of this BIP does not require full payment request ([BIP
70](bip-0070.mediawiki "wikilink")) support.

## Motivation

The payment protocol augments the bitcoin: uri scheme with an additional
"payment" parameter that specifies a URL where a payment request can be
downloaded. This creates long URIs that, when rendered as a QR code,
have a high information density. Dense QR codes can be difficult to scan
resulting in a more frustrating user experience. The goal is to create a
standard that would allow QR scanning wallets to use less dense QR
codes. It also makes general purpose QR code scanners more usable with
bitcoin accepting websites.

## Specification

QR scanning wallets will consider a non bitcoin URI scanned from a QR
code to be an end point where either a bitcoin URI or a payment request
can be obtained.

A wallet client uses the Accept: HTTP header to specify whether it can
accept a payment request, a URI, or both. A media type of text/uri-list
specifies that the client accepts a bitcoin URI. A media type of
application/bitcoin-paymentrequest specifies that the client can process
a payment request. In the absence of an Accept: header, the server is
expected to respond with text/html suitable for rendering in a browser.
An HTML response will ensure that QR codes scanned by non Bitcoin wallet
QR scanners are useful (they could render an HTML page with a payment
link that when clicked would open a wallet on the device).

It is not required that the client and server support the full semantics
of an HTTP Accept header. If application/bitcoin-paymentrequest is
specified in the header, the server should send a payment request
regardless of anything else specified in the Accept header. If
text/uri-list is specified (but not application/bitcoin-paymentrequest),
a valid Bitcoin URI should be returned. If neither is specified, the
server can return an HTML page. When a uri-list is returned only the
first item in the list is used (and expected to be a bitcoin URI), any
additional URIs should be ignored.

## Compatibility

Only QR scanning wallets that implement this BIP will be able to process
QR codes containing payment request URLs. There are two possible
workarounds for QR scanning wallets that do not implement this BIP: 1)
the server gives the user an option to change the QR code to a bitcoin:
URI or 2) the user scans the code with a generic QR code scanner.

In the second scenario, if the server responds with a webpage containing
a link to a bitcoin URI, the user can complete the payment by clicking
that link provided the user has a wallet installed on their device and
it supports bitcoin URIs. If the wallet/device does not have support for
bitcoin URIs, the user can fall back on address copy/paste.

This BIP should be fully compatible with BIP 70 assuming it is required
that wallets implementing BIP 70 make use of the Accept: HTTP header
when retrieving a payment request.

## Examples

The first image below is of a bitcoin URI with an amount and payment
request specified (note, this is a fairly minimal URI as it does not
contain a label and the request URL is of moderate size). The second
image is a QR code with only the payment request url specified.

<img src=bip-0073/a.png></img><img src=bip-0073/b.png></img>
