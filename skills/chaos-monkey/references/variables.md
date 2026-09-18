# Variables worth injecting

Turn each into a ticket-specific hypothesis: `If we inject X, Y still holds.` Skip any row that cannot apply to this diff.

| Variable | What would disprove steady state |
| --- | --- |
| Sibling prefix | `agent-test` matches `agent-test-old`; `Finance/Invoices` matches `Finance/Invoices-old` via raw `in`, `CONTAINS`, or character prefix of a longer segment |
| Wrong identity field | Item id used as drive id; remote folder metadata instead of the local parent; a null display column while path segments exist; a document path used as source scope |
| Empty vs missing vs denied | Empty tuple blocking a shared corpus; `None` vs `[]`; missing path treated as "no access" or as success |
| Dead path still live | Short-circuit left in production; stub wired to look complete; deleted module still imported |
| Ambiguity still proceeds | Two candidates, a guessed ceiling, no clarification |
| Over-extraction | Regex lifts `last quarter` / `the report` as a folder name and yields not-found instead of unscoped |
| Cache poisoning | Truncated hop-cap/cycle walks cached; concurrent walks defeating a hop cache; token fingerprint missing from the key |
| Silent swallow | Retryable errors indexed as empty; overlay/merge result discarded so callers keep the old object |
| Wrong bus | Local tool counted as remote/MCP; document search used as folder lookup |
| Contract fixture drift | New required API/SSE field; recorders and golden payloads omitted; CI fails on serialization, not on the unit you wrote |
| Counts / overlay discarded | Live status hardcoded to 0; merge returns a new object nobody assigns |
