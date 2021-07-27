---
title: The Rune is Not Email (RUNE) protocol RFC
subtitle: Draft
lang: en
numbersections: true
...

<!-- # The Rune is Not Email (RUNE) protocol -->

# Introduction

**NOTE:** This is a draft, incomplete proposal.

Standard Email, as defined by the SMTP [[RFC
821](https://datatracker.ietf.org/doc/html/rfc821)], POP3 [[RFC
1939](https://datatracker.ietf.org/doc/html/rfc1939)] and IMAP [[RFC
3501](https://datatracker.ietf.org/doc/html/rfc3501)] protocols, suffers from
various privacy and anonymity issues, the most obvious and easy to solve being:

- When TLS *is not* used, any Eavesdropper is able to read the messages.

- When TLS *is* used, email servers are still able to read the messages.

- When end-to-end encryption *is not* used, email servers are able to read
  messages.

However:

- When end-to-end encryption *is* used, email servers are still able to breach
  privacy and anonymity to various degrees via email metadata and headers: IP
  addresses, timestamps, graphs and volume of interactions, etc.

- When trusted email servers are used, privacy and anonymity guaranties suffer
  during inter-domain email transmissions to other servers.

- When a trusted email server is used for inter-domain exclusive communication,
  privacy and anonymity suffers if the server is compromised.


Rune IS NOT a substitute for standard Email, conversely it aims to provide a
communication mechanism that behaves akin to email, but provides strong privacy
and anonymity guarantees, including safety from metadata analysis attacks and
compromised servers.


## What Rune is not

Rune is Not Email:

- It cannot communicate with email servers nor does it use any standard email
  protocol
- It cannot receive traditional email
- It cannot send traditional email

## What Rune is

- It provides private and anonymous communication between two parties via
  email-like messages
- It can send and receive intra-domain messages only
- It provides privacy and anonymity guarantees for both communication initiators
  and recipients
- It aims to preserve some privacy and anonymity guarantees in the face of a
  compromised server


# Terminology

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC
2119](https://datatracker.ietf.org/doc/html/rfc2119). Only UPPERCASE usage of
these keywords have the defined special meanings, as described in [RFC
8174](https://datatracker.ietf.org/doc/html/rfc8174).


# Security requirements

- Every communication between client and server MUST occur over TLS 1.2 or
  grater.

- The client SHOULD generate a new private/public key pair that is specifically
  and only used for Rune messaging and no other purposes. Furthermore, every
  registered address SHOULD use a different private/public key pair.

- The server MUST NOT generate private/public key pairs for its clients.

- The server MUST only store the encrypted messages in the receiver's mailbox
  and no additional metadata.

- The server MAY set file system date and time metadata in the receiver's
  mailbox to random dates after each access operation.



# Protocol Overview

Rune allows for private and anonymous communication between two parties in much
a similar fashion to that of traditional email. The messages however, are
end-to-end encrypted with private keys generated and kept secret by each client.
Messages are stored in a centralized server in such a way as to minimize the
amount of metadata and information that in contains.

The protocol contemplates three major operations:

- Register a client
- Send a message
- Manage mailbox: list, retrieve and delete messages

A new client registers on the server with a new address and a public key, either
freely or by using an invitation token. A registered client can send end-to-end
encrypted messages to other addresses of the same domain as well as manage its
own mailbox.

A registered client sends a message by asking the server for the public key of
the receiver address. Signs the message using its own private key and then
encrypts the message using the public key of the receiver. The server
concatenates the encrypted message with the sender address, the sending time and
date; Sings it with its private key and encrypts it again with the public key of
the receiver before storing it for later retrieval upon the receiver request.

Stored messages on the server can only be retrieved and read by the receiver
using its private key, including its sender address and other metadata. This way
both message and sender address are only known to the receiver. The server can
only know which messages are for which receivers and nothing else.

## Addresses

Each registered client possess an address of the form \<USERNAME\>@\<DOMAIN\>,
where both *USERNAME* and *DOMAIN* are arbitrary 3-50 symbol alphanumeric
strings that may contain dots, hyphens and underscores. The *DOMAIN* does not
need to be a valid nor registered Internet domain name. It serves the only
purpose of identifying the address namespace of a Rune server. It is RECOMMENDED
that the *DOMAIN* does not correspond to a valid or registered Internet domain
name for anonymity reasons. Both *USERNAME* and *DOMAIN* MAY be randomly
generated for anonymity reasons.


# Operations

The client can issue commands to the server only after receiving the greeting
message of the form:

    HOWDY [supported protocol versions] [server configurable welcome message]


## Register a Client

The client MUST generate a private/public key pair. The server SHOULD allow for
two possible (configurable) ways of allowing client registration: free
registration and via invitation token. Once a client is successfully registered
the server stores its public key associated with its address.


### Free Registration

A client can register a new address on the server by issuing the *REGISTER*
command with the desired address to create, and it's public key. The DOMAIN must
be the same one that the server is configured to accept:

    REGISTER <USERNAME>@<DOMAIN> <client public key...>

If the server is configured to not allow free registration it will respond with
a status code:

    405 FREE REGISTRATION NOT AVAILABLE

Otherwise, the server will respond with a registration challenge.


### Registration by Invitation Token

An invited client can register a new address on the server by issuing the
*REGISTER_INVITATION* command with the desired address to create, and it's
public key. The DOMAIN must be the same one that the server is configured to
accept. Every invitation token is bounded to a particular address upon creation,
so it is REQUIRED to match in this request (The `Invitation Token` section
explains how it's created).

    REGISTER_INVITATION <USERNAME>@<DOMAIN> <invitation token> <client public key...>

If the server is configured to not allow invitation registration it will respond
with a status code:

    416 INVITATION REGISTRATION NOT AVAILABLE

If the invitation token is invalid or expired, the server will respond with a
respective status code and immediately close the connection:

    417 INVITATION TOKEN INVALID

    418 INVITATION TOKEN EXPIRED

Otherwise, the server will respond with a registration challenge.


### Registration Challenge

Upon success of either a *REGISTER* o *REGISTER_INVITATION* command, the server
responds with a registration challenge encrypted with the provided client's
public key and signed with the server's public key, which is provided at the end
of the response:

    215 <challenge string> <server public key...>

The client must then verify the signature using the server's public key, decrypt
the challenge string using its own private key and give back the challenge
string encrypted with the server's public key using the command:

    CHALLANGE_ACCEPTED <re-encrypted challenge string>

If the challenge is passed the server responds with a status code:

    210 REGISTERED <USERNAME>@<DOMAIN>

Otherwise, the server responds with a status code and immediately close the
connection:

    410 REGISTRATION CHALLENGE FAILED



### Invitation Token

<!-- TODO -->


## Send a Message

A registered client can send a message to another registered client by issuing
the *SEND* command and providing the receiver address:

    SEND <RECEIVER_USERNAME>@<DOMAIN>

The server will respond with a success status code with the maximum payload in
MiB, the public key for the receiver address and the server's public key:

    201 <maximum payload> <receiver public key...> <server public key...>

The client can now use the *MESSAGE* command, specifying its own address, to
send the signed, encrypted message, known as the *CLIENT MESSAGE* (The `Message
Format` section explains how it's created).

    MESSAGE <SENDER_USERNAME>@<DOMAIN> <CLIENT MESSAGE>

The server verifies that the signature of the encrypted *CLIENT MESSAGE* is both
valid and matches the sender address, if that's the case it will then create a
*STORED MESSAGE* (The `Message Format` section explains how it's created), store
it in the receiver's mailbox and respond with a success status code.

    202 ROGER

Otherwise, it will respond with an error code and immediately close the
connection:

    405 SIGNATURE INVALID


### Message Format

The client creates a `MESSAGE` which encrypts and signs to create a `CLIENT
MESSAGE` that sends to the server. The server unwraps the `CLIENT MESSAGE`, adds
some metadata needed by the receiver, and encrypts this new data to create an
`STORED MESSAGE` that rests on the server for later receiver's retrieval.

#### Message & Client Message

The client transmits a signed, encrypted message with a maximum payload sized
imposed by server configuration and announced during the sending negotiation.

The original sender message is referred to as the `MESSAGE` which is encrypted
with the receiver's public key, and then wrapped in a `CLIENT MESSAGE` which is
then encrypted with the server's public key to be actually send to the server
with the `MESSAGE` command.

The `MESSAGE` is formed as the following UTF-8 encoded JSON:

```json
{
    "content": {
        "subject":     "User message subject",
        "body":        "User message body",

        "attachments": [
            {
                "filename": "Attachment file name",
                "base91":   "BASE91-encoded attachment"
            },
        ]
    },

    "signature": "signature of the concatenated {content} using the sender's private key"
}
```

Notice the absence of `to` and `from` metadata in the MESSAGE. That information
is respectively derived from: (1) the message is encrypted with the receiver's
public key and (2) the sender signs the message with its private key.


The `CLIENT MESSAGE` that is actually sent to the server conforms to the
following UTF-8 encoded JSON:

```json
{
    "content":   "Encrypted MESSAGE (entire JSON) using the receiver's public key",
    "signature": "signature of the {content} using the sender's private key"
}
```

The final `CLIENT MESSAGE` string is constructed by (1) encrypting the MESSAGE
entire JSON using the receiver's public key, (2) signing the result with the
sender's private key and (3) encrypting the CLIENT MESSAGE entire JSON with the
server's public key. This layering of encryption and signatures allows to
authenticate the sender to both the receiver and the server while hiding the
message from the server itself.


#### Stored Message

The server receives the encrypted `CLIENT MESSAGE`, decrypts it using its own
private key, and verifies the signature using the sender's public key.

Then constructs the `STORED MESSAGE` that conforms to the following UTF-8
encoded JSON:

```json
{
    "message": {
        "sender":      "Sender address",
        "received_at": "UTC ISO8601 encoded date",
        "content":     "Verbatim CLIENT MESSAGE's *content*"
    },

    "signature":   "signature of the {message} using the server's private key"
}
```

The final `STORED MESSAGE` string is constructed by (1) adding the
signature-authenticated sender's address and UTC date, (2) signing the result
with the server's private key and (3) encrypting the STORED MESSAGE entire JSON
with the receiver's public key. The `STORED MESSAGE` final encrypted string is
then stored in the receiver's mailbox with an associated randomly generated
UUID.



<!-- The server verifies that the signature of the encrypted *CLIENT MESSAGE* is both -->
<!-- valid and matches the sender address, if that's the case it will then create a -->
<!-- *STORED MESSAGE* (The `Message Format` section explains how it's created), store -->
<!-- it in the receiver's mailbox and respond with a success status code. -->





## Manage Mailbox

### List messages

### Retrieve a message

### Delete a message
