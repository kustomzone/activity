# vocab

The `vocab` package provides static types for Core and Extended types to the
[ActivityStream Vocabulary](https://www.w3.org/TR/activitystreams-vocabulary).
The library is battle-tested against all 159 examples in the Vocabulary
specification linked above in addition to unit tests.

Its mission is simple: Provide meaningful static types for the ActivityStream
Vocabulary in golang.

This library is entirely code-generated by the `tools/vocab/gen` library
and `tools/vocab` tool. Run `go generate` to refresh the library, which
which requires `$GOPATH/bin` to be on your `$PATH`.

The [go-fed.org](https://go-fed.org) website has a tutorial. It also hosts godoc
documentation for every version of this library.

## This library's API is huge!

**The W3C does not require client applications to support all of these
use cases.** The W3C only requires that *"all implementations must at least be
capable of serializing and deserializing the Extended properties in accordance
with the Activity Streams 2.0 Core Syntax,"* which what this library and the
`activity/streams` libraries do for clients. This library's API is large to 
permit clients to use as much or as little as desired.

## What it does

This library is given a `map[string]interface{}`, presumably from an
ActivityStream or JSON-LD kind of JSON, and returns a static type that provides
a statically-typed API.

For example, consider an application that receives the simple ActivityStream
Vocabulary object in the following JSON:

```golang
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Note",
  "name": "Automated Train",
  "content": "Now arriving at sector C test labs and control facilities.",
  "published": "1998-11-08T08:47Z",
  "actor": "http://freeman.example.org"
}
```

In order to extract this information, the application can use `encoding/json`
to obtain a `map[string]interface{}` of this data. It cannot use a raw direct
type because JSON-LD, and therefore the ActivityStream Vocabulary, permit the
same property to be any of a JSON string, a JSON object, or a JSON array in most
conditions. Consider:

* The `type`, `name`, `content`, and `actor` properties could be a single value
  or a JSON array, resulting in JSON deserializing to an `interface{}` or
  `[]interface{}` for these properties
* Any of these properties could be a string value or need to be treated as an
  [IRI](https://www.ietf.org/rfc/rfc3987.txt)
* The `published` time conforms to RFC3339 with the exception that seconds may
  be omitted

Therefore, trying to statically determine this with typical JSON tagging does
not work:

```golang
type NaiveActivity struct {
  type string `json:"Type"`   // Not OK, cannot handle array of strings
  name []string `json:"Name"` // Not OK, cannot handle single values
  // ...
  published time.Time `...`   // Not OK, cannot handle when seconds are omitted
}
```

This is the motivation for this library.

All of these considerations are presented as:

```golang
type Note struct { ... }
func (n *Note) NameLen() int { ... }
func (n *Note) IsNameString(index int) bool { ... }
func (n *Note) IsNameIRI(index int) bool { ... }
func (n *Note) GetNameString(index int) string { ... }
// And so on
```

Note that the resulting API and property type possibilities is *large*. This is
a natural consequence of the specification being built on top of JSON-LD.

## What it doesn't do

This library does not use the `reflect` package at all. It prioritizes
minimizing dependencies and speed over binary size.

The ActivityStream specification is built on top of JSON-LD, which uses JSON.
This library should be used with `encoding/json` in order to transform a raw
string to a `map[string]interface{}` and then to these static types.

This library does not set the `"@context"` property required when sending
serialized data. Clients are in charge of setting it to
`"https://www.w3.org/ns/activitystreams"`.

This implementation is heavily opinionated against understanding JSON-LD due to
its sacrifice of semantic meaning, significant increase of complexity, even
weaker typing, and increased exposure to partially-understood messages. These
costs earn a degree of flexibility that is not needed for the ActivityStream
Vocabulary.

This library is *not* a [JSON-LD](https://json-ld.org/) parser, and by design
does not implement any further understanding of JSON-LD that may be outlined in
the [W3C's JSON-LD](https://www.w3.org/TR/json-ld/) specification. Furthermore,
it does *not* implement any of the JSON-LD
[processing algorithms](https://www.w3.org/TR/json-ld-api/). If this
functionality is strictly needed, or this library is not suitable, please see 
[piprate/json-gold/ld](https://github.com/piprate/json-gold) and its
[documentation](https://godoc.org/github.com/piprate/json-gold/ld).

## Other considerations

This library is entirely code-generated. Determined clients can add their own
custom extended types to the `tools/defs` library and generate a useful type.
However, this process is purposefully painful to force clients to seriously
consider whether they need their own [custom type](https://xkcd.com/927).

The code-generation aspect also allows the specification to be translated into
declarative data, which permits certain kinds of validation and verification.
This has resulted in feedback to the specification and the W3C working group.

## Thanks

Many thanks to those who have worked on JSON-LD, ActivityStreams Core, and the
ActivityStreams Vocabulary specifications.
