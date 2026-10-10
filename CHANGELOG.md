# Changelog

Notable changes, newest first. Releases are git tags; this file says what each tag contains.

## v0.1.1

A security release. **The service now builds with Go 1.27.2**, and its libraries move to their Go 1.27.2
releases. Nothing about the API, the configuration or the database changes.

### Changed

- **Go 1.27.2** (was 1.27.0), both in `go.mod` and as the image's default build toolchain
  (`ARG GO_VERSION` in the `Dockerfile`). Go 1.27.2 fixes standard-library and `golang.org/x/net`
  vulnerabilities that this service's code reaches under 1.27.0: `govulncheck` found nine it calls,
  GO-2026-6603, -6605, -6607, -6608, -6610, -6611, -6612, -6613 and -6617, in `net/http`,
  `crypto/tls`, `net/textproto`, `mime/multipart` and `golang.org/x/net`. Under the new set it finds
  none. **Rebuild the image to pick the fixes up**; an image built before this change still carries
  them.
- **Libraries:** `go-eudi-rpcert` → v0.0.7, `go-platform-kit` → v1.11.4, azugo → v0.40.0,
  `fasthttp` → v1.75.0, `golang.org/x/crypto` → v0.58.0, plus 37 indirect modules with them. None of
  azugo v0.40.0's behaviour changes reaches this service: it sets no cookies, issues no redirects,
  makes no outbound HTTP calls and implements no azugo cache.

### Notes

- `LICENSE` drops the boilerplate header that preceded the licence text (merged from the published
  repository). The licence itself is unchanged: EUPL-1.2.
- The image build now caches Go modules and compiled packages between builds. The binary is built by
  the same command with the same flags; only the build's layer order and caches changed.
- CI's linter moved to golangci-lint v2.14.0. The earlier release cannot read Go 1.27.2's compiled
  standard library and stops before linting anything.
- The gate is green on Go 1.27.2, with the `testhelpers` build tag as CI uses it: `go mod verify`,
  `go mod tidy -diff`, build, vet, `gofmt`, golangci-lint v2.14.0, `go test -race` with **0 races**,
  and both fuzz targets for 30 seconds each with no failing input; `govulncheck` reports **no
  vulnerabilities found**.

## v0.1.0 — initial public release

**The Relying Party onboarding control plane: client lifecycle and verification-key minting, over a registry schema, behind an admin API key.**

It is deliberately small — no HTML, no operator login, no sessions, no background jobs. Onboarding a
Relying Party is entirely API calls. It is **never in the request path of a live verification**: it
prepares the registry data that the wallet-facing and session services read at runtime.

It owns no schema of its own. Every access to the shared registry goes through `SECURITY DEFINER`
stored procedures, as a role that holds no direct table grants.

### Running it

The image entrypoint is `["/server", "web"]`, with `["/server", "health"]` as the healthcheck
command — a compose or orchestrator file that also passes `command: ["web"]` would append a second
argument. Every setting is an environment variable with a safe default where one is possible; the
README lists them, and any secret can be supplied as `<NAME>_FILE` pointing at a mounted file instead.

### What it needs

Go **1.27.0** to build. Libraries at this release:

`go-eudi-rpcert` v0.0.6 · `go-platform-kit` v1.11.3 · `azugo.io/azugo` + `azugo.io/core` v0.38.1
