# ironrdp-wasm

Build harness that compiles [Devolutions/IronRDP](https://github.com/Devolutions/IronRDP)'s
`ironrdp-web` crate (a pure-Rust RDP client compiled to WebAssembly) into an
npm package and publishes it as a GitHub Release asset.

This repo contains no Rust source of its own. The workflow clones IronRDP at
build time, runs `wasm-pack build` on `crates/ironrdp-web`, rebrands the
generated `pkg/` as `@ys-ll/ironrdp-wasm`, and attaches the npm tarball to the
release. Each release notes record the exact IronRDP commit it was built from.

## Why a plain build (no custom wrapper)

uniTerm consumes the crate's own wasm-bindgen API (`SessionBuilder`,
`Session`, `InputTransaction`, `DeviceEvent`, `ClipboardData`, `DesktopSize`).
The crate speaks the RDCleanPath handshake over its WebSocket transport; the
RDP-side of that protocol is implemented in uniTerm's local proxy
(`backend/session/ironrdp_proxy.go`), so the WASM build stays stock and can be
rebuilt from upstream at any time without carrying a patch.

## Release

```
git tag v0.1.0
git push origin v0.1.0
```

The workflow builds and uploads `ys-ll-ironrdp-wasm-<version>.tgz`. uniTerm
installs it as an optional URL dependency:

```json
"optionalDependencies": {
  "@ys-ll/ironrdp-wasm": "https://github.com/ys-ll/ironrdp-wasm/releases/download/v0.1.0/ys-ll-ironrdp-wasm-0.1.0.tgz"
}
```

Until the release exists, uniTerm's vite config aliases the package to a stub
so builds stay green (the IronRDP tab surfaces an explanatory error).

## Rebuilding after an upstream IronRDP update

Just cut a new tag — the workflow always builds from IronRDP master HEAD and
records the commit in the release notes. Bump the tag (and the URL in
uniTerm's `package.json`) to pick the new build up.
