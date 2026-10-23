# Godot preview Docker images (GDScript)

This fork publishes Godot's *official* dev/alpha/beta/RC editor builds and matching export templates as a Docker image in GitHub Container Registry (GHCR). It builds the GDScript editor, **not** the Mono/.NET variant.

The upstream source is [`godotengine/godot-builds` releases](https://github.com/godotengine/godot-builds/releases), rather than the GitHub `latest` release endpoint (which excludes prereleases).

## Image tags

For Godot `4.8-beta1`:

```bash
docker pull ghcr.io/fivepebblesgpt/godot-ci-dev:4.8-beta1
docker run --rm ghcr.io/fivepebblesgpt/godot-ci-dev:4.8-beta1 godot --headless --version
```

`ghcr.io/fivepebblesgpt/godot-ci-dev:preview` points to the most recently auto-discovered official prerelease. For reproducible builds, **pin the full release tag**; the `preview` alias moves over time.

The export templates are located at `/root/.local/share/godot/export_templates/4.8.beta1` for version `4.8-beta1`. On GitHub Actions jobs using the image, the container entrypoint may run under a different `$HOME`, so reuse or move the templates as needed.

## Automatic and manual publication

- **Every six hours**, `Check Godot Preview Releases` polls the 100 most recent official release records for the newest published prerelease (`dev`, `alpha`, `beta`, or `rc`). Scheduled workflows are best-effort and may run late; GitHub cannot directly subscribe to another repository's release events.
- When the corresponding **fork release does not already exist**, it calls the reusable `Build Godot Preview Image` workflow. The builder validates the release's binary and export-template asset names, builds the upstream-compatible `Dockerfile`, pushes a versioned GHCR image and the `preview` alias, then creates the fork's GitHub prerelease **only after the push succeeded**. The fork release acts as the published marker.
- To build a particular version immediately, run **Actions → Build Godot Preview Image → Run workflow**, set `godot_tag` (e.g. `4.8-beta1`), and leave `update_preview` unchecked unless intentionally moving the floating alias.
- The checker also supports manual dispatch with an optional tag. If that tag already has a fork release, it skips it; use the builder directly to rebuild.
- A release manually published in this fork also runs the builder if its tag matches an official preview. A release created by `GITHUB_TOKEN` does not itself trigger another workflow, preventing a build loop.

Only the latest preview is auto-discovered, so if multiple previews are published between polls, earlier versions can be built manually. Release/check records can be recreated manually after a failed or deleted package.

## Permissions and caveats

The workflows request only the necessary `contents: write` (for mirrored GitHub releases) and `packages: write` (for pushing GHCR images) at the build boundary. No Docker Hub credentials or personal access token are needed. In the GitHub repository settings, ensure workflow permissions and package publication are allowed; if the package is initially private, set its visibility to **public** from the GHCR package settings when ready.

The original upstream `manual_build.yml` still exists independently. It is not part of this preview publishing process and may require Docker Hub credentials; use the preview builder described here instead.

Unlike nightly compiling of Godot's master source branch, this workflow packages official prerelease *binaries*. Godot can change APIs between prerelease versions. Test project exports independently.
