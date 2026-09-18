# Variables worth injecting

Choose only variables that plausibly affect the changed behavior. Turn each into
a ticket-specific hypothesis:

`If we inject X, Y still holds.`

Prefer 3–8 high-risk experiments. Large-input and concurrency probes belong here
only when testing correctness, authorization, availability, or contract
behavior, not throughput optimization.

| Variable | Example falsifiers |
| --- | --- |
| Boundary and cardinality | Empty collection, singleton, first/last item, zero, negative value, exact minimum/maximum, open-vs-closed range, off-by-one index |
| Token and path boundaries | Prefix or substring accepted instead of a whole token or segment; trailing separator; mixed separators; case-sensitive mismatch |
| Identifier and namespace | Correctly shaped ID from the wrong resource type, parent, tenant, account, region, or source |
| Absence states | Missing, null, empty, unknown, denied, and redacted collapse to the same behavior |
| Unicode and normalization | NFC/NFD difference, case-folding, grapheme cluster, confusable character, non-breaking or unusual whitespace |
| Ambiguous resolution | Zero, one, several, or tied candidates silently produce a guessed result |
| Parsing and extraction | Ordinary prose, quoted text, adjacent clauses, or partial payload becomes unintended structured input |
| Ordering and duplication | Reordered items, duplicate entries, duplicate events, repeated keys, or unstable iteration changes the result |
| Time and locale | UTC/local mismatch, DST gap or fold, leap day, expiry boundary, clock skew, locale-specific decimal/date/casing |
| Permission and ownership | Wrong tenant, inherited access, stale credential, downgraded role, or inaccessible parent leaks or hides data |
| Retry and idempotency | Timeout after side effect, duplicate delivery, partial retry, repeated request, or replay applies work twice |
| Concurrency and interruption | Interleaving, stale read, lost update, cancellation, shutdown, or resumed work violates the invariant |
| Cache correctness | Partial, truncated, negative, unauthorized, or stale result is cached; key omits identity, scope, version, locale, or permissions |
| Serialization and schema | Missing/extra field, old/new schema, nullability change, reordered payload, or failed round-trip changes behavior |
| Error handling | Retryable and terminal errors collapse; partial success is treated as empty success; fallback silently hides failure |
| Ignored returned update | Immutable merge, returned copy, count, status, or replacement object is not assigned or published |
| Large or capped input correctness | Truncation, overflow, pagination boundary, hop cap, recursion cap, or batch split loses or misclassifies data |
| Stale path or wiring | Deleted branch remains reachable; obsolete module is imported; stub or fallback is still selected |
| Routing and capability | Request is sent to the wrong handler, backend, transport, parser, index, or tool |
