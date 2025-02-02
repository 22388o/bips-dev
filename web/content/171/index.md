+++
title = "Currency/exchange rate information API"
date = 2017-03-04
weight = 171
in_search_index = true

[taxonomies]
authors = ["Luke Dashjr"]
status = ["Rejected"]

[extra]
bip = 171
status = ["Rejected"]
github = "https://github.com/bitcoin/bips/blob/master/bip-0171.mediawiki"
+++

``` 
  BIP: 171
  Layer: Applications
  Title: Currency/exchange rate information API
  Author: Luke Dashjr <luke+bip@dashjr.org>
  Comments-Summary: No comments yet.
  Comments-URI: https://github.com/bitcoin/bips/wiki/Comments:BIP-0171
  Status: Rejected
  Type: Standards Track
  Created: 2017-03-04
  License: BSD-2-Clause
```

## Abstract

A common interface for requesting currency exchange rate information
from a server.

## Copyright

This BIP is licensed under the BSD 2-clause license.

## Specification

Four requests are defined, which are all made by a GET request to a
common URI with parameters encoded in application/x-www-form-urlencoded
format. All matching parameters may be specified with multiple
comma-separated values, which are to be interpreted as "any of these".
Each result is always in JSON format, with a line-feed (never a
carriage-return) separating multiple results.

Authentication for subscription-based services MAY be supported using
standard HTTP authentication. It is recommended to use TLS (HTTPS)
and/or Linked Data Signatures, so that MITM attackers cannot deceive the
client.

To be BIP 171 compatible, servers MUST support at least one
currency-pair compared to XBT. All inquiries for bitcoin amounts MUST be
specified in XBT, even if the presentation to the end user is in another
unit. (FIXME: or should this be satoshis?)

Currency-pair tokens are arbitrary Strings no longer than 255
characters, which may include any ASCII [RFC 3986 unreserved
characters](https://tools.ietf.org/html/rfc3986#section-2.3) (ie,
alphanumerics and the hyphen, underscore, period, and tilde symbols).

Currency code(s) used herein are defined as such:

  - All ISO 4217 codes are valid currency codes.
  - XBT is defined as 100000000 satoshis (commonly known as 1 BTC).
  - Strings longer than 3 characters may be used for currencies without
    an applicable code. (If a shorter code is desired despite this, it
    may be padded with space(s) to the left until it is 4 characters.
    Software MAY strip these spaces.)

Rate is defined as the amount of quote-currency to be exchanged for one
unit of the base-currency. In other words, `1 baseCurrency = rate
quoteCurrency`.

### Enumerating supported currency-pair tokens

Parameters:

  - *mode* - Always "list" for this request.
  - *quote* - If provided, the server MAY limit the results to only
    currency-pairs describing a currency with the given currency
    code(s).
  - *base* - If provided, the server MAY limit the results to only
    currency-pairs describing currency rates compared to the given
    currency code(s).
  - *locale* - If provided, the server MAY limit the results to only
    currency-pairs supporting the given Unicode CLDR locale(s).

Each currency-pair will receive a separate result, a JSON Object, with
the following information:

  - *cp* - The currency-pair token.
  - *quote* - The currency code for the quote currency.
  - *base* - The currency code for the base currency.
  - *locale* - If provided, a String with the applicable Unicode CLDR
    locale.
  - *desc* - Optional description. For example, it could be "Based on
    Florida BTM prices." or any other short String that provides
    information useful to the user. SHOULD be shorter than 45
    characters.
  - *signature* - Optional. May be used for Linked Data Signatures.

Example:

`   Request: `<http://api.example.tld/?mode=list&quote=USD&base=XBT&locale=en_US,en_GB>  
`   Result:`  
`     {"cp":"XBTUSD-ver4", "quote":"USD", "base": "XBT", "locale": "en_US", "desc": "Smoothed averages"}`  
`     {"cp":"2", "quote":"USD", "base": "XBT", "locale": "en_US", "desc": "Updated per-trade"}`  
`     {"cp":"XBTUSD-european", "quote":"USD", "base": "XBT", "locale": "en_GB"}`

### Currency-pair information

Parameters:

  - *mode* - Always "info" for this request.
  - *cp* - Currency pair(s) for which information is requested.

Each currency-pair will receive a separate result, a JSON Object, with
the following information:

  - *cp* - The currency-pair token.
  - *quote* - The currency code for the quote currency.
  - *base* - The currency code for the base currency.
  - *locale* - If provided, a String with the applicable Unicode CLDR
    locale.
  - *desc* - Optional description. For example, it could be "Based on
    Florida BTM prices." or any other short String that provides
    information useful to the user. SHOULD be shorter than 45
    characters.
  - *longdesc* - Optional description, but may be longer and include
    newlines.
  - *symbol* - An Array of prefix and suffix for the quote currency.
    Each may be either a fixed String, an Array of two Strings (negative
    and positive), or null. Any positive or negative symbols must be
    included in this prefix/suffix; it MUST NOT be implied otherwise.
  - *digits* - The type of digits to use for the quote currency's
    numbers. "arabic" should be used for common 0-9 digits.
  - *grouping* - An Array alternating between Numbers representing a
    series of digits, and Strings used as delimiters. If terminated by a
    zero, the final grouping is to be repeated continually. For example,
    the common US locale thousands grouping would be `[3, ",", 0]`
  - *fraction\_sep* - A String to be placed between whole numbers and a
    fractional amount.
  - *fraction\_digits* - Array of absolute minimum (even for whole
    numbers) number of fractional digits, minimum fractional digits when
    a fraction exists, and maximum number of fractional digits when
    absolute precision is not demanded (below which is to be rounded in
    an implementation-dependent manner).
  - *minpoll* - A Number of seconds indicating a minimum time between
    polls to the server. Clients should be prudent about not polling too
    often, even if this number is low.
  - *longpoll* - If provided and true, indicates longpolling is
    supported by the server.
  - *history* - If provided, indicates the server has historical records
    going back no earlier than the POSIX timestamp provided as a value.
  - *archive* - If provided, indicates the server no longer has current
    rates, and has no historical rates more recent than the POSIX
    timestamp provided as a value.
  - *signature* - Optional. May be used for Linked Data Signatures.

Example:

`   Request: `<http://api.example.tld/?mode=info&cp=XBTUSD-ver4,2>  
`   Result:`  
`     {"cp":"XBTUSD-ver4", "quote":"USD", "base": "XBT", "locale": "en_US", "desc": "Smoothed averages", "longdesc": "USD price quotes as compared to Bitcoin value\n\nRecommended for casual usage", "symbol": [["-$", "$"], null], "digits": "arabic", "grouping": [3, ",", 0], "fraction_sep": ".", "fraction_digits": [0, 2, 2], "minpoll": 300, "longpoll": true, "history": 1457231416}`  
`     {"cp":"2", "quote":"USD", "base": "XBT", "locale": "en_US", "desc": "Updated per-trade", "longdesc": "Maximum precision USD price quotes as compared to Bitcoin value", "symbol": [["-$", "$"], null], "digits": "arabic", "grouping": [3, ",", 0], "fraction_sep": ".", "fraction_digits": [0, 2, 2], "minpoll": 3600, "longpoll": false, "history": 1467458333.1225}`

### Current exchange rate

Parameters:

  - *mode* - Always "rate" for this request.
  - *cp* - Currency pair(s) for which rate is requested.
  - *type* - Type of exchange rate data being requested. May be "high",
    "low", "average", "typical", or any other arbitrary name. If
    omitted, the server may provide any rates it deems appropriate.
  - *minrate*, *maxrate* - If specified, indicates this request is a
    longpoll. The server should not send a response until the rate(s)
    fall below or above (respectively) the provided value.
  - *nonce* - If specified, the server SHOULD return it in each result.

Each currency-pair receives a separate result (a JSON Object) with all
requested rate types:

  - *cp* - The currency-pair token.
  - *time* - The time (as a POSIX timestamp) the rate information is
    applicable to (should be approximately the request time).
  - *rates* - A JSON Object with each rate type provided as a key, and a
    Number as the value specifying the rate.
  - *nonce* - Only if the request specified a nonce, the server SHOULD
    include it here as a JSON String.
  - *signature* - Optional. May be used for Linked Data Signatures.

Example:

`   Request: `<http://api.example.tld/?mode=rate&cp=XBTUSD-ver4,2&type=typical,high>  
`   Result:`  
`     {"cp":"XBTUSD-ver4", "time": 1488767410.5463133, "rates": {"typical": 1349.332215, "high": 1351.2}}`  
`     {"cp":"2", "time": 1488767410, "rates": {"typical": 1350.111332}}`

### Historical exchange rates

Parameters:

  - *mode* - Always "history" for this request.
  - *cp* - Currency pair(s) for which rate is requested.
  - *type* - Type of exchange rate data being requested. May be "high",
    "low", "average", "typical", or any other arbitrary name. If
    omitted, the server may provide any rates it deems appropriate.
  - *from* - POSIX timestamp the results should begin with.
  - *to* - POSIX timestamp the results should end with. If omitted, the
    present time shall be used.
  - *nearest* - If provided and true, indicates that only the nearest
    timestamp to "from" must be returned, and a range is not desired.
    ("to" should be omitted in this case.)
  - *ratedelta*, *timedelta* - If specified, the server may omit data
    where the rate or time has not changed since the last provided rate
    and time. If both are provided, either a significant rate change OR
    time change should trigger a new record in the results.

A result is provided for each currency-pair and timestamp record, in the
same format as the current exchange rate request. Records MUST be
provided in chronological order, but only within the scope of the
applicable currency-pair (ie, it is okay to send the full history for
one currency-pair, and then the full history for the next; or to
intermix them out of any given order).

If there is no exact record for the times specified by "from" and/or
"to", a single record before "from" and/or after "to" should also be
included. This is not necessary when only the nearest record is
requested, or when "to" is omitted (ie, ending at the most recent
record).

Example:

`   Request: `<http://api.example.tld/?mode=history&cp=XBTUSD-ver4,2&type=typical&ratedelta=0.1&timedelta=10&from=1488759998&to=1488760090>  
`   Result:`  
`     {"cp":"XBTUSD-ver4", "time": 1488760000, "rates": {"typical": 1300}}`  
`     {"cp":"XBTUSD-ver4", "time": 1488760010, "rates": {"typical": 1301.1}}`  
`     {"cp":"XBTUSD-ver4", "time": 1488760020, "rates": {"typical": 1320}}`  
`     {"cp":"XBTUSD-ver4", "time": 1488760030, "rates": {"typical": 1305}}`  
`     {"cp":"2", "time": 1488760000.1, "rates": {"typical": 1300}}`  
`     {"cp":"2", "time": 1488760010.2, "rates": {"typical": 1301.1}}`  
`     {"cp":"2", "time": 1488760020.2, "rates": {"typical": 1320.111332}}`  
`     {"cp":"2", "time": 1488760031, "rates": {"typical": 1305.222311}}`  
`     {"cp":"XBTUSD-ver4", "time": 1488760040, "rates": {"typical": 1303.33}}`  
`     {"cp":"2", "time": 1488760042, "rates": {"typical": 1303.33}}`  
`     {"cp":"XBTUSD-ver4", "time": 1488760050, "rates": {"typical": 1305}}`  
`     {"cp":"2", "time": 1488760052, "rates": {"typical": 1307}}`  
`     {"cp":"XBTUSD-ver4", "time": 1488760060, "rates": {"typical": 1309}}`  
`     {"cp":"XBTUSD-ver4", "time": 1488760072, "rates": {"typical": 1308}}`  
`     {"cp":"2", "time": 1488760062, "rates": {"typical": 1309.55555555}}`  
`     {"cp":"2", "time": 1488760072, "rates": {"typical": 1308}}`  
`     {"cp":"XBTUSD-ver4", "time": 1488760082, "rates": {"typical": 1309}}`  
`     {"cp":"2", "time": 1488760082, "rates": {"typical": 1309.1}}`

## Motivation

End users often desire to see fiat currency information in their Bitcoin
wallet software. Due to the nature of Bitcoin, there is inherently no
authoritative source for exchange rates. There are many independent
providers of such information, but they all use different formats for
providing it, so wallet software is currently forced to implement
dedicated code for each provider.

By providing a standard interface for retrieving this information,
wallets (and other software) and service providers can implement it
once, and become immediately interoperable with all other compatible
implementations.

## Rationale

Why are multiple results separated by a line-feed rather than using a
JSON Array?

  - Clients ought to cache historical data, and using a line-feed format
    allows them to simply append to a cache file.
  - Parsing JSON typically requires the entire data parsed together as a
    single memory object. Using simple lines to separate results,
    however, allows parsing a single result at a time.

What if long descriptions require line and paragraph breaks?

  - Clients should word-wrap long lines, and JSON escapes newlines as
    "\\n" which can be used doubly ("\\n\\n") for paragraph breaks.

## Backwards compatibility

While this new standard is adopted, software and providers can continue
to use and provide their current formats until they are no longer
needed.

## Reference implementation

TODO

## See also

  - [Draft W3c Linked Data Signatures
    specification](https://w3c-dvcg.github.io/ld-signatures/)
