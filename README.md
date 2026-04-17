# Pear Desktop Snap Deployment

This repository is only focused in publishing od the snap artifact from the
offical repository <https://github.com/pear-devs/pear-desktop>

See more: <https://github.com/pear-devs/pear-desktop/issues/530>

> For the original authors, I will be glad to pass to you the ownership of this
app in Snapcraft once that publisihng is automated and you have Snapcraft
logins.

## Operations

The [`Publish Snap`](.github/workflows/publish-snap.yml) workflow runs daily
(06:17 UTC) and also on manual dispatch. For each run it:

1. Resolves the latest non-prerelease tag of `pear-devs/pear-desktop` (or the
   tag passed via `workflow_dispatch`).
2. Skips if a matching tag already exists in this repo.
3. Checks out upstream at the tag, renames the package to `pear-desktop`,
   and builds the snap using upstream's own electron-builder pipeline.
4. Verifies the snap's internal `name`/`version`.
5. Publishes to the Snapcraft store with [`snapcore/action-publish`](https://github.com/snapcore/action-publish).
6. Pushes a matching dedup tag back to this repo.

### Required secret

| Name                          | How to generate                                                                              |
| ----------------------------- | -------------------------------------------------------------------------------------------- |
| `SNAPCRAFT_STORE_CREDENTIALS` | `snapcraft export-login --snaps pear-desktop --channels stable --acls package_upload -`      |

Add it under **Settings → Secrets and variables → Actions → New repository
secret**. The token is scoped to a single snap and the `package_upload` ACL,
which minimises blast radius if it leaks. Rotate by running the command
again and updating the secret; old tokens can be revoked from the Snapcraft
dashboard.

Do **not** store these credentials as a GitHub Variable — Variables are
plaintext and can surface in workflow logs.

### Manually republishing a specific version

Use the Actions tab → `Publish Snap` → `Run workflow`, and set:

- `tag`: `vX.Y.Z` (upstream tag, e.g. `v3.11.0`)
- `channel`: `stable` (or `candidate`, `beta`, `edge`)

If the tag already exists in this repo the workflow will skip. Delete the
local tag first to force a rebuild:

```bash
git push origin :refs/tags/vX.Y.Z
```

### Dedup tags

On every successful publish the workflow creates a tag in this repo matching
the upstream tag (e.g. `v3.11.0`). These tags are the source of truth for
"what has already been published." Do not delete them unless you intend to
republish that version.
