Authentication and Security
===========================

This document describes how EWP deals with integrity and confidentiality of the
messages exchanged.

* [What is the status of this document?][statuses]
* [See the index of all other EWP Specifications][develhub]


Introduction
------------

This version (2.x.x) of this document describes the new way of handling
authentication and security in EWP.

Initially (in version 1.x.x), we hoped that EWP will use TLS for all security
purposes (server authentication, client authentication, encryption). This however
[proved](https://github.com/erasmus-without-paper/ewp-specs-architecture/issues/17)
to be a disputable choice.

As we analyzed all the consequences of this decision, we
[realized](https://github.com/erasmus-without-paper/ewp-specs-sec-intro/issues/1)
that we won't be able to require every API designer, and every partner in the
future, to stick with a single security solution indefinitely. This caused our
decision to split the initial security document into multiple "standalone"
parts, and to allow servers and clients to choose which protocols they
currently support, for each API separately.


Scope of this document
----------------------

It's important to understand, that each API decides of its own security options
independently.

 * Some APIs MAY require specific security protocols to be used. These APIs MAY
   refer to some of the "standard method" documents (described in separate EWP
   specifications), but they also MAY require implementers to follow some other
   protocols, not described in this document, nor any other EWP document.

 * If an API decides to follow security rules described in this document (or
   any other document), then it MUST state this explicitly. In other words,
   mere existence of this document in EWP specifications, does NOT imply that
   the following requirements must be satisfied by the implementers of all EWP
   APIs.

 * EWP designers MAY release new major versions of this document. Such new
   versions will not be compatible with the previous ones. That why it is very
   important for all APIs to explicitly state which *major* version of this
   document it follows.

Examples of statements which should appear in the specification of independent
APIs:

 - *For all endpoints of this API, implementers MUST follow the rules
   described in [EWP Authentication and Security, Version 1][sec-v1]
   document.*

 - *For all endpoints of this API, implementers MUST follow the rules
   described in [EWP Authentication and Security, Version 2][sec-v2]
   document.*

 - *For all endpoints of this API, clients MUST authenticate the servers with
   the [TLS Server Certificate method][srvauth-tlscert]. The servers are
   REQUIRED to [not validate the client][cliauth-none] (all clients MUST be
   allowed to access this API).*

This last statement implies, that such API is NOT following the requirements of
*Authentication and Security* document, but instead it requires the
implementers to follow the requirements of specific server and client
authentication methods (as described in their own repositories). Since nothing
is stated about encryption, it implies that regular TLS encryption is
sufficient.


Security Aspects
----------------

There are two actors in HTTP communication - the server and the client. We can
partition the security of this communication by splitting it into **four
security aspects**:

 * client authentication,
 * server authentication,
 * request encryption,
 * response encryption.

Each endpoint MUST support at least one method of covering **each** of
these security aspects. Before making the request, clients verify security
protocols supported by the server, and construct their request appropriately.


<a name="standard-methods"></a>

Standard methods
----------------

EWP designers provide a couple of standard methods of dealing with these
security aspects. The number of these methods MAY grow in time, some methods
also MAY get deprecated.


### Where are they defined?

Each is documented in its own repository. Look in the Table of Contents on the
[Developers' Page][develhub].


### Which of them am I required to support?

If an API chooses to follow Authentication and Security guidelines described in
this document, then:

 * **Server implementers choose by themselves** which security methods they
   support for this API, or each of its endpoints (some APIs MAY allow to make
   this distinction on a per-endpoint basis). API designers allow server
   implementers to declare their choices via proper elements in their manifest
   entries.

 * **Client implementers choose by themselves** if methods supported by the
   server are "secure enough" for them to connect with this particular API on
   this particular server.

 * EWP designers MAY make certain recommendations, as to which security methods
   should be supported, and which should not, for which APIs. These
   recommendations can be included in both API specifications, and/or
   specifications of each of the security methods. Both server and client
   implementers are RECOMMENDED (but not required) to comply with these
   recommendations.

 * In some rare cases, EWP administrators MAY take action and try to prohibit
   certain security methods to be used within the EWP Network. In particular,
   they are allowed to instruct the Registry Service to dynamically remove API
   entries from the Registry's catalogue (based on security protocols these
   APIs do, or don't, support).

Also, please remember the following:

 * A single endpoint **MAY support multiple methods** of handling any single
   security aspect (for example, a couple of ways of authenticating a client),
   as long as these methods are compatible with each other. The server needs
   to be able to identify which methods the client actually uses in its request
   (and - in case of responses - which method the client wants the *server* to
   use).

 * A single API **MAY be served via multiple separate endpoints**, each of them
   using different security requirements. Clients need to remember that when
   they are browsing through the implemented APIs in the Registry!

 * There are many APIs, for which it doesn't make much sense to authenticate
   the client, nor to encrypt the response. Allowing unencrypted anonymous
   requests may significantly increase performance.

 * Some servers and some clients may be forced to use certain methods of
   authentication or encryption, because of the limitations of their internal
   system architecture. That's why it is usually the best for all implementers
   to support *all* standard methods of authentication and encryption, except
   the ones they cannot (or explicitly choose not to, for their own reasons).


### How do I declare my support for a certain method?

By putting its unique element name, in a proper place declared for it, in the
API's `manifest-entry.xsd` file.

 * You will retrieve the "unique element name" from `security-entries.xsd` file
   attached to the security method document you have chosen.

 * You will find the "proper place declared for it" by reading
   `manifest-entry.xsd` file of the certain API.

   Most APIs will require you to wrap your choices in a `HttpSecurityOptions`
   data type, which is defined in *this* document (the one you are reading
   now), in the [`schema.xsd`](schema.xsd) file.


### Are there and "default" values?

Yes, `HttpSecurityOptions` data type makes use of some defaults (see
[`schema.xsd`](schema.xsd) file for details).

This means, that you don't need to explicitly provide values for all four
security aspects. You need to provide them only when you want to override the
defaults.

Note, that these default values were designed to be "as compatible as they can"
with Version 1 of the *Authentication and Security* document, to make it
possible for some APIs to transition to "version 2 security" without breaking
backward compatibility. For this reason, you should keep in mind that "being a
*default* option" does NOT imply "being a *recommended* one".


### Can I use non-standard security methods?

Servers MAY declare support for non-standard, custom methods (by including
non-standard elements in their `HttpSecurityOptions` sub-elements). Such
methods however are out of scope of EWP specifications, and SHOULD be ignored
by most clients.


<a name="rules"></a>

About introducing new methods
-----------------------------

This section describes requirements for the **writing specifications** of
security methods, for each of the four security aspects covered by this
document. This means that you must read this chapter only if you want to
*design a new method* of handling a certain EWP security aspect.

These requirements apply also to the standard methods mentioned above (the
specifications of all standard methods MUST meet these requirements).


### Client Authentication

Each *client authentication method* SHOULD explicitly answer the following
questions:

 * How the client's request must look like? How can the server detect that
   the client is using this particular method for authentication?
 * How can the server verify which HEIs are covered by the requester?
 * How can the server verify that the request has not been tampered with, nor
   [replayed](https://en.wikipedia.org/wiki/Replay_attack) by a third party?
 * Does it provide non-repudiation? Can a server provide a solid proof later
   on, that a particular request took place, and that it originated from a
   certain client?


### Server Authentication

Each *client authentication method* SHOULD explicitly answer the following
questions:

 * How the client's request must look like? How can the server know, that the
   client *wants the server* to use this particular method of authentication?
 * How can the client verify the server's identity?
 * How can the client verify that the response has not been tampered with? Can
   it also verify that it was indeed generated for this particular request?
 * Does it provide non-repudiation? Can a client provide a solid proof later
   on, that the server sent a particular response (in response to a particular
   client's request)?


### Request Encryption

Each *request encryption method* SHOULD explicitly answer the following
questions:

 * How the client's request must look like? How can the server detect that the
   client is using this particular method of encryption?
 * How the server publishes his encryption key? How the client retrieves it
   securely?
 * How to encrypt and decrypt the request? Which parts are covered by the
   encryption and which are not?


### Response Encryption

Each *response encryption method* SHOULD explicitly answer the following
questions:

 * How the client's request must look like? How can the server know, that the
   client *wants the server* to use this particular method of encryption?
 * How the client delivers his encryption key to the server?
 * How to encrypt and decrypt the response? Which parts are covered by the
   encryption and which are not?


<a name="error-signing"></a>

Authentication and Encryption of Error Responses
------------------------------------------------

*Server Authentication* and *Response Encryption* methods describe how *all*
HTTP responses should be signed and encrypted. This includes error responses
(responses with HTTP 4xx and HTTP 5xx status codes).

Why it might be important to **encrypt** error responses?

 * Non-encrypted responses can be overheard. If the error response may contain
   private data, then it is important that only the requester can read this
   data.

 * Note, that using TLS is usually enough to protect response confidentiality.

Why it might be important to **sign** error responses?

 * So that the requester will be able to prove, that the server responded with
   an error for his particular request. This non-repudiation might be relevant
   in some cases.

For these reasons:

 * If you are a server implementer and you use particular security method for
   signing and encryption your responses, then it is RECOMMENDED that you use
   exactly the same method when you are responding with errors. If you are
   unable to do so, then you MAY fall back to sending your error response over
   regular TLS. You MUST NOT however send your error responses over completely
   unprotected channel.

 * The clients SHOULD be prepared for both cases. The servers MAY send their
   error response bodies either encrypted or not encrypted; they MAY contain
   a HTTP signature, or they may not. However, all such responses will be sent
   over TLS (HTTPS).


Examples
--------

In all following examples:

 * The `sec` prefix is bound to the
   `https://github.com/erasmus-without-paper/ewp-specs-sec-intro/tree/stable-v2`
   namespace, as declared in [`schema.xsd`](schema.xsd) file.

 * The `<http-security>` element is in an unknown namespace (because it is
   irrelevant in this context), but its data type is `sec:HttpSecurityOptions`.


### Example 1

```xml
<http-security>
    <sec:client-auth-methods>
        <tlscert
            xmlns="https://github.com/erasmus-without-paper/ewp-specs-sec-cliauth-tlscert/tree/stable-v1"
            allows-self-signed="true"
        />
    </sec:client-auth-methods>
    <sec:server-auth-methods>
        <tlscert xmlns="https://github.com/erasmus-without-paper/ewp-specs-sec-srvauth-tlscert/tree/stable-v1"/>
    </sec:server-auth-methods>
</http-security>
```

In this example, the implementer declares that he is still following the
default set of security options, compatible with *Version 1* of the
*Authentication and Security* document. The XML above is also equivalent to
both of the following examples (all three of these XMLs have **exactly** the
same meaning):

```xml
<http-security>
    <!-- Empty -->
</http-security>
```

```xml
<!-- These are the defaults -->
<http-security>
    <sec:client-auth-methods>
        <tlscert
            xmlns="https://github.com/erasmus-without-paper/ewp-specs-sec-cliauth-tlscert/tree/stable-v1"
            allows-self-signed="true"
        />
    </sec:client-auth-methods>
    <sec:server-auth-methods>
        <tlscert xmlns="https://github.com/erasmus-without-paper/ewp-specs-sec-srvauth-tlscert/tree/stable-v1"/>
    </sec:server-auth-methods>
    <sec:request-encryption-methods>
        <tls xmlns="https://github.com/erasmus-without-paper/ewp-specs-sec-reqencr-tls/tree/stable-v1"/>
    </sec:request-encryption-methods>
    <sec:response-encryption-methods>
        <tls xmlns="https://github.com/erasmus-without-paper/ewp-specs-sec-resencr-tls/tree/stable-v1"/>
    </sec:response-encryption-methods>
</http-security>
```


### Example 2

```xml
<http-security>
    <sec:client-auth-methods>
        <anonymous xmlns="https://github.com/erasmus-without-paper/ewp-specs-sec-cliauth-none/tree/stable-v1"/>
        <tlscert
            xmlns="https://github.com/erasmus-without-paper/ewp-specs-sec-cliauth-tlscert/tree/stable-v1"
            allows-self-signed="false"
        />
    </sec:client-auth-methods>
</http-security>
```

In this example, the server implementer states that:

 * It allows its endpoint(s) to be accessed by [anonymous
   clients][cliauth-none].

 * It also allows its endpoint(s) to be accessed by clients who use [TLS Client
   Certificate Authentication][cliauth-tlscert], but the certificate they use
   MUST NOT be self-signed.

BTW: This is a bit "off topic", but you might wonder why this server doesn't
simply require *all* requests to be anonymous - what's the reason for allowing
the client to authenticate itself, if the resource can also be accessed by
anonymous clients? However, without proper context, we cannot reliably answer
that. Perhaps there actually *is* a reason for supporting both, for example,
the server is returning a broader set of results for authenticated clients
(while anonymous clients get only a small subset). All of this depends on the
particular API with which this example `<http-security>` element is used along
with.


### Example 3

```xml
<http-security>
    <sec:client-auth-methods>
        <tlscert
            xmlns="https://github.com/erasmus-without-paper/ewp-specs-sec-cliauth-tlscert/tree/stable-v1"
            allows-self-signed="true"
        />
        <httpsig xmlns="https://github.com/erasmus-without-paper/ewp-specs-sec-cliauth-httpsig/tree/stable-v1"/>
    </sec:client-auth-methods>
    <sec:server-auth-methods>
        <tlscert xmlns="https://github.com/erasmus-without-paper/ewp-specs-sec-srvauth-tlscert/tree/stable-v1"/>
        <httpsig xmlns="https://github.com/erasmus-without-paper/ewp-specs-sec-srvauth-httpsig/tree/stable-v1"/>
    </sec:server-auth-methods>
</http-security>
```

In this example, the server states that:

 * It supports client authentication via both [TLS Certificate][cliauth-tlscert]
   and [HTTP Signature][cliauth-httpsig]. This means that the client can freely
   choose which one of those it wants to use.

 * Similarly, it supports server authentication with two methods. This means
   that the client MAY ask the server to sign its response with an additional
   [HTTP Signature][srvauth-httpsig], and the server will honor such request.


### More examples coming soon

More examples will be included here, as more security methods are released as
stable.


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
[sec-v1]: https://github.com/erasmus-without-paper/ewp-specs-sec-intro/tree/stable-v1
[sec-v2]: https://github.com/erasmus-without-paper/ewp-specs-sec-intro/tree/stable-v2
