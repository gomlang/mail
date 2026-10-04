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
name is valid, while an absent or comment-only label is not. Quoted group names
may contain `@` and angle brackets, just like quoted mailbox display names;
an unquoted `@` remains invalid display-name punctuation. `Address` exposes the decoded
display `name`, original validated `address`, `local` and `domain`. The local
part supports dot atoms and quoted strings, including space and horizontal tab
inside the quotes or after a quoted-pair backslash; the validated wire spelling
is retained. The domain supports DNS-style ASCII
labels and bracketed literals. Domain literals preserve RFC 5322 `dtext`
punctuation, including commas, semicolons, colons, parentheses, quotes, angle
brackets and `@`, without interpreting it as list, comment or mailbox syntax.
Brackets, backslashes, controls, whitespace and non-ASCII literal contents remain
unsupported; the library does not interpret a literal as an IP address. Display names may be UTF-8, quoted strings or
MIME encoded words. Literal square brackets in display names must be quoted or
encoded; unquoted brackets cannot hide display-name or comment syntax. A literal
backslash in a display or group name must likewise be quoted and escaped, or
MIME encoded; a bare backslash is not display-name atom syntax. Comments and horizontal whitespace are supported around
mailboxes; comments inside an addr-spec and obsolete route syntax are rejected.
Internationalized local parts and domains need an explicit application policy
and are outside this module.

`parse_address_items(value, limits)` preserves address-list structure as ordered
`AddressItem::Mailbox(Address)` and `AddressItem::Group(name, members)` entries.
Group names are decoded using the same quoted-string/MIME rules as display names;
empty groups and explicitly empty quoted names are retained. Group members remain
ordered, and nesting remains invalid. The shared parser retains all syntax and
address limits of `parse_address_list`. For this structured API, `max_addresses`
also independently caps the number of groups, including empty groups. The older
flattening API keeps its existing empty-group and address-count behavior. This
model preserves semantic structure, not comments, whitespace or original quoting.

`parse_headers` accepts a byte buffer containing a CRLF-terminated header block
and optional body. Its `Headers` result preserves duplicate fields in order,
offers `values(name)`, `decoded(name, limits)` and `addresses(name, limits)`, and
reports `body_offset()` into the supplied buffer. Header values must be UTF-8;
binary bodies remain untouched. MIME encoded-word decoding is explicit through
`decoded` or `decode_header_value`.
Unfolding removes only the CRLF before a continuation, preserving spaces and
tabs inside the resulting value, including quoted parameter values and local
parts. Leading and trailing spaces/tabs are trimmed once around the complete
unfolded field; its remaining bytes count toward `max_value_bytes`. Adjacent
MIME encoded words still discard intervening whitespace when explicitly decoded.
The unfolded size is checked before copying the value. Intermediate framing
storage is bounded by `max_header_bytes`, rather than the per-value limit.

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
