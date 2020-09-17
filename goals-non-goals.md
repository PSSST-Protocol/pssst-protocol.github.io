# Goals and non-goals

Whenever one seeks to design a protocol it is important to set out what the
goals and non-goals of the protocol are; this is especially the case when one
is designing within tight constraints. For security protocols this includes
not just modelling the threats one aims to protect against but also
explicitly stating the types of threat one is {\em not} aiming to protect
against, so that there is no doubt among users as to what protection they can
expect.

## Goals of PSSST

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

## Non-goals

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


