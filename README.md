# Mail headers and addresses

`ecosystem::mail` parses bounded mail address lists and header blocks in pure
GoML. It builds on `ecosystem::textproto` for CRLF and folded-header framing and
on `ecosystem::mime` for RFC 2047 encoded words. It does not send mail, implement
SMTP, decode message bodies, or claim full support for obsolete RFC 5322 syntax.

`parse_address` accepts a single bare or angle-bracket mailbox.
`parse_address_list` accepts comma-separated mailboxes and groups, flattening
group members into the returned address list. A completed group requires a comma
before another address or group; trailing commas, including those followed by
whitespace or comments, are rejected. Comments may follow the final member or
an empty group. Group labels use the same display-name validation as mailboxes; an empty quoted
name is valid, while an absent or comment-only label is not. `Address` exposes the decoded
display `name`, original validated `address`, `local` and `domain`. The local
part supports dot atoms and quoted strings; the domain supports DNS-style ASCII
labels and bracketed literals. Display names may be UTF-8, quoted strings or
MIME encoded words. Comments and horizontal whitespace are supported around
mailboxes; comments inside an addr-spec and obsolete route syntax are rejected.
Internationalized local parts and domains need an explicit application policy
and are outside this module.

`parse_headers` accepts a byte buffer containing a CRLF-terminated header block
and optional body. Its `Headers` result preserves duplicate fields in order,
offers `values(name)`, `decoded(name, limits)` and `addresses(name, limits)`, and
reports `body_offset()` into the supplied buffer. Header values must be UTF-8;
binary bodies remain untouched. MIME encoded-word decoding is explicit through
`decoded` or `decode_header_value`.

`Limits::standard()` caps total input at 1 MiB, an address at 4 KiB, addresses
at 128, comment depth at 8, a physical header line at 998 bytes, the header
block at 64 KiB, fields at 128, unfolded values at 16 KiB and decoded values
at 64 KiB. All limits are configurable nonnegative values. `Error` provides a
kind, byte offset and diagnostic message. Parsing does not resolve domains,
interpret `Received`/`Date`, or validate delivery policy.

The address grammar is a bounded modern subset of [RFC 5322](https://www.rfc-editor.org/rfc/rfc5322.html),
with display-name decoding from [RFC 2047](https://www.rfc-editor.org/rfc/rfc2047.html).
Run `(cd ../verification && just ecosystem-test mail)` from this library repository to verify the library
and its example, including independent downstream verification.

## Development and examples

Requires GoML 0.1.56 or newer. The `examples/basic/` example shares the root manifest and its dependencies. From the library root, run:

```sh
goml run --example basic
goml test
goml verify --timeout 300s
```

`goml test` builds the example and runs its tests. `goml verify` repeats the example checks as an independent module against an isolated registry snapshot. `(cd ../verification && just ecosystem-test mail)` also retains the library-specific smoke and compatibility checks.
