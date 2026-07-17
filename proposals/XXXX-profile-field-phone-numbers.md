# MSCXXXX: Profile field for user phone numbers

Knowing another user's phone number is useful to reach them via an alternate means of contact,
such as low power/bandwidth air gapped systems that may not be replaced with existing or currently proposed Matrix-based
real time communication solutions.

Example uses include:

* Internal direct dialing in local phone networks (office, hacker congress).
* Making your "normal phone number(s)" easily accessible to a circle you connect with through Matrix.

## Proposal

Profiles can provide an optional `m.phone_numbers` property,
containing a list of objects that associate an optional description with a number*.
Clients can set and fetch this via the [normal API endpoints](https://spec.matrix.org/v1.14/client-server-api/#profiles).

* Servers MAY validate that each phone number uses only ASCII characters.
* Servers MAY decide to ignore data in either number or label property that exceeds 100 characters.
* Since both label and number field allow arbitrary strings,
  clients supporting this feature MUST handle contents with the usual care towards untrusted input.
* Clients MAY decide to only display numbers consisting of ASCII characters.
* Clients MAY decide to ignore data in either number or label property that exceeds 100 characters.

The rationale for somewhat loose validation is that phone number formatting conventions vary widely internationally,
including spaces, braces, dashes, etc.
This proposal specifically also allows using letters as phone numbers,
e.g. as per [ITU REC E.161](https://www.itu.int/rec/T-REC-E.161).
Clients might choose to offer a translation to numbers according to E.161 in their UI.

If the field is not provided, it SHOULD be interpreted as having no phone number information for that user.

An example request to set the time zone would be:

```
PUT /_matrix/client/v3/profile/@alice:example.org/m.phone_numbers

{
  "m.phone_numbers": [
    { "number": "6879",
      "label": "40C3 Matrix Assembly" },
    { "number": "MTRX",
      "label": "40C3 Matrix Assembly" }
  ]
}
```

Similarly when retrieving a user's profile:

```
GET /_matrix/client/v3/profile/@alice:spezi.expert

{
  "displayname": "Alice",
  "m.phone_numbers": [
    { "number": "5707",
      "label": "39C3" }
  ]
}
```

## Potential issues

Clients may wish to periodically fetch the phone numbers of other users as they may change over time.
Currently, profile data isn't propagated/synchronized between servers,
but that's left to a future MSC to solve.
It is recommended that clients cache the value for 12 - 24 hours.
Clients might also offer UI for users to trigger a refresh manually.

There should not be backwards compatibility concerns since clients should be ignoring
unknown profile fields.

## Alternatives

The field could make tighter prescriptions to allowed formats, characters, or length.
This could for example follow [ITU REC E.123](https://www.itu.int/rec/T-REC-E.123) and
[ITU REC E.164](https://www.itu.int/rec/T-REC-E.164), however this would not cover
the length of direct dialling extensions.

### Delegate profile fields

There are several standards related to storing of contact information electronically,
notably vCard and its derivatives (see below). It is unclear if Matrix profile
information is similar enough to the contact information found in vCard to warrant using
that format directly, although there is certainly some overlap.

Some of the JSON formats for vCard which include time zone information are detailed below:

[RFC7095: jCard The JSON Format for vCard](https://datatracker.ietf.org/doc/html/rfc7095)
format could be used instead, but this doesn't make much sense unless the entire
profile was replaced.

[RFC9553](https://datatracker.ietf.org/doc/html/rfc9553) offers an alternative
representation for contacts (which is not backwards compatible with vCard). There
exists `timeZone` field under the `addresses` field which uses an time zone name
from the IANA Time Zone Database.

Note there's an alternative [jCard](https://microformats.org/wiki/jCard) format
which is a non-standard derivative of [hCard](https://microformats.org/wiki/hcard).

## Security considerations

Showing a user's phone number gives some information on how to reach them out of band.
This extends a user's attack surface also out of band, based on their profile's visibility.
There is currently no way to limit what profile fields other users can see.

Clients may wish to warn users when providing a phone number and give
the option to not include it in their profile.

## Unstable prefix

`ac.ccc.phone_numbers` should be used in place of `m.phone_numbers`.

Clients may immediately use the stable profile field once this MSC is accepted. This is
a client-to-client protocol and no feature negotiation is necessary.

## Dependencies

None.
