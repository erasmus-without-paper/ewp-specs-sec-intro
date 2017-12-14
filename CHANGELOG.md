Release notes
=============

This document describes all the changes made to the *Authentication and
Security* document, starting from its first released version.


2.0.2
-----

* Added `processContents="lax"` to all `<xs:any>` elements. This is needed in
  order for parsers to accept custom security methods.


2.0.1
-----

* Added a more complex example of `sec:HttpSecurityOptions` usage, with
  multiple authentication methods.


2.0.0
-----

New major version of the document. Many sections were entirely rewritten, so
implementers are REQUIRED to re-read the document in its entirety. See [this
thread](https://github.com/erasmus-without-paper/ewp-specs-architecture/issues/17)
and [this thread](https://github.com/erasmus-without-paper/ewp-specs-sec-intro/issues/1)
for reasoning on most of these changes.

Summary of major changes:

 * Server implementers now decide by themselves (on a per-host and per-API
   basis) which authentication and security methods they support (or require).
   They are no longer required to support a predetermined set of security
   solutions.

 * Clients can no longer assume that the server supports a predetermined set of
   security methods. Before making the request, clients MUST verify which
   security protocols are supported by the server, and construct their request
   appropriately.

 * Each API explicitly declares if it follows the requirements of this
   document, or not. This means that some "live" APIs may still follow the
   requirements of the *Version 1* of this document, while others upgrade their
   requirements to *Version 2*.

 * If an API follows the requirements of *Version 2* security, then its
   designers are RECOMMENDED to include a "standard" element of the
   `sec:HttpSecurityOptions` data type (defined in this document) in their API
   entries (their `manifest-entry.xsd` file). Clients and servers use this
   element to communicate which authentication and encryption methods the
   server supports (the document contains examples of its use).


1.1.0
-----

* New section added (named *"Scope of this document"*). It describes a new
  requirement: Each API that wants to require its implementers to follow the
  requirements of *Authentication and Security* document, MUST state this
  fact explicitly in its own specification.

* Clarified that - if an API wants to enable server implementers to allow
  anonymous client authentication - then it SHOULD state that explicitly in
  its own specification, and ot SHOULD introduce a new element in their
  `manifest-entry.xsd` (which will enable clients to detect that anonymous
  requests are supported).

* Fix spelling mistakes and style.


1.0.0
-----

This document was created as a result of splitting the Architecture and
Security document into a number of separate smaller documents. This document
was already stable (version 1.7.0) before it has been split, and the "meaning"
of this "child" document has not been modified much, so we made it stable
(1.0.0) immediately.

It's worth noting however, that - while the *meaning* remained the same - the
*wording* changed substantially, to accommodate the terms introduced by the
new multiple authentication and encryption methods.
