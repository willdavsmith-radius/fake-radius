# Fake Radius Repository

This directory contains an example repository you can push to your own
organization (for example `willdavsmith-radius/fake-radius`). Its workflow
listens for a `repository_dispatch` event coming from the deployment-engine
repository and re-publishes the container image to GHCR on behalf of that
organization.

## Prerequisites

1. Install the same GitHub App that builds the image in the Radius organization.
   The installation must grant at least:
   - `contents:read` (to fetch workflow files)
   - `actions:write`
   - `packages:write`
2. Add the following secrets (or adjust the workflow to use your own names):
   - `GH_APP_ID`: numeric GitHub App ID.
   - `GH_APP_PRIVATE_KEY`: the PEM contents of the App's private key.
   - Optionally `SOURCE_APP_ID` / `SOURCE_APP_PRIVATE_KEY` if you prefer to use a
     different installation for retrieving the deployment-engine source.
3. Ensure the App installation in the source organization (`willdavsmith-azure-octo`)
   has read access to the `deployment-engine` repository.
4. Set repository variables if you need to override defaults:
   - `SOURCE_REPOSITORY` (defaults to payload `source_repo`)
   - `CONTAINER_REGISTRY` (falls back to payload `registry`)

> ℹ️ The workflow uses the scripts that live inside the deployment-engine
> repository (`.github/scripts/build-container.sh`, `.github/scripts/push-container.sh`).
> No additional build files are needed in this repo.

## Triggering the Workflow

After copying this directory into a standalone repository, push it to GitHub and
configure the deployment-engine workflow to dispatch events to it. An example
step that triggers the dispatch is added in `.github/workflows/build-and-push.yaml`
in this repository. The dispatch payload includes:

- Container registry, image name, and tag.
- Git SHA/ref for the build.
- Platforms requested for the multi-arch build.

When the workflow runs it:

1. Creates an installation token for the Radius organization (for GHCR push).
2. Creates another token for the source org to check out the deployment-engine
   commit referenced in the payload.
3. Builds the container image with Buildx for the requested platforms.
4. Pushes the image to GHCR.

Adjust the workflow as needed if your registry or image naming differs.
# fake-radius
