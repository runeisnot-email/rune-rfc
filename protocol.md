# The Rune is Not Email (RUNE) protocol

# Introduction

**NOTE:** This is a draft, incomplete proposal.

Standard Email, as defined by the SMTP
([rfc821](https://datatracker.ietf.org/doc/html/rfc821)), POP3
([RFC1939](https://datatracker.ietf.org/doc/html/rfc1939)) and IMAP
([RFC3501](https://datatracker.ietf.org/doc/html/rfc3501)) protocols, suffers
from various privacy and anonymity issues, the most obvious and easy to solve
being:

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

- It cannot communicate with email servers nor does it uses any standard email protocol
- It cannot receive traditional email
- It cannot send traditional email

## What Rune is

- It provides private and anonymous communication between two parties via email-like messages
- It can send and receive inter-domain messages
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
[RFC8174](https://datatracker.ietf.org/doc/html/rfc8174).
