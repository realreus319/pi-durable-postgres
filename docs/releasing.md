# Releasing

The source repository is **github.com/realreus319/pi-durable-postgres**.
The public npm package is **@ainavite/pi-durable-postgres**. These identities
intentionally differ.

## One-time GitHub/npm setup

1. Ensure the npm organization grants the publisher permission for
   `@ainavite/pi-durable-postgres`.
2. Create a granular npm access token limited to `@ainavite/pi-durable-postgres`
   with publish permission and bypass 2FA enabled.
3. Add the token as the GitHub Actions repository secret `NPM_TOKEN`. Never commit
   the token or place it in the workflow file.
4. Protect release tags and review changes to the publish workflow. A pushed
   matching version tag publishes after CI.

This token-based flow follows the maintainer's current choice not to enable 2FA.
npm has announced that direct publishing with bypass-2FA tokens will be removed
in January 2027. Automated publishing will need an authentication change before
then; npm's Trusted Publisher setup requires an interactive 2FA challenge.
See [npm access tokens](https://docs.npmjs.com/about-access-tokens/) and
[npm Trusted Publishers](https://docs.npmjs.com/trusted-publishers/).

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
match before calling `npm publish --access public` with the `NPM_TOKEN` secret.
If npm already contains this
version with the exact same `gitHead`, the upload is skipped. A different or unknown
commit is rejected instead of treating it as a successful release. Publishing runs
`prepack` to build dist. CI uses the packageManager-pinned pnpm version.

For token-based publishing, the `NPM_TOKEN` secret must have publish permission
for this package. The token is not included in the repository or package.

The consumer smoke test creates an isolated temporary project, installs the real
tarball, type-checks using the consuming app's `skipLibCheck` setting, and imports the compiled package. It
does not contact a model provider or modify a database.

Never reuse an already published npm version. Correct a bad release with a new
version and an explicit changelog entry.
