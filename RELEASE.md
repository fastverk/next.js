# Releasing fastverk next-swc binaries

The `release-binaries` workflow builds the `@next/swc` native binaries for the
three platforms savvi-studio consumes and publishes them as GitHub Release
assets so downstream projects don't have to vendor 467MB `.node` files into git
or build them locally.

## Platforms

- `darwin-arm64` (Apple Silicon dev machines)
- `linux-x64-gnu` (CI, x86_64 Linux)
- `linux-arm64-gnu` (CI, arm64 Linux)

Each tarball mirrors the layout that `npm publish` produces for the upstream
`@next/swc-<platform>` package: the tarball root is a `package/` directory
containing `package.json` + `next-swc.<platform>.node`.

## Cutting a release

1. Make sure all patches you want to ship are on `fastverk/patched` (or whatever
   branch the release tag will point at).
2. Pick a tag name matching `v*-fastverk*`. Convention:

   `v<upstream-version>-fastverk-<short-feature>-<n>`

   e.g. `v16.1.4-fastverk-extension-alias-1`, then `-2` for the next iteration.

3. Tag and push:

   ```bash
   git tag v16.1.4-fastverk-extension-alias-1
   git push fastverk v16.1.4-fastverk-extension-alias-1
   ```

4. The tag push triggers `.github/workflows/release-binaries.yml`. Once all
   three `build-native` jobs succeed, the `release` job creates a GitHub
   Release at `https://github.com/fastverk/next.js/releases/tag/<tag>` with
   three `.tgz` files attached.

You can also run the workflow manually via `workflow_dispatch`, passing the
target tag as an input — useful for re-running after a transient failure
without having to retag.

## Asset URLs

For tag `<TAG>`, consumers fetch:

```
https://github.com/fastverk/next.js/releases/download/<TAG>/next-swc-darwin-arm64-<TAG>.tgz
https://github.com/fastverk/next.js/releases/download/<TAG>/next-swc-linux-x64-gnu-<TAG>.tgz
https://github.com/fastverk/next.js/releases/download/<TAG>/next-swc-linux-arm64-gnu-<TAG>.tgz
```

## Consuming in savvi-studio

In `pnpm-workspace.yaml` (or `package.json#pnpm.overrides`), pin each
platform package to its tarball URL. pnpm treats `.tgz` URLs as regular
registry tarballs, so this drops directly into the existing resolution:

```yaml
overrides:
  '@next/swc-darwin-arm64': 'https://github.com/fastverk/next.js/releases/download/v16.1.4-fastverk-extension-alias-1/next-swc-darwin-arm64-v16.1.4-fastverk-extension-alias-1.tgz'
  '@next/swc-linux-x64-gnu': 'https://github.com/fastverk/next.js/releases/download/v16.1.4-fastverk-extension-alias-1/next-swc-linux-x64-gnu-v16.1.4-fastverk-extension-alias-1.tgz'
  '@next/swc-linux-arm64-gnu': 'https://github.com/fastverk/next.js/releases/download/v16.1.4-fastverk-extension-alias-1/next-swc-linux-arm64-gnu-v16.1.4-fastverk-extension-alias-1.tgz'
```

After updating, `pnpm install --force` to refetch.

## Build commands (for reference)

The workflow uses the same commands Next.js's own `build_and_deploy.yml`
invokes for these targets:

- macOS host: `pnpm dlx turbo@<v> run build-native-release -- --target aarch64-apple-darwin`
- Linux glibc (in napi-rs docker image): `npm run build-native-release -- --target x86_64-unknown-linux-gnu`
- Linux arm64 (native arm runner): `npm run build-native-release -- --target aarch64-unknown-linux-gnu`

Toolchain versions (`NAPI_CLI_VERSION`, `TURBO_VERSION`, `NODE_LTS_VERSION`,
`MACOSX_DEPLOYMENT_TARGET`) are mirrored from `build_and_deploy.yml` to stay in
lockstep with upstream.
