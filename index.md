Packet Security for Stateless Server Transactions (PSSST) is a light weight cryotographic
protocol to allow clients to securely send messages to a server or pool of servers,
and receive secure responses, in such a way that the servers do not need to maintain
any state in between calls. Unlike sessions-based protocols suchh as TLS (and SSL),
the protection of each request is self-contained and there is no "handshake", so
there is no added network latency when sending a request. Most importantly, since
there is no state kept at the server there is no need for complex load balancing
when a pool of servers is used. This allows systems to scale to millions of clients
relatively cheaply. PSSST also includes support for optional cryptographic authentication
of the client making the request (with very little added burden on the client when in
use), making it well suited to IoT applications.

Key features of PSSST are:

- No per-client state kept by servers between transactions
- No added network round-trips
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

PSSST does not set out to be the be all and end all of protocols; it
is designed to be very good for a certain subset of problems. To find
out more about what the protocol aims to do, and what it does not aim
to do, check out the [goals and non-goals](goals-non-goals.md).


## Protocol detail

Find out more about the details of the [PSSST protocol](protocol.md).

## Implementations:

There are several open source implementations of PSSST. The ones that we
know about are:

* Go: [gopssst](https://github.com/PSSST-Protocol/gopssst)
* Python: [pypssst](https://github.com/PSSST-Protocol/pypssst)
* Rust: [pssst.rs](https://github.com/ctz/pssst.rs)
