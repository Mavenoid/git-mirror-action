# Git Mirror Action

A GitHub Action for [mirroring a git repository](https://help.github.com/en/articles/duplicating-a-repository#mirroring-a-repository-in-another-location) to another location via SSH.

> N.B. this will discard any changes in the destination repo that are not already in the source-repo. For this reason it is not a reliable approach for 2-way sync unless both repos are very infrequently updated.

## Inputs

### `source-repo`

**Required** SSH URLs of the source repo.

### `destination-repo`

**Required** SSH URLs of the destination repo.

### `ssh-private-key`

**Required** SSH key to authenticate to both the source and destination repo.

Create a [SSH key](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key) which has access to both repositories. On GitHub they are called "deploy keys". Store [the private key as a secret](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets) and use it in your workflow as seen in the example usage below.

### `github-token`

**Optional** Defaults to `${{ github.token }}`. A github token with at least the `read:packages` scope.

This is used to clone the pre-built docker image, which speeds up execution vs. building the docker image every time. It does not need to be a token for Mavenoid as the Docker image itself is public, but GitHub does not allow you to read packages without a token, even when the packages are public.

## Example workflow

```
name: Mirror to Bitbucket Repo

on: [ push, delete, create ]

jobs:
  git-mirror:
    runs-on: ubuntu-latest
    steps:
      - uses: mavenoid/git-mirror-action@2.2.0
        with:
          source-repo: 'git@github.com:wearerequired/swisscom-magazine.git'
          destination-repo: 'git@bitbucket.org:wearerequired/git-mirror-action.git'
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Development

### Running pre-built image locally

Authenticate with docker:

```
docker login docker.pkg.github.com -u YOUR_GITHUB_USERNAME
```

When prompted, provide a GitHub access token with `read:packages` scope.

Run the prebuilt image:

docker run --rm -e "SSH_PRIVATE_KEY=$(cat ~/.ssh/id_rsa)" "docker.pkg.github.com/mavenoid/git-mirror-action/git-mirror-action:2.2.0" "$SOURCE_REPO" "$DESTINATION_REPO"

### Building and running locally

Clone this repository and then run:

```
docker run --rm -e "SSH_PRIVATE_KEY=$(cat ~/.ssh/id_rsa)" $(docker build -q .) "$SOURCE_REPO" "$DESTINATION_REPO"
```

### Releasing

To release a new version of git-mirror-action:

1. Find and replace the current version number ("2.2.0") with the new version number
2. Commit and push to GitHub
3. [Create and publish a new release](https://github.com/Mavenoid/git-mirror-action/releases/new) with the "Tag version" exactly matching the new version number.

The docker image is automatically built and published once you have created the release, so the release should become usable within about 1 minute.

## License

The Dockerfile and associated scripts and documentation in this project are released under the [MIT License](LICENSE).
