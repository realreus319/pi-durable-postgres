# Releasing

The source repository is **github.com/realreus319/pi-durable-postgres**.
The public npm package is **@ainavite/pi-durable-postgres**. These identities
intentionally differ.

## One-time GitHub/npm setup

1. Ensure the npm organization grants the publisher permission for
   `@ainavite/pi-durable-postgres`.
2. In the package's npm settings, add a GitHub Actions trusted publisher:
   GitHub owner `realreus319`, repository `pi-durable-postgres`, workflow filename
   `ci.yml`, environment `npm`.
3. In the GitHub repository, create the `npm` environment. Add approval rules and
   permitted release tags if required by the organization.
4. Protect release tags and review changes to the publish workflow. A pushed
   matching version tag can publish after CI and environment approval.

The release job uses OIDC trusted publishing; it needs `id-token: write` and npm
CLI 11.5.1 or later. It does not store a long-lived npm write token in GitHub.
npm's trusted-publishing setup and its exact workflow/environment matching are
documented in the [npm trusted publishers guide](https://docs.npmjs.com/trusted-publishers/).

## Upstream release watch

Dependabot (`.github/dependabot.yml`) opens a PR when a new `@earendil-works`
release appears, by bumping the exact **dev** pins. Dependabot does not touch
`peerDependencies`; move the peer pin in the same PR once CI is green, together
with the README compatibility table and a CHANGELOG entry. Never merge on
assumption — the conformance suite is the gate.

## Release checklist

- Update package version and CHANGELOG.md.
- Run `pnpm install --frozen-lockfile`, `pnpm lint:types`, `pnpm test` and
  `pnpm test:package`.
- Check the README compatibility table against actually tested peer versions.
  Update the Pi Durable, Chord and Pi AI development pins together; the consumer
  test takes its Pi AI version from package.json. Pi Durable is experimental:
  do not widen peer ranges on assumption alone.
- Review `npm pack --dry-run` output: JS, declarations, source-map sources,
  documentation and license must be included, but no secrets or test data.
- Confirm live-deletion limitations and any changed adapter contracts are accurate.
- Commit and push the reviewed changes, then deliberately create/push a tag
  matching package.json, e.g. `v0.3.0`.

Branch pushes and PRs run tests but do not publish. A `v*` tag push runs the same
matrix, then the release job verifies the package identity and exact tag/version
match before calling `npm publish --access public`. If npm already contains this
version with the exact same `gitHead`, the upload is skipped. A different or unknown
commit is rejected instead of treating it as a successful release. Publishing runs
`prepack` to build dist. CI uses the packageManager-pinned pnpm version.

For the first publication, the package must exist on npm before its settings can
be assigned a trusted publisher. An organization owner can make that initial
publication from the clean, committed release checkout using an authenticated npm
account. After the trusted publisher is configured, version tags publish from
GitHub Actions without a write token.

The consumer smoke test creates an isolated temporary project, installs the real
tarball, type-checks without skipLibCheck, and imports the compiled package. It
does not contact a model provider or modify a database.

Never reuse an already published npm version. Correct a bad release with a new
version and an explicit changelog entry.
