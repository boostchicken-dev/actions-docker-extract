# Docker Extract

A GitHub Action for extracting files from a Docker Image.  This was forked from shrink and was modified to be compatible with Self Hosted runners, and cross platform in general

```yaml
- uses: boostchicken-dev/actions-docker-extract@v3
  with:
    image: 'ghost:alpine'
    path: '/var/lib/ghost/current/core/built/assets/.'
```

## Inputs

All inputs are required.

| ID  | Description | Examples |
| --- | ----------- | -------- |
| `image` | Docker Image to extract files from | `alpine` `ghcr.io/github/super-linter:latest` |
| `path` | Path (from root) to a file or directory within Image | `files/example.txt` `files` `files/.` |

> :paperclip: To copy the **contents** of a directory the `path` must end with
`/.` otherwise the directory itself will be copied. More information about the
specific rules can be found via the [docker cp][docker-cp] documentation.

## Outputs

| ID  | Description | Example |
| --- | ----------- | ------- |
| `destination` | Destination path containing the extracted file(s) | `.extracted-1598717412/` |

## Examples

### Build, Extract

Using [docker/build-push-action][build-push-action] to build a Docker
Image and then extract the contents of the `/app` directory within the newly
built image to upload as a `dist` artifact.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build Docker Image
        uses: docker/build-push-action@v1
        with:
          repository: my-example-image
          tags: latest
      - uses: boostchicken-dev/actions-docker-extract@v2
        id: extract
        with:
          image: my-example-image
          path: /app/.
      - name: Upload Dist
        uses: actions/upload-artifact@v2
        with:
          path: ${{ steps.extract.outputs.destination }}
          name: dist
```

### Login, Pull, Extract

Using [docker/login-action][login-action] to authenticate with the GitHub
Container Registry to extract from a published Docker Image.

```yaml
jobs:
  extract:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v1
        with:
          registry: ghcr.io
          username: ${{ github.repository_owner }}
          password: ${{ secrets.GHCR_PAT }}
      - uses: boostchicken-dev/actions-docker-extract@v3
        id: extract
        with:
          image: ghcr.io/${{ github.repository }}:latest
          path: /app/.
      - name: Upload Dist
        uses: actions/upload-artifact@v2
        with:
          path: ${{ steps.extract.outputs.destination }}
          name: dist
```

[build-push-action]: https://github.com/docker/build-push-action
[login-action]: https://github.com/docker/login-action
[docker-cp]: https://docs.docker.com/engine/reference/commandline/cp/#extended-description
