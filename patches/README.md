# Patches

Applied to the cloned IronRDP checkout by the release workflow before
`wasm-pack build`.

## 0001-expose-autologon-via-extension.patch

Exposes the Client Info PDU autologon flag to JS via the existing
`Extension` mechanism (`new Extension('autologon', true)`), instead of the
hardcoded `autologon: false`. uniTerm sets it when a saved password is
present, mirroring mstsc's saved-credentials behavior. Regenerate against a
fresh IronRDP checkout if `git apply` fails after an upstream change.
