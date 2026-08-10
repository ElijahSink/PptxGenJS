# Publishing `@ejsink/pptxgenjs`

This fork publishes to npm as **`@ejsink/pptxgenjs`** via
[npm Trusted Publishing](https://docs.npmjs.com/trusted-publishers) (OIDC).
There is no npm token stored anywhere - GitHub Actions mints a short-lived
OIDC token at publish time, and npm verifies it came from this repo and this
workflow.

## One-time setup

### 1. Bootstrap publish (required once, by hand)

npm cannot configure a Trusted Publisher for a package that does not exist yet -
the settings page only appears once the package is on the registry. So version
`4.1.0` has to be published manually, from a machine you trust:

```bash
npm login
npm run dist
npm publish --access public
```

Every later release goes through CI. This is the only time a human publishes.

### 2. Configure the Trusted Publisher

On npmjs.com, open the package -> **Settings** -> **Trusted Publisher**, choose
**GitHub Actions**, and enter:

| Field                 | Value                |
| --------------------- | -------------------- |
| Organization or user  | `ElijahSink`         |
| Repository            | `PptxGenJS`          |
| Workflow filename     | `publish.yml`        |
| Environment name      | *(leave blank)*      |
| Allowed actions       | `npm publish`        |

The workflow filename must match exactly, including the `.yml` extension. If the
workflow is ever renamed or moved, update it here or publishes start failing.

### 3. Remove any leftover token

If an `NPM_TOKEN` repository secret exists from an earlier setup, delete it. The
workflow does not read it, and an unused long-lived token is pure risk.

## Cutting a release

1. Bump the version in **both** `package.json` and `src/pptxgen.ts`
   (`const VERSION`). CI fails the release if they disagree.
2. Update `CHANGELOG.md`.
3. Rebuild and commit the bundles: `npm run dist`.
4. Merge to the default branch.
5. Tag and push:

   ```bash
   git tag v4.1.1
   git push origin v4.1.1
   ```

The tag must match `package.json` exactly, minus the leading `v`; CI checks this
too. The workflow then installs, verifies versions, typechecks, builds `dist/`
from source, packs, and publishes.

### Dry run

Run the workflow manually from the Actions tab with `dry_run` left checked. It
does everything except publish, ending at `npm pack --dry-run` so you can see the
exact tarball contents. Useful after changing the build or the `files` field.

## Notes

- **Provenance is automatic.** Trusted publishing attaches a provenance
  attestation without the `--provenance` flag. It requires a public repository,
  which this is.
- **npm CLI >= 11.5.1** is required for OIDC, which is newer than what Node 22
  bundles - hence the explicit `npm install -g npm@latest` step.
- **`package.json` `repository`** must point at this fork for provenance to
  verify. It does.
- **Scope.** `@ejsink/*` publishes only if the npm account owns that scope.
  `publishConfig.access: public` is set because scoped packages default to
  restricted, which a free account cannot publish.
