# Patches

Applied to the cloned IronRDP checkout by the release workflow before
`wasm-pack build`.

## 0001-expose-autologon-via-extension.patch

Exposes the Client Info PDU autologon flag to JS via the existing
`Extension` mechanism (`new Extension('autologon', true)`), instead of the
hardcoded `autologon: false`. uniTerm sets it when a saved password is
present, mirroring mstsc's saved-credentials behavior. Regenerate against a
fresh IronRDP checkout if `git apply` fails after an upstream change.

## 0003-expose-connection-type-and-performance-flags.patch

Exposes two more `Extension` idents so the embedder can pick the network
profile instead of the hardcoded `ConnectionType::Lan` +
`PerformanceFlags::default()`:

- `connection_type` (number, 1-7): the MS-RDPBCGR `TS_CONNECTION_TYPE` value
  (1 = modem, 2 = broadband-low, 4 = broadband-high, 6 = LAN). The server
  uses it to pick the desktop experience tier. Honored because the connector
  already sets the `VALID_CONNECTION_TYPE` early capability flag.
- `performance_flags` (number): raw `TS_PERF_*` bitmask; unknown bits are
  ignored. Disabling wallpaper/font smoothing/desktop composition etc.
  cuts server-side graphics traffic on low-bandwidth links.

Both are optional: when absent the previous hardcoded behavior is kept.
Regenerate against a fresh IronRDP checkout if `git apply` fails after an
upstream change.

## 0004-tolerate-pdus-during-finalization.patch

During connection finalization the client sends the Font List PDU and waits
for the Font Map PDU. MS-RDPBCGR already lets the server start sending share
traffic (graphics updates, pointer shapes, session info, ...) once Confirm
Active has been processed, so those PDUs legitimately race with the Font Map.
Upstream treated any non-finalization PDU in that window as a hard error
(`unexpected server message`), killing connections intermittently depending
on server/network timing. The patch skips the benign PDUs (SaveSessionInfo,
FrameAcknowledge, SuppressOutput, RefreshRectangle, Update, Pointer,
PlaySound, SetKeyboardIndicators, SetKeyboardImeStatus, StatusInfoPdu) and
keeps waiting; genuinely unexpected PDUs now fail with the PDU short name in
the error message instead of a bare string.
