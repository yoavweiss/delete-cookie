---
title: "Delete-Cookie"
abbrev: "DelCookie"
category: info

docname: draft-deletecookie-weiss-http-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "yoavweiss/delete-cookie"
  latest: "https://yoavweiss.github.io/delete-cookie/draft-deletecookie-weiss-http.html"

author:
 -
    fullname: Yoav Weiss
    organization: Shopify
    email: yoav@yoav.ws
normative:
  STRUCTURED-FIELDS:
    title: Structured Field Values for HTTP
    date: 1 May 2024
    target: https://datatracker.ietf.org/doc/draft-ietf-httpbis-sfbis/

  COOKIES:
    title: Cookies HTTP State Management Mechanism
    date: 20 February 2025
    target: https://datatracker.ietf.org/doc/draft-ietf-httpbis-rfc6265bis/

informative:


--- abstract

This document specifies a `Delete-Cookie` HTTP header that instructs clients to delete cookies of certain names,
without requiring the server to know more details about them.

--- middle

# Introduction

Long-operating web sites can often find themselves dealing with "cookie cruft" -
cookies that no longer have backend logic that corresponds with them.

Such cookies may have been set at some point in the past with
far-reaching expiration dates, and are now causing useless cookie bloat at
best, or using up quotas at the expense of relevant cookies at worst

Deleting cookies is possible today by setting their expiry date to one in the past,
but that requires one to know the "domain" and "path" parameters with which the cookies were set.
That is not something that can be passively observed on the server side by default.

This draft proposes a new `Delete-Cookie` header which will enable servers to instruct clients
to delete cookies of a certain name from their cookie stores.


## Notational Conventions

{::boilerplate bcp14-tagged}

This document uses the following terminology from Section 3 of {{STRUCTURED-FIELDS}} to specify syntax and parsing: Lists, strings.

# Delete-Cookie

The Delete-Cookie response header is a Structured Field {{STRUCTURED-FIELDS}} List of strings.
Each string represents a cookie {{COOKIES}} name to be deleted.

A user agent that receives a Delete-Cookie response header MUST process it before any Set-Cookie headers received as part of the same response.

When receiving a Delete-Cookie response header, for each string `name` in the List, the user agent MUST do the following:

1) For each cookie in the user agent's cookie store whose name is `name`, and domain domain-matches {{COOKIES}} the canonicalized request-host {{COOKIES}}:
   1) If the cookie's host-only-flag is true and the cookie's domain is not the canoicalized request-host, continue.
   1) Delete cookie from the user agent's cookie store.

## Example

MegaCorp servers receive a request from a client with the following header:

~~~ http-message
Cookie: foo=bar, fizz=baz
~~~

The server operators know that cookies named "foo" and "fizz" are no longer meaningful for their service, and are unaware when and how those cookies were set.
In order to clear that user's cookie store, they send the following header:

~~~ http-message
Delete-Cookie: "foo", "fizz"
~~~

That clears the client's cookie store of "foo" and "fizz" and ensures that these cookies won't be sent again.

# Security Considerations

The Delete-Cookie header enables servers to delete cookies from user agents on their own registrable domains.
These servers could have already deleted these same cookies by setting cookies with identical name, path and domain with an expiration date of 0.
As such the header does not provide servers any new capabilities, beyond the convenience of not having to know the path and domain of a cookie in order to delete it.

# IANA Considerations

## Header Field Registration

IANA is asked to update the
"Hypertext Transfer Protocol (HTTP) Field Name Registry" registry maintained at
<[](https://www.iana.org/assignments/http-fields/http-fields.xhtml)> according
to the table below:

|----------------------|-----------|-------------------------------------------|
| Field Name           | Status    |                 Reference                 |
|----------------------|-----------|-------------------------------------------|
| Delete-Cookie        | permanent | {{delete-cookie}} of this document        |
|----------------------|-----------|-------------------------------------------|


--- back

# Acknowledgments
{:numbered="false"}

Thanks to Anne van Kesteren and Pat Meenan for their feedback on early versions of this proposal.
