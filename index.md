# The PSSST Protocol

Packet Security for Stateless Server Transactions (PSSST) is a light weight cryotographic
protocol to allow clients to securely send messages to a server or pool of servers,
and receive secure responses, in such a way that the servers do not need to maintain
any state in between calls. Unlike sessions-based protocols suchh as TLS (and SSL),
the protection of each request is self-contained and there is no "handshake", so
there is no added network latency when sending a request. Most importantly, since
there is no state kept at the server there is no need for complex load balancing
when a pool of servers is used. This allows systems to scale to millions of clients
relatively cheaply. PSSST also includes optional support for cryptographic authentication
of the client making the request (with very little added burden on the client when in
use), making it well suited to IoT applications.

Key features of PSSST are:

- No per-client state kept by servers between transactions
- No added network round-trips for handshake
- Small data overhead
- Optional client authentication

## Why do we need another protocol?

Cloud computing, and the ability to add new servers on demand, has opened up
the opportunity for businesses to deploy services with unprecedented scale
and flexibility. Scaling by adding more servers is not without its
challenges, however. In particular this sort of horizontal scaling requires
load to be distributed across all the servers. The problem can be mitigated
somewhat by making the application servers as stateless as possible, so that
requests to be sent to any server.

Unfortunately most modern security protocols are not designed to be
stateless. Protocols such as TLS require multiple packets to perform their
handshakes; the cost of these may then be amortised over the duration of the
connection. In some situations the cost of setting up a connection might be
amortised over multiple transactions but in order to do that it is necessary
to persist state not just for the duration of the transaction but also
between transactions. To achieve this sort of persistence, Cloud operators
need to offer load balancing systems that keep track of client connections,
binding clients to specific servers for the duration of a request, holding
session resumption state between requests and determining if and when this
state may be expired and discarded. This sort of load balancing costs a
significant amount to deploy and operate and as a result load balancing
services from Cloud operators often carry a high price.


When requests and replies are large or the processing cost for a transaction
is substantial then the high price of load balancing may not be a
major factor. However there are many situations where requests and replies are
small and processing is minimal, for instance with IoT device or sensor
logging, clients polling for state changes and the calling of lightweight
functions. It is also the case that for many such applications the request
and reply can each fit within a single datagram and retrying in the case of
packet loss can be handled separately. In these situations load balancing can
become the dominant cost for deploying a service.

Packet Security for Stateless Server Transactions (PSSST) sets out to address
the security and performance needs of these lightweight call-ins. The
protocol is designed for datagram communication so as to have low processing
requirements, minimal data transfer demands, and zero server-side state
retention between calls, allowing for large scale deployment of lightweight
services with extremely simple (and cheap) load balancing.

## Goals and non-goals

Whenever one seeks to design a protocol it is important to set out what the
goals and non-goals of the protocol are; this is especially the case when one
is designing within tight constraints. For security protocols this includes
not just modelling the threats one aims to protect against but also
explicitly stating the types of threat one is {\em not} aiming to protect
against, so that there is no doubt among users as to what protection they can
expect.

### Goals}

#### Stateless server-side implementation:

In order to allow completely stateless load distribution, a key design goal
of PSSST is that servers responding to single datagram requests should not be
required to maintain _any_ state about the requesting client when it is
not processing that request and preparing a reply, other than optionally
having access to an identity database if client authentication is required.

#### Light weight:

Low communication overhead is generally desirable in any protocol and
especially so for frequent, small communications. PSSST is expected to be
used for small communication, so a large overhead can lead to very large
message expansion. Low communication overhead is also particularly important
when, due to the stateless nature of the server, any protocol overhead can
not be amortised over longer sessions.

#### No extra packets:

As a corollary to the need to be light weight when handling large numbers of
small messages _that can fit in one packet_, coupled with the stateless
nature of the protocol requiring any handshake to be re-run for each
transaction, a need for any extra packets to be sent by the client would at
least double the number of packets to be processed. Of course any protocol
that did require more than one packet to be sent would, of necessity, violate
the goal of being stateless, but we call this out explicitly due to the fact
that this would be a goal even without the requirement to be stateless. We also
explicitly support the case in which a client does not expect a reply to
every request.
 
#### Confidentiality and integrity against active MITM attack:

Request packets from a client to a server and replies from the server to the
client must not be readable by an active adversary, nor should they be
malleable by an active adversary without detection.

#### Replay resistance at the client:

When a client receives a reply to a pending request, it must be able to
detect if the reply was to a different request, even if the received reply
was a correctly formed and authenticated reply to a different request from
the same server that is being replayed.

#### Support for optional client authentication

Requests from clients may either include cryptographic authentication of the
identity of the requesting client or may be anonymous.

#### Request are unlinkable:

An attacker viewing request and reply packets must not be able to identify the client in the transaction and must also not be able to determine if two transactions were instigated by the same party or not, irrespective of if client authentication was used. Note that this goal is limited to the PSSST protocol itself and not to the network transport of the packets, which may be subject to other sorts of traffic analysis.

### Non-goals

#### No replay resistance at the server:

Servers are _not_ expected to be able to detect if a request from a
client is a reply of a previous request. There are two reasons that this is a
non-goal. Firstly, server-side replay detection, of necessity, requires
servers to maintain at least some knowledge of the past (and thus
state). Furthermore, single-packet datagram requests are likely to be sent
over UDP, which is an inherently _unreliable_ protocol. In the event that
a client sends a request but does not receive a reply it is useful for the
client to simply send the same packet again. That way, as long as the
transaction processing is idempotent it will not matter if the packet loss
was with the request packet or the reply.

#### No Perfect Forward Secrecy:

While Perfect Forward Secrecy is a highly desirable characteristic of a
secure communication protocol, it is unfortunately incompatible with being
truly stateless. This is perhaps ironic, or even unexpected, given that the
server is required to dispose of all state about a transaction once it is
complete but we show in the security analysis section that it must be so for
information theoretic reasons. We can (and do) however deliver
_imperfect_ forward secrecy, in the sense that while a compromise of the
server keys can allow past communications to be decrypted, a compromise of a
client's keys does not allow past or future communications from the same
client to be decrypted. For many remote sensor or IoT applications this
asymmetric security may be sufficient, since IoT devices in remote and
possibly hostile locations are probably more likely to be compromised than
well-run cloud servers.

#### No certificate distribution

The final non-goal of PSSST is support for the distribution of client or
server public keys, or their certificates, as part of the
protocol. Certificates are large and change fairly slowly; PSSST is designed
from small but frequent communications. The client is expected to have a current
and authentic copy of the server's public key before it begins a transaction
and if client authentication is being used the server is expected to already
know each client's public key. In practice keys do get periodically changed
and occasionally revoked, so clients and servers are expected to use some
other channel to ensure to ensure that they are using up-to-date keys. We
propose that clients might infrequently check in with the server using a much
heavier-weight protocol such as TLS to exchange current key information.



### Protocol detail

Find out more about the details of the [PSSST protocol](protocol.md).

### Implementations:

There are several open source implementations of PSSST. The ones that we
know about are:

* Go: [gopssst](https://github.com/PSSST-Protocol/gopssst)
* Python: [pypssst](https://github.com/PSSST-Protocol/pypssst)
* Rust: [pssst.rs](https://github.com/ctz/pssst.rs)
