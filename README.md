# Trigger Buildkite Pipeline GitHub Action

A [GitHub Action](https://github.com/actions) for triggering a build on a [Buildkite](https://buildkite.com/) pipeline.


<img src="screenshot.png" alt="Screenshot of the Trigger Buildkite GitHub Action Node" width="298" />

## Features

* Creates builds in Buildkite pipelines, setting commit, branch, message.
* Saves the build JSON response to `${HOME}/${GITHUB_ACTION}.json` for downstream actions.

## Usage

Create a [Buildkite API Access Token](https://buildkite.com/docs/apis/rest-api#authentication) with `write_builds` scope, and save it to your GitHub repository’s **Settings → Secrets**. Then you can configure your Actions workflow with the details of the pipeline to be triggered, and the settings for the build.

For example, the following workflow creates a new Buildkite build on every commit:

```workflow
on:
  push:

jobs:
  run_pipeline:
    name: Run Buildkite pipeline
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: buildkite/trigger-pipeline-action@v1.2.0
        env:
          BUILDKITE_API_ACCESS_TOKEN: ${{ secrets.buildkite_api_access_token }}
          PIPELINE: "my-org/my-deploy-pipeline"
          MESSAGE: ":github: Triggered from a GitHub Action"
          COMMIT: "HEAD"
          BRANCH: "main"
```

Example workflow that creates a build based on pull request against main branch:

```workflow
name: Run Buildkite pipelines on pull request to main branch

on:
  push:
  pull_request:
    branches:
      - main

jobs:
  run_pipeline:
    name: Run Buildkite pipeline
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: buildkite/trigger-pipeline-action@v1.2.0
        env:
          BUILDKITE_API_ACCESS_TOKEN: ${{ secrets.buildkite_api_access_token }}
          PIPELINE: "my-org/my-deploy-pipeline"
          MESSAGE: ":github: Triggered from a GitHub Action via pull request to main"
          PULL_REQUEST_ID: $(jq --raw-output .pull_request.number "$GITHUB_EVENT_PATH")
```

Example workflow that creates a build based on `/build` being used in a comment in the PR by a organization member:

```
name: Run Buildkite pipelines on comment in PR

on:
  issue_comment:
    types: [created]

jobs:
  run_pipeline:
    name: Run Buildkite pipeline
    if: github.event.issue.pull_request != '' && contains(github.event.comment.body, '/build') && github.event.comment.author_association == 'MEMBER'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: buildkite/trigger-pipeline-action@v1.2.0
        env:
          BUILDKITE_API_ACCESS_TOKEN: ${{ secrets.buildkite_api_access_token }}
          PIPELINE: "my-org/my-deploy-pipeline"
          MESSAGE: ":github: Triggered from a GitHub Action via pull request comment"
          PULL_REQUEST_ID: ${{github.event.issue.number}}
```

## Configuration Options

The following environment variable options can be configured:

|Env var|Description|Default|
|-|-|-|
|PIPELINE|The pipline to create a build on, in the format `<org-slug>/<pipeline-slug>`||
|COMMIT|The commit SHA of the build. Optional.|`$GITHUB_SHA`|
|BRANCH|The branch of the build. Optional.|`$GITHUB_REF`|
|PULL_REQUEST_ID|The PR of the build. Optional.||
|MESSAGE|The message for the build. Optional.||
|BUILD_ENV_VARS|Additional environment variables to set on the build, in JSON format. e.g. `{"FOO": "bar"}`. Optional. ||

## Development

To run the test workflow, you use [act](https://github.com/nektos/act) which will run it just as it does on GitHub:

```bash
act
```

## Contributing

* Fork this repository
* Create a new branch for your work
* Push up any changes to your branch, and open a pull request. Don't feel it needs to be perfect — incomplete work is totally fine. We'd love to help get it ready for merging.

## Releasing

* Create a new GitHub release. The version numbers in the readme will be automatically updated.

## Roadmap

* Add a `WAIT` option for waiting for the Buildkite build to finish.
* Support other properties available in the [Buildkite Builds REST API](https://buildkite.com/docs/apis/rest-api/builds#create-a-build), such as environment variables and meta-data.

Contributions welcome! ❤️
