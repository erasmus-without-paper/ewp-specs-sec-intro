Authentication and Security
===========================

This document describes how EWP deals with integrity and confidentiality of the
messages exchanged.

* [What is the status of this document?][statuses]
* [See the index of all other EWP Specifications][develhub]


Introduction
------------

This version (1.x.x) of this document described the **initial** way we wanted
to handle authentication and security in EWP. We hoped that EWP will use TLS
for all security purposes (server authentication, client authentication,
encryption). This however
[proved](https://github.com/erasmus-without-paper/ewp-specs-architecture/issues/17)
to be a disputable choice, and a new major version of this document (2.x.x)
will be introduced in the near future.


Security Aspects
----------------

There are two actors in HTTP communication - the server and the client. We can
partition the security of this communication by splitting it into **four
security aspects**:

 * client authentication,
 * server authentication,
 * request encryption,
 * response encryption.

Each endpoint (also known as a "API URL") MUST support at least one "method"
of covering **each** of those security aspects. This choice is currently fixed
(see *Which should I implement?* section below), but this will change in the
2.x.x version of this document.


<a name="standard-methods"></a>

Standard methods
----------------

### Where are they defined?

EWP documentation includes a couple of **standard methods** for
request/response encryption and authentication. To see all of them, review the
list of chapters on the [Developers' Page][develhub].


### Which should I implement?

We currently require all servers to support a fixed set of security methods,
for all APIs:

 * Servers MUST use [TLS Client Certificate method][cliauth-tlscert] for
   authenticating clients. In case of some APIs, servers MAY also allow
   [anonymous clients][cliauth-none].

 * Clients MUST use [TLS Server Certificate method][srvauth-tlscert] for
   authenticating servers.

 * Both [request][reqencr-tls] and [response][resencr-tls] MUST be covered by
   regular TLS encryption only (no additional layer of encryption is allowed).

Note, that this will soon change (a 2.x.x version of this document is on the
way).


<a name="rules"></a>

About introducing new methods
-----------------------------

This section describes requirements **for the specifications** of security
methods, for each of the four security aspects covered by this document. This
means that you must read this chapter **only if you want to design a new
method** of handling a certain EWP security aspect.

These requirements apply also to the standard methods mentioned above (the
specifications of all standard methods MUST meet these requirements).


### Client Authentication

Each *client authentication method* SHOULD explicitly answer the following
questions:

 * How can the server verify which HEIs are covered by the requester?
 * How can the server verify that the request has not been tampered with, nor
   [replayed](https://en.wikipedia.org/wiki/Replay_attack) by a third party?
 * Does it provide non-repudiation? Can a server provide a solid proof later
   on, that a particular request took place, and that it originated from a
   certain client?


### Server Authentication

Each *client authentication method* SHOULD explicitly answer the following
questions:

 * How can the client verify the server's identity?
 * How can the client verify that the response has not been tampered with? Can
   it also verify that it was indeed generated for this particular request?
 * Does it provide non-repudiation? Can a client provide a solid proof later
   on, that the server sent a particular response (in response to a particular
   client's request)?


### Request Encryption

Each *request encryption method* SHOULD explicitly answer the following
questions:

 * How the server publishes his encryption key? How the client retrieves it
   securely?
 * How to encrypt and decrypt the request? Which parts are covered by the
   encryption and which are not?


### Response Encryption

Each *response encryption method* SHOULD explicitly answer the following
questions:

 * How the client delivers his encryption key to the server?
 * How to encrypt and decrypt the response? Which parts are covered by the
   encryption and which are not?


[discovery-api]: https://github.com/erasmus-without-paper/ewp-specs-api-discovery
[develhub]: http://developers.erasmuswithoutpaper.eu/
[statuses]: https://github.com/erasmus-without-paper/ewp-specs-management/blob/stable-v1/README.md#statuses
[registry]: https://registry.erasmuswithoutpaper.eu/
[registry-api]: https://github.com/erasmus-without-paper/ewp-specs-api-registry
[echo-api]: https://github.com/erasmus-without-paper/ewp-specs-api-echo
[dev-catalogue-xml]: https://dev-registry.erasmuswithoutpaper.eu/catalogue-v1.xml
[favoritism]: https://github.com/erasmus-without-paper/ewp-specs-management#server-favoritism
[ref-integrity-wiki]: https://en.wikipedia.org/wiki/Referential_integrity
[latin]: https://en.wikipedia.org/wiki/Basic_Latin_(Unicode_block)
[master-slave]: https://en.wikipedia.org/wiki/Master/slave_(technology)
[cliauth-none]: https://github.com/erasmus-without-paper/ewp-specs-sec-cliauth-none
[cliauth-tlscert]: https://github.com/erasmus-without-paper/ewp-specs-sec-cliauth-tlscert
[cliauth-httpsig]: https://github.com/erasmus-without-paper/ewp-specs-sec-cliauth-httpsig
[srvauth-tlscert]: https://github.com/erasmus-without-paper/ewp-specs-sec-srvauth-tlscert
[srvauth-httpsig]: https://github.com/erasmus-without-paper/ewp-specs-sec-srvauth-httpsig
[reqencr-tls]: https://github.com/erasmus-without-paper/ewp-specs-sec-reqencr-tls
[resencr-tls]: https://github.com/erasmus-without-paper/ewp-specs-sec-resencr-tls
