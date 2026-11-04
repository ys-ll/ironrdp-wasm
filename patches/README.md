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

## 0003-standard-rdp-security.patch

Implements the legacy standard RDP security layer (PROTOCOL_RDP): full RC4
key derivation and per-packet MAC/encryption on the slow and fast paths, the
Security Exchange PDU, and GCC negotiation of the 40/56/128-bit encryption
methods. uniTerm advertises SSL/HYBRID as before but additionally accepts a
server that selects PROTOCOL_RDP (xrdp configured with `security_layer=rdp`,
servers with broken TLS, old appliances), instead of failing the negotiation
with "server selected STANDARD_RDP_SECURITY". FIPS (3DES) is rejected.

This is a substantial patch (connector, session, pdu, web crates) — keep it
rebased on the pinned `IRONRDP_SHA` in the release workflow.
