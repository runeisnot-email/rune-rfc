# The Rune is Not Email (RUNE) protocol

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
communication mechanism that behaves in a similar way to email, but provides
strong privacy and anonymity guarantees, including safety from metadata analysis
attacks and compromised servers.


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

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC
2119](https://datatracker.ietf.org/doc/html/rfc2119). Only UPPERCASE usage of
these keywords have the defined special meanings, as described in
[RFC 8174](https://datatracker.ietf.org/doc/html/rfc8174).


# Protocol Overview

Rune allows for private and anonymous communication between two parties in much
a similar fashion to that of traditional email. The messages however, are
end-to-end encrypted with private keys generated and kept secret by each client.
Messages are stored in a centralized server in such a way as to minimize the
amount of metadata and information that in contains.

The protocol contemplates four major operations:

- Register a client
- Send a message
- Manage mailbox: list, retrieve, delete messages

A registered client sends a message by asking the server for the public key of
the receiver address. Signs the message using it's own private key and then
encrypts the message using the public key of the receiver. The server
concatenates the encrypted message with the sender address, the sending time and
date, sings it with it's private key, and encrypts it again with the public key
of the receiver before storing it for later retrieval upon the receiver
request.

<!-- TODO: graph -->

Stored messages on the server can only be read by the receiver using it's
private key, including it's sender address and other metadata. This way both
message and sender address are only known to the receiver. The server can only
known which messages are for which receivers and nothing else.

## Addresses

Each registered client possess an address of the form <USERNAME>@<DOMAIN>, where
both *USERNAME* and *DOMAIN* are arbitrary 3-50 symbol alphanumeric strings. The
*DOMAIN* does not need to be a valid nor registered Internet domain name. It
serves the only purpose of identifying the addresses namespace of a server. It
is RECOMMENDED that the *DOMAIN* does not correspond to a valid or registered
Internet domain name for anonymity reasons. Both *USERNAME* and *DOMAIN* MAY be
randomly generated for anonymity reasons.


# Operations

The client can issue commands to the server only after receiving the greeting
message of the form:

    HOWDY [server configurable welcome message]


## Register a Client

The client MUST generate a private/public key pair. The server MUST allow for
two possible (configurable) ways of allowing client registration: free
registration and via invitation token.

### Free Registration

A client can register a new address on the server by issuing the *REGISTER*
command with the desired address to create, and it's public key. The DOMAIN must
be the same one that the server is configured to accept:

    REGISTER <USERNAME>@<DOMAIN> <client public key...>

If the server is configured to not allow free registration it will respond with
a status code:

    405 free registration not available

Otherwise, the server will respond with a registration challenge.


### Invitation Token Registration

An invited client can register a new address on the server by issuing the
*REGISTER_INVITATION* command with the desired address to create, and it's
public key. The DOMAIN must be the same one that the server is configured to
accept. Every invitation token is bounded to a particular address upon creation,
so it is REQUIRED to match in this request (The `Invitation Token` explains how
it's created).

    REGISTER_INVITATION <USERNAME>@<DOMAIN> <invitation token> <client public key...>

If the server is configured to not allow invitation registration it will respond
with a status code:

    406 invitation registration not available

If the invitation token is invalid or expired, the server will respond
with a respective status code and immediately close the connection:

    407 invitation token invalid

    408 invitation token expired

Otherwise, the server will respond with a registration challenge.

### Registration Challenge

Upon success of either a *REGISTER* o *REGISTER_INVITATION* command, the server
responds with a registration challenge encrypted with the provided client's
public key and signed with the server's public key, which is provided at the end
of the response:

    205 <challenge string> <server public key...>

The client must then verify the signature using the server's public key, decrypt
the challenge string using it's own private key and give back the challenge
string encrypted with the server's public key using the command:

    CHALLANGE_ACCEPTED <re-encrypted challenge string>

If the challenge is passed the server responds with a status code:

    210 registered <USERNAME>@<DOMAIN>

Otherwise the server responds with a status code and immediately close the
connection:

    410 registration challenge failed


## Send a Message

## Manage Mailbox



# Security considerations

- Every communication between client and server MUST be over TLS 1.2 or grater.

- The client SHOULD generate a new private/public key pair that is specifically
  and only used for Rune messaging and no other purposes. Furthermore every
  registered address SHOULD use a different private/public key pair.
