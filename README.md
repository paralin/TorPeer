# TorPeer

P2P discovery & communcation protocol via hidden services on [Tor](https://www.torproject.org/).

Concept
=======

Standard P2P discovery requires routers, forwarded ports/NAT hole punching,
and reveals client IP addresses. In situations where we'd rather not reveal
IP addresses, are operating in a situation without reliable NAT
punchthrough/dedicated routing servers, and have a relatively small number
of clients, it makes sense to piggyback on the Tor network for peer to peer
communication.

There's a small problem here, though. While hidden services are a great
way to talk to other nodes in the network by name and it's fairly
simple to share the addresses of others in the network, there's not much
in the way of peer discovery via Tor already written. This project aims
to define a super-simple protocol for requesting and sharing connection
information within the Tor network.

The basic idea is that with a few initial nodes, a client can request a
list of other known online/offline nodes in order to make additional
connections later. Each node keeps a table of known online hosts, which
it has either directly contacted, or has had contact with via other
hosts. As a potential DOS for this system would be flooding invalid
connection addresses to incoming connections, the system requires
signed timestampped verification messages from each host to verify that
they actually exist.

All communication is done with [proto3][protodocs]. Over HTTP transports, JSON
encoding via proto3 is supported. Generally this system is designed for
direct binary communication, though. HTTP is used usually just to grab
an initial list of peers to try, especially for dedicated routers.

[protodocs]: https://developers.google.com/protocol-buffers/docs/proto3

Identification
==============

Each node in the network has identification composed of two parts:

 - Tor hidden service address
 - Computed proof-of-work paired with the identity.

Tor handles message verification internally via keypair with hidden
services. As such, other sign/verify methods are redundant. However,
it's possible to flood the network with new hidden service addresses
(these are very easy to generate). The system can use a configurable
difficulty proof-of-work derived from the hidden service address to make
sure it's not too inexpensive to re-create identities.

There are currently the following archetype nodes:

 - Participants: care about peering events as well as network traffic.
 - Routers: connect to participants but only care about peering events.

Implementation
==============

Reference implementation is written in C++.

Interaction with Tor
====================

It is expected that Tor be configured externally from all of the
implementations. A hidden service configuration is necessary. Certain
elements of this configuration, plus stored proof-of-work, must be
passed into the library when initialized, or generated using included
helper methods.

Authors
=======

Christian Stewart <christian@paral.in>
