# Patches

Applied to the cloned IronRDP checkout by the release workflow before
`wasm-pack build`.

## 0001-expose-autologon-via-extension.patch

Exposes the Client Info PDU autologon flag to JS via the existing
`Extension` mechanism (`new Extension('autologon', true)`), instead of the
hardcoded `autologon: false`. uniTerm sets it when a saved password is
present, mirroring mstsc's saved-credentials behavior. Regenerate against a
fresh IronRDP checkout if `git apply` fails after an upstream change.

## 0002-cliprdr-ready-on-rejected-initial-format-list.patch

`FormatListResponse::Fail` never completed the cliprdr initialization
handshake, so servers that reject the initial (empty) format list — 2008 R2
does — left the channel stuck in `Initialization` and every later
`initiate_copy` failed with "clipboard channel is not in Ready state". The
patch treats any format-list response as completing the handshake.
