+++
title = "Passphrase-protected private key"
date = 2012-11-20
weight = 38
in_search_index = true

[taxonomies]
authors = ["Mike Caldwell", "Aaron Voisine"]
status = ["Draft (Some confusion applies: The announcements for this never made it to the list, so it hasn't had public discussion)"]

[extra]
bip = 38
status = ["Draft (Some confusion applies: The announcements for this never made it to the list, so it hasn't had public discussion)"]
github = "https://github.com/bitcoin/bips/blob/master/bip-0038.mediawiki"
+++

``` 
  BIP: 38
  Layer: Applications
  Title: Passphrase-protected private key
  Author: Mike Caldwell <mcaldwell@swipeclock.com>
          Aaron Voisine <voisine@gmail.com>
  Comments-Summary: Unanimously Discourage for implementation
  Comments-URI: https://github.com/bitcoin/bips/wiki/Comments:BIP-0038
  Status: Draft (Some confusion applies: The announcements for this never made it to the list, so it hasn't had public discussion)
  Type: Standards Track
  Created: 2012-11-20
  License: PD
```

## Abstract

A method is proposed for encrypting and encoding a passphrase-protected
Bitcoin private key record in the form of a 58-character
Base58Check-encoded printable string. Encrypted private key records are
intended for use on paper wallets and physical Bitcoins. Each record
string contains all the information needed to reconstitute the private
key except for a passphrase, and the methodology uses salting and
*scrypt* to resist brute-force attacks.

The method provides two encoding methodologies - one permitting any
known private key to be encrypted with any passphrase, and another
permitting a shared private key generation scheme where the party
generating the final key string and its associated Bitcoin address (such
as a physical bitcoin manufacturer) knows only a string derived from the
original passphrase, and where the original passphrase is needed in
order to actually redeem funds sent to the associated Bitcoin address.

A 32-bit hash of the resulting Bitcoin address is encoded in plaintext
within each encrypted key, so it can be correlated to a Bitcoin address
with reasonable probability by someone not knowing the passphrase. The
complete Bitcoin address can be derived through successful decryption of
the key record.

## Motivation

The motivation to make this proposal stems from observations of the way
physical bitcoins and paper wallets are used.

An issuer of physical bitcoins must be trustworthy and trusted. Even if
trustworthy, users are rightful to be skeptical about a third party with
theoretical access to take their funds. A physical bitcoin that cannot
be compromised by its issuer is always more intrinsically valuable than
one that can.

A two-factor physical bitcoin solution is highly useful to individuals
and organizations wishing to securely own bitcoins without any risk of
electronic theft and without the responsibility of climbing the
technological learning curve necessary to produce such an environment
themselves. Two-factor physical bitcoins allow a secure storage solution
to be put in a box and sold on the open market, greatly enlarging the
number of people who are able to securely store bitcoins.

Existing methodologies for creating two-factor physical bitcoins are
limited and cumbersome. At the time of this proposal, a user could
create their own private key, submit the public key to the physical
bitcoin issuer, and then receive a physical bitcoin that must be kept
together with some sort of record of the user-generated private key, and
finally, must be redeemed through a tool. The fact that the physical
bitcoin must be kept together with a user-produced private key negates
much of the benefit of the physical bitcoin - the user may as well just
print and maintain a private key.

A standardized password-protected private key format makes acquiring and
redeeming two-factor physical bitcoins simpler for the user. Instead of
maintaining a private key that cannot be memorized, the user may choose
a passphrase of their choice. The passphrase may be much shorter than
the length of a typical private key, short enough that they could use a
label or engraver to permanently commit their passphrase to their
physical Bitcoin piece once they have received it. By adopting a
standard way to encrypt a private key, we maximize the possibility that
they'll be able to redeem their funds in the venue of their choice,
rather than relying on an executable redemption tool they may not wish
to download.

Password and passphrase-protected private keys enable new practical use
cases for sending bitcoins from person to person. Someone wanting to
send bitcoins through postal mail could send a password-protected paper
wallet and give the recipient the passphrase over the phone or e-mail,
making the transfer safe from interception of either channel. A user of
paper wallets or Bitcoin banknote-style vouchers ("cash") could carry
funded encrypted private keys while leaving a copy at home as an element
of protection against accidental loss or theft. A user of paper wallets
who leaves bitcoins in a bank vault or safety deposit box could keep the
password at home or share it with trusted associates as protection
against someone at the bank gaining access to the paper wallets and
spending from them. The foreseeable and unforeseeable use cases for
password-protected private keys are numerous.

## Copyright

This proposal is hereby placed in the public domain.

## Rationale

  -   
    ***User story:** As a Bitcoin user who uses paper wallets, I would
    like the ability to add encryption, so that my Bitcoin paper storage
    can be two factor: something I have plus something I know.*
    ***User story:** As a Bitcoin user who would like to pay a person or
    a company with a private key, I do not want to worry that any part
    of the communication path may result in the interception of the key
    and theft of my funds. I would prefer to offer an encrypted private
    key, and then follow it up with the password using a different
    communication channel (e.g. a phone call or SMS).*
    ***User story:** (EC-multiplied keys) As a user of physical
    bitcoins, I would like a third party to be able to create
    password-protected Bitcoin private keys for me, without them knowing
    the password, so I can benefit from the physical bitcoin without the
    issuer having access to the private key. I would like to be able to
    choose a password whose minimum length and required format does not
    preclude me from memorizing it or engraving it on my physical
    bitcoin, without exposing me to an undue risk of password cracking
    and/or theft by the manufacturer of the item.*
    '**'User story:** (EC multiplied keys) As a user of paper wallets, I
    would like the ability to generate a large number of Bitcoin
    addresses protected by the same password, while enjoying a high
    degree of security (highly expensive scrypt parameters), but without
    having to incur the scrypt delay for each address I generate.

## Specification

This proposal makes use of the following functions and definitions:

  - **AES256Encrypt, AES256Decrypt**: the simple form of the well-known
    AES block cipher without consideration for initialization vectors or
    block chaining. Each of these functions takes a 256-bit key and 16
    bytes of input, and deterministically yields 16 bytes of output.
  - **SHA256**, a well-known hashing algorithm that takes an arbitrary
    number of bytes as input and deterministically yields a 32-byte
    hash.
  - **scrypt**: A well-known key derivation algorithm. It takes the
    following parameters: (string) password, (string) salt, (int) n,
    (int) r, (int) p, (int) length, and deterministically yields an
    array of bytes whose length is equal to the length parameter.
  - **ECMultiply**: Multiplication of an elliptic curve point by a
    scalar integer with respect to the [secp256k1](secp256k1 "wikilink")
    elliptic curve.
  - **G, N**: Constants defined as part of the
    [secp256k1](secp256k1 "wikilink") elliptic curve. G is an elliptic
    curve point, and N is a large positive integer.
  - **[Base58Check](Base58Check "wikilink")**: a method for encoding
    arrays of bytes using 58 alphanumeric characters commonly used in
    the Bitcoin ecosystem.

### Prefix

It is proposed that the resulting Base58Check-encoded string start with
a '6'. The number '6' is intended to represent, from the perspective of
the user, "a private key that needs something else to be usable" - an
umbrella definition that could be understood in the future to include
keys participating in multisig transactions, and was chosen with
deference to the existing prefix '5' most commonly observed in [Wallet
Import Format](Wallet_Import_Format "wikilink") which denotes an
unencrypted private key.

It is proposed that the second character ought to give a hint as to what
is needed as a second factor, and for an encrypted key requiring a
passphrase, the uppercase letter P is proposed.

To keep the size of the encrypted key down, no initialization vectors
(IVs) are used in the AES encryption. Rather, suitable values for
IV-like use are derived using scrypt from the passphrase and from using
a 32-bit hash of the resulting Bitcoin address as salt.

### Proposed specification

  - Object identifier prefix: 0x0142 (non-EC-multiplied) or 0x0143
    (EC-multiplied). These are constant bytes that appear at the
    beginning of the Base58Check-encoded record, and their presence
    causes the resulting string to have a predictable prefix.
  - How the user sees it: 58 characters always starting with '6P'
      - Visual cues are present in the third character for visually
        identifying the EC-multiply and compress flag.
  - Count of payload bytes (beyond prefix): 37
      - 1 byte (*flagbyte*):
          - the most significant two bits are set as follows to preserve
            the visibility of the compression flag in the prefix, as
            well as to keep the payload within the range of allowable
            values that keep the "6P" prefix intact. For
            non-EC-multiplied keys, the bits are 11. For EC-multiplied
            keys, the bits are 00.
          - the bit with value 0x20 when set indicates the key should be
            converted to a base58check encoded P2PKH bitcoin address
            using the DER compressed public key format. When not set, it
            should be a base58check encoded P2PKH bitcoin address using
            the DER uncompressed public key format.
          - the bits with values 0x10 and 0x08 are reserved for a future
            specification that contemplates using multisig as a way to
            combine the factors such that parties in possession of the
            separate factors can independently sign a proposed
            transaction without requiring that any party possess both
            factors. These bits must be 0 to comply with this version of
            the specification.
          - the bit with value 0x04 indicates whether a lot and sequence
            number are encoded into the first factor, and activates
            special behavior for including them in the decryption
            process. This applies to EC-multiplied keys only. Must be 0
            for non-EC-multiplied keys.
          - remaining bits are reserved for future use and must all be 0
            to comply with this version of the specification.
      - 4 bytes: SHA256(SHA256(expected\_bitcoin\_address))\[0...3\],
        used both for typo checking and as salt
      - 16 bytes: Contents depend on whether EC multiplication is used.
      - 16 bytes: lasthalf: An AES-encrypted key material record
        (contents depend on whether EC multiplication is used)
  - Range in base58check encoding for non-EC-multiplied keys without
    compression (prefix 6PR):
      - Minimum value:
        6PRHv1jg1ytiE4kT2QtrUz8gEjMQghZDWg1FuxjdYDzjUkcJeGdFj9q9Vi
        (based on 01 42 C0 plus thirty-six 00's)
      - Maximum value:
        6PRWdmoT1ZursVcr5NiD14p5bHrKVGPG7yeEoEeRb8FVaqYSHnZTLEbYsU
        (based on 01 42 C0 plus thirty-six FF's)
  - Range in base58check encoding for non-EC-multiplied keys with
    compression (prefix 6PY):
      - Minimum value:
        6PYJxKpVnkXUsnZAfD2B5ZsZafJYNp4ezQQeCjs39494qUUXLnXijLx6LG
        (based on 01 42 E0 plus thirty-six 00's)
      - Maximum value:
        6PYXg5tGnLYdXDRZiAqXbeYxwDoTBNthbi3d61mqBxPpwZQezJTvQHsCnk
        (based on 01 42 E0 plus thirty-six FF's)
  - Range in base58check encoding for EC-multiplied keys without
    compression (prefix 6Pf):
      - Minimum value:
        6PfKzduKZXAFXWMtJ19Vg9cSvbFg4va6U8p2VWzSjtHQCCLk3JSBpUvfpf
        (based on 01 43 00 plus thirty-six 00's)
      - Maximum value:
        6PfYiPy6Z7BQAwEHLxxrCEHrH9kasVQ95ST1NnuEnnYAJHGsgpNPQ9dTHc
        (based on 01 43 00 plus thirty-six FF's)
  - Range in base58check encoding for EC-multiplied keys with
    compression (prefix 6Pn):
      - Minimum value:
        6PnM2wz9LHo2BEAbvoGpGjMLGXCom35XwsDQnJ7rLiRjYvCxjpLenmoBsR
        (based on 01 43 20 plus thirty-six 00's)
      - Maximum value:
        6PnZki3vKspApf2zym6Anp2jd5hiZbuaZArPfa2ePcgVf196PLGrQNyVUh
        (based on 01 43 20 plus thirty-six FF's)

#### Encryption when EC multiply flag is not used

Encrypting a private key without the EC multiplication offers the
advantage that any known private key can be encrypted. The party
performing the encryption must know the passphrase.

Encryption steps:

1.  Compute the Bitcoin address (ASCII), and take the first four bytes
    of SHA256(SHA256()) of it. Let's call this "addresshash".
2.  Derive a key from the passphrase using scrypt
      - Parameters: *passphrase* is the passphrase itself encoded in
        UTF-8 and normalized using Unicode Normalization Form C (NFC).
        salt is *addresshash* from the earlier step, n=16384, r=8, p=8,
        length=64 (n, r, p are provisional and subject to consensus)
      - Let's split the resulting 64 bytes in half, and call them
        *derivedhalf1* and *derivedhalf2*.
3.  Do AES256Encrypt(block = bitcoinprivkey\[0...15\] xor
    derivedhalf1\[0...15\], key = derivedhalf2), call the 16-byte result
    *encryptedhalf1*
4.  Do AES256Encrypt(block = bitcoinprivkey\[16...31\] xor
    derivedhalf1\[16...31\], key = derivedhalf2), call the 16-byte
    result *encryptedhalf2*

The encrypted private key is the Base58Check-encoded concatenation of
the following, which totals 39 bytes without Base58 checksum:

  - 0x01 0x42 + *flagbyte* + *salt* + *encryptedhalf1* +
    *encryptedhalf2*

Decryption steps:

1.  Collect encrypted private key and passphrase from user.
2.  Derive *derivedhalf1* and *derivedhalf2* by passing the passphrase
    and *addresshash* into scrypt function.
3.  Decrypt *encryptedhalf1* and *encryptedhalf2* using AES256Decrypt,
    merge them to form the encrypted private key.
4.  Convert that private key into a Bitcoin address, honoring the
    compression preference specified in *flagbyte* of the encrypted key
    record.
5.  Hash the Bitcoin address, and verify that *addresshash* from the
    encrypted private key record matches the hash. If not, report that
    the passphrase entry was incorrect.

#### Encryption when EC multiply mode is used

Encrypting a private key with EC multiplication offers the ability for
someone to generate encrypted keys knowing only an EC point derived from
the original passphrase and some salt generated by the passphrase's
owner, and without knowing the passphrase itself. Only the person who
knows the original passphrase can decrypt the private key. A code known
as an *intermediate code* conveys the information needed to generate
such a key without knowledge of the passphrase.

This methodology does not offer the ability to encrypt a known private
key - this means that the process of creating encrypted keys is also the
process of generating new addresses. On the other hand, this serves a
security benefit for someone possessing an address generated this way:
if the address can be recreated by decrypting its private key with a
passphrase, and it's a strong passphrase one can be certain only he
knows himself, then he can safely conclude that nobody could know the
private key to that address.

The person who knows the passphrase and who is the intended beneficiary
of the private keys is called the *owner*. He will generate one or more
"intermediate codes", which are the first factor of a two-factor
redemption system, and will give them to someone else we'll call
*printer*, who generates a key pair with an intermediate code can know
the address and encrypted private key, but cannot decrypt the private
key without the original passphrase.

An intermediate code should, but is not required to, embed a printable
"lot" and "sequence" number for the benefit of the user. The proposal
forces these lot and sequence numbers to be included in any valid
private keys generated from them. An owner who has requested multiple
private keys to be generated for him will be advised by applications to
ensure that each private key has a unique lot and sequence number
consistent with the intermediate codes he generated. These mainly help
protect *owner* from potential mistakes and/or attacks that could be
made by *printer*.

The "lot" and "sequence" number are combined into a single 32 bit
number. 20 bits are used for the lot number and 12 bits are used for the
sequence number, such that the lot number can be any decimal number
between 0 and 1048575, and the sequence number can be any decimal number
between 0 and 4095. For programs that generate batches of intermediate
codes for an *owner*, it is recommended that lot numbers be chosen at
random within the range 100000-999999 and that sequence numbers are
assigned starting with 1.

Steps performed by *owner* to generate a single intermediate code, if
lot and sequence numbers are being included:

1.  Generate 4 random bytes, call them *ownersalt*.
2.  Encode the lot and sequence numbers as a 4 byte quantity
    (big-endian): lotnumber \* 4096 + sequencenumber. Call these four
    bytes *lotsequence*.
3.  Concatenate *ownersalt* + *lotsequence* and call this
    *ownerentropy*.
4.  Derive a key from the passphrase using scrypt
      - Parameters: *passphrase* is the passphrase itself encoded in
        UTF-8 and normalized using Unicode Normalization Form C (NFC).
        salt is *ownersalt*. n=16384, r=8, p=8, length=32.
      - Call the resulting 32 bytes *prefactor*.
      - Take SHA256(SHA256(*prefactor* + *ownerentropy*)) and call this
        *passfactor*. The "+" operator is concatenation.
5.  Compute the elliptic curve point G \* *passfactor*, and convert the
    result to compressed notation (33 bytes). Call this *passpoint*.
    Compressed notation is used for this purpose regardless of whether
    the intent is to create Bitcoin addresses with or without compressed
    public keys.
6.  Convey *ownersalt* and *passpoint* to the party generating the keys,
    along with a checksum to ensure integrity.
      - The following Base58Check-encoded format is recommended for this
        purpose: magic bytes "2C E9 B3 E1 FF 39 E2 51" followed by
        *ownerentropy*, and then *passpoint*. The resulting string will
        start with the word "passphrase" due to the constant bytes, will
        be 72 characters in length, and encodes 49 bytes (8 bytes
        constant + 8 bytes *ownerentropy* + 33 bytes *passpoint*). The
        checksum is handled in the Base58Check encoding. The resulting
        string is called *intermediate\_passphrase\_string*.

If lot and sequence numbers are not being included, then follow the same
procedure with the following changes:

  - *ownersalt* is 8 random bytes instead of 4, and *lotsequence* is
    omitted. *ownerentropy* becomes an alias for *ownersalt*.
  - The SHA256 conversion of *prefactor* to *passfactor* is omitted.
    Instead, the output of scrypt is used directly as *passfactor*.
  - The magic bytes are "2C E9 B3 E1 FF 39 E2 53" instead (the last byte
    is 0x53 instead of 0x51).

Steps to create new encrypted private keys given
*intermediate\_passphrase\_string* from *owner* (so we have
*ownerentropy*, and *passpoint*, but we do not have *passfactor* or the
passphrase):

1.  Set *flagbyte*.
      - Turn on bit 0x20 if the Bitcoin address will be formed by
        hashing the compressed public key (optional, saves space, but
        many Bitcoin implementations aren't compatible with it)
      - Turn on bit 0x04 if *ownerentropy* contains a value for
        *lotsequence*. (While it has no effect on the keypair generation
        process, the decryption process needs this flag to know how to
        process *ownerentropy*)
2.  Generate 24 random bytes, call this *seedb*. Take
    SHA256(SHA256(*seedb*)) to yield 32 bytes, call this *factorb*.
3.  ECMultiply *passpoint* by *factorb*. Use the resulting EC point as a
    public key and hash it into a Bitcoin address using either
    compressed or uncompressed public key methodology (specify which
    methodology is used inside *flagbyte*). This is the generated
    Bitcoin address, call it *generatedaddress*.
4.  Take the first four bytes of SHA256(SHA256(*generatedaddress*)) and
    call it *addresshash*.
5.  Now we will encrypt *seedb*. Derive a second key from *passpoint*
    using scrypt
      - Parameters: *passphrase* is *passpoint* provided from the first
        party (expressed in binary as 33 bytes). *salt* is *addresshash*
        + *ownerentropy*, n=1024, r=1, p=1, length=64. The "+" operator
        is concatenation.
      - Split the result into two 32-byte halves and call them
        *derivedhalf1* and *derivedhalf2*.
6.  Do AES256Encrypt(block = (seedb\[0...15\] xor
    derivedhalf1\[0...15\]), key = derivedhalf2), call the 16-byte
    result *encryptedpart1*
7.  Do AES256Encrypt(block = ((encryptedpart1\[8...15\] +
    seedb\[16...23\]) xor derivedhalf1\[16...31\]), key = derivedhalf2),
    call the 16-byte result *encryptedpart2*. The "+" operator is
    concatenation.

The encrypted private key is the Base58Check-encoded concatenation of
the following, which totals 39 bytes without Base58 checksum:

  - 0x01 0x43 + *flagbyte* + *addresshash* + *ownerentropy* +
    *encryptedpart1*\[0...7\] + *encryptedpart2*

##### Confirmation code

The party generating the Bitcoin address has the option to return a
*confirmation code* back to *owner* which allows *owner* to
independently verify that he has been given a Bitcoin address that
actually depends on his passphrase, and to confirm the lot and sequence
numbers (if applicable). This protects *owner* from being given a
Bitcoin address by the second party that is unrelated to the key
derivation and possibly spendable by the second party. If a Bitcoin
address given to *owner* can be successfully regenerated through the
confirmation process, *owner* can be reasonably assured that any
spending without the passphrase is infeasible. This confirmation code is
75 characters starting with "cfrm38".

To generate it, we need *flagbyte*, *ownerentropy*, *factorb*,
*derivedhalf1* and *derivedhalf2* from the original encryption
operation.

1.  ECMultiply *factorb* by G, call the result *pointb*. The result is
    33 bytes.
2.  The first byte is 0x02 or 0x03. XOR it by (derivedhalf2\[31\] &
    0x01), call the resulting byte *pointbprefix*.
3.  Do AES256Encrypt(block = (pointb\[1...16\] xor
    derivedhalf1\[0...15\]), key = derivedhalf2) and call the result
    *pointbx1*.
4.  Do AES256Encrypt(block = (pointb\[17...32\] xor
    derivedhalf1\[16...31\]), key = derivedhalf2) and call the result
    *pointbx2*.
5.  Concatenate *pointbprefix* + *pointbx1* + *pointbx2* (total 33
    bytes) and call the result *encryptedpointb*.

The result is a Base58Check-encoded concatenation of the following:

  - 0x64 0x3B 0xF6 0xA8 0x9A + *flagbyte* + *addresshash* +
    *ownerentropy* + *encryptedpointb*

A confirmation tool, given a passphrase and a confirmation code, can
recalculate the address, verify the address hash, and then assert the
following: "It is confirmed that Bitcoin address *address* depends on
this passphrase". If applicable: "The lot number is *lotnumber* and the
sequence number is *sequencenumber*."

To recalculate the address:

1.  Derive *passfactor* using scrypt with *ownerentropy* and the user's
    passphrase and use it to recompute *passpoint*
2.  Derive decryption key for *pointb* using scrypt with *passpoint*,
    *addresshash*, and *ownerentropy*
3.  Decrypt *encryptedpointb* to yield *pointb*
4.  ECMultiply *pointb* by *passfactor*. Use the resulting EC point as a
    public key and hash it into *address* using either compressed or
    uncompressed public key methodology as specifid in *flagbyte*.

##### Decryption

1.  Collect encrypted private key and passphrase from user.
2.  Derive *passfactor* using scrypt with *ownersalt* and the user's
    passphrase and use it to recompute *passpoint*
3.  Derive decryption key for *seedb* using scrypt with *passpoint*,
    *addresshash*, and *ownerentropy*
4.  Decrypt *encryptedpart2* using AES256Decrypt to yield the last 8
    bytes of *seedb* and the last 8 bytes of *encryptedpart1*.
5.  Decrypt *encryptedpart1* to yield the remainder of *seedb*.
6.  Use *seedb* to compute *factorb*.
7.  Multiply *passfactor* by *factorb* mod N to yield the private key
    associated with *generatedaddress*.
8.  Convert that private key into a Bitcoin address, honoring the
    compression preference specified in the encrypted key.
9.  Hash the Bitcoin address, and verify that *addresshash* from the
    encrypted private key record matches the hash. If not, report that
    the passphrase entry was incorrect.

## Backwards compatibility

Backwards compatibility is minimally applicable since this is a new
standard that at most extends [Wallet Import
Format](Wallet_Import_Format "wikilink"). It is assumed that an entry
point for private key data may also accept existing formats of private
keys (such as hexadecimal and [Wallet Import
Format](Wallet_Import_Format "wikilink")); this draft uses a key format
that cannot be mistaken for any existing one and preserves
auto-detection capabilities.

## Suggestions for implementers of proposal with alt-chains

If this proposal is accepted into alt-chains, it is requested that the
unused flag bytes not be used for denoting that the key belongs to an
alt-chain.

Alt-chain implementers should exploit the address hash for this purpose.
Since each operation in this proposal involves hashing a text
representation of a coin address which (for Bitcoin) includes the
leading '1', an alt-chain can easily be denoted simply by using the
alt-chain's preferred format for representing an address. Alt-chain
implementers may also change the prefix such that encrypted addresses do
not start with "6P".

## Discussion item: scrypt parameters

This proposal leaves the scrypt parameters up in the air. The following
items are proposed for consideration:

The main goal of scrypt is to reduce the feasibility of brute force
attacks. It must be assumed that an attacker will be able to use an
efficient implementation of scrypt. The parameters should force a highly
efficient implementation of scrypt to wait a decent amount of time to
slow attacks.

On the other hand, an unavoidably likely place where scrypt will be
implemented is using slow interpreted languages such as javascript. What
might take milliseconds on an efficient scrypt implementation may take
seconds in javascript.

It is believed, however, that someone using a javascript implementation
is probably dealing with codes by hand, one at a time, rather than
generating or processing large batches of codes. Thus, a wait time of
several seconds is acceptable to a user.

A private key redemption process that forces a server to consume several
seconds of CPU time would discourage implementation by the server owner,
because they would be opening up a denial of service avenue by inviting
users to make numerous attempts to invoke the redemption process.
However, it's also feasible for the server owner to implement his
redemption process in such a way that the decryption is done by the
user's browser, offloading the task from his own server (and providing
another reason why the chosen scrypt parameters should be tolerant of
javascript-based decryptors).

The preliminary values of 16384, 8, and 8 are hoped to offer the
following properties:

  - Encryption/decryption in javascript requiring several seconds per
    operation
  - Use of the parallelization parameter provides a modest opportunity
    for speedups in environments where concurrent threading is available
    - such environments would be selected for processes that must handle
    bulk quantities of encryption/decryption operations. Estimated time
    for an operation is in the tens or hundreds of milliseconds.

## Reference implementation

Added to alpha version of Casascius Bitcoin Address Utility for Windows
available at:

  - via https: <https://casascius.com/btcaddress-alpha.zip>
  - at github: <https://github.com/casascius/Bitcoin-Address-Utility>

Click "Tools" then "PPEC Keygen" (provisional name)

## Other implementations

  - Javascript - <https://github.com/bitcoinjs/bip38>

## Test vectors

### No compression, no EC multiply

Test 1:

  - Passphrase: TestingOneTwoThree
  - Encrypted:
    6PRVWUbkzzsbcVac2qwfssoUJAN1Xhrg6bNk8J7Nzm5H7kxEbn2Nh2ZoGg
  - Unencrypted (WIF):
    5KN7MzqK5wt2TP1fQCYyHBtDrXdJuXbUzm4A9rKAteGu3Qi5CVR
  - Unencrypted (hex):
    CBF4B9F70470856BB4F40F80B87EDB90865997FFEE6DF315AB166D713AF433A5

Test 2:

  - Passphrase: Satoshi
  - Encrypted:
    6PRNFFkZc2NZ6dJqFfhRoFNMR9Lnyj7dYGrzdgXXVMXcxoKTePPX1dWByq
  - Unencrypted (WIF):
    5HtasZ6ofTHP6HCwTqTkLDuLQisYPah7aUnSKfC7h4hMUVw2gi5
  - Unencrypted (hex):
    09C2686880095B1A4C249EE3AC4EEA8A014F11E6F986D0B5025AC1F39AFBD9AE

Test 3:

  - Passphrase ϓ␀𐐀💩 (`\u03D2\u0301\u0000\U00010400\U0001F4A9`; [GREEK
    UPSILON WITH HOOK](http://codepoints.net/U+03D2), [COMBINING ACUTE
    ACCENT](http://codepoints.net/U+0301),
    [NULL](http://codepoints.net/U+0000), [DESERET CAPITAL LETTER LONG
    I](http://codepoints.net/U+10400), [PILE OF
    POO](http://codepoints.net/U+1F4A9))
  - Encrypted key:
    6PRW5o9FLp4gJDDVqJQKJFTpMvdsSGJxMYHtHaQBF3ooa8mwD69bapcDQn
  - Bitcoin Address: 16ktGzmfrurhbhi6JGqsMWf7TyqK9HNAeF
  - Unencrypted private key (WIF):
    5Jajm8eQ22H3pGWLEVCXyvND8dQZhiQhoLJNKjYXk9roUFTMSZ4
  - *Note:* The non-standard UTF-8 characters in this passphrase should
    be NFC normalized to result in a passphrase of
    `0xcf9300f0909080f09f92a9` before further processing

### Compression, no EC multiply

Test 1:

  - Passphrase: TestingOneTwoThree
  - Encrypted:
    6PYNKZ1EAgYgmQfmNVamxyXVWHzK5s6DGhwP4J5o44cvXdoY7sRzhtpUeo
  - Unencrypted (WIF):
    L44B5gGEpqEDRS9vVPz7QT35jcBG2r3CZwSwQ4fCewXAhAhqGVpP
  - Unencrypted (hex):
    CBF4B9F70470856BB4F40F80B87EDB90865997FFEE6DF315AB166D713AF433A5

Test 2:

  - Passphrase: Satoshi
  - Encrypted:
    6PYLtMnXvfG3oJde97zRyLYFZCYizPU5T3LwgdYJz1fRhh16bU7u6PPmY7
  - Unencrypted (WIF):
    KwYgW8gcxj1JWJXhPSu4Fqwzfhp5Yfi42mdYmMa4XqK7NJxXUSK7
  - Unencrypted (hex):
    09C2686880095B1A4C249EE3AC4EEA8A014F11E6F986D0B5025AC1F39AFBD9AE

### EC multiply, no compression, no lot/sequence numbers

Test 1:

  - Passphrase: TestingOneTwoThree
  - Passphrase code:
    passphrasepxFy57B9v8HtUsszJYKReoNDV6VHjUSGt8EVJmux9n1J3Ltf1gRxyDGXqnf9qm
  - Encrypted key:
    6PfQu77ygVyJLZjfvMLyhLMQbYnu5uguoJJ4kMCLqWwPEdfpwANVS76gTX
  - Bitcoin address: 1PE6TQi6HTVNz5DLwB1LcpMBALubfuN2z2
  - Unencrypted private key (WIF):
    5K4caxezwjGCGfnoPTZ8tMcJBLB7Jvyjv4xxeacadhq8nLisLR2
  - Unencrypted private key (hex):
    A43A940577F4E97F5C4D39EB14FF083A98187C64EA7C99EF7CE460833959A519

Test 2:

  - Passphrase: Satoshi
  - Passphrase code:
    passphraseoRDGAXTWzbp72eVbtUDdn1rwpgPUGjNZEc6CGBo8i5EC1FPW8wcnLdq4ThKzAS
  - Encrypted key:
    6PfLGnQs6VZnrNpmVKfjotbnQuaJK4KZoPFrAjx1JMJUa1Ft8gnf5WxfKd
  - Bitcoin address: 1CqzrtZC6mXSAhoxtFwVjz8LtwLJjDYU3V
  - Unencrypted private key (WIF):
    5KJ51SgxWaAYR13zd9ReMhJpwrcX47xTJh2D3fGPG9CM8vkv5sH
  - Unencrypted private key (hex):
    C2C8036DF268F498099350718C4A3EF3984D2BE84618C2650F5171DCC5EB660A

### EC multiply, no compression, lot/sequence numbers

Test 1:

  - Passphrase: MOLON LABE
  - Passphrase code:
    passphraseaB8feaLQDENqCgr4gKZpmf4VoaT6qdjJNJiv7fsKvjqavcJxvuR1hy25aTu5sX
  - Encrypted key:
    6PgNBNNzDkKdhkT6uJntUXwwzQV8Rr2tZcbkDcuC9DZRsS6AtHts4Ypo1j
  - Bitcoin address: 1Jscj8ALrYu2y9TD8NrpvDBugPedmbj4Yh
  - Unencrypted private key (WIF):
    5JLdxTtcTHcfYcmJsNVy1v2PMDx432JPoYcBTVVRHpPaxUrdtf8
  - Unencrypted private key (hex):
    44EA95AFBF138356A05EA32110DFD627232D0F2991AD221187BE356F19FA8190
  - Confirmation code:
    cfrm38V8aXBn7JWA1ESmFMUn6erxeBGZGAxJPY4e36S9QWkzZKtaVqLNMgnifETYw7BPwWC9aPD
  - Lot/Sequence: 263183/1

Test 2:

  - Passphrase (all letters are Greek - test UTF-8 compatibility with
    this): ΜΟΛΩΝ ΛΑΒΕ
  - Passphrase code:
    passphrased3z9rQJHSyBkNBwTRPkUGNVEVrUAcfAXDyRU1V28ie6hNFbqDwbFBvsTK7yWVK
  - Encrypted private key:
    6PgGWtx25kUg8QWvwuJAgorN6k9FbE25rv5dMRwu5SKMnfpfVe5mar2ngH
  - Bitcoin address: 1Lurmih3KruL4xDB5FmHof38yawNtP9oGf
  - Unencrypted private key (WIF):
    5KMKKuUmAkiNbA3DazMQiLfDq47qs8MAEThm4yL8R2PhV1ov33D
  - Unencrypted private key (hex):
    CA2759AA4ADB0F96C414F36ABEB8DB59342985BE9FA50FAAC228C8E7D90E3006
  - Confirmation code:
    cfrm38V8G4qq2ywYEFfWLD5Cc6msj9UwsG2Mj4Z6QdGJAFQpdatZLavkgRd1i4iBMdRngDqDs51
  - Lot/Sequence: 806938/1
