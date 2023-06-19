# cog-action

[Cog](https://github.com/replicate/cog) is an open-source tool that lets you package machine learning models in a standard, production-ready container. To learn more about Cog, visit [cog.run](https://cog.run).

This repo is a GitHub Action that makes it easy to install Cog on a GitHub-hosted runner so you can run, test, and push machine learning models to Replicate, or to your own infrastructure. This action takes care of setting up prerequisites like Dockerx and NVIDIA's CUDA drivers, so you can focus on building and deploying your models without the headache.

## Usage

Add a step to your workflow that uses this action:

```yml
- name: Install Cog
  uses: replicate/cog-action@main
  with:
    replicate_api_token: ${{ secrets.REPLICATE_API_TOKEN }}
```

Now you can run the `cog` CLI in subsequent steps to `cog run`, `cog push`, etc.

### Inputs

This following inputs can be specified using the `with` keyword:

- `cog_version`: The version of Cog to install. Defaults to `v0.8.0-beta3` (a prerelease version with support for pushing GPU models from runners that don't have a GPU!)
- `replicate_api_token`: Your Replicate API token. If set, the Action will automatically authenticate to Replicate using `cog login`. Get a token from [replicate.com/account](https://replicate.com/account), then create a repository secret named REPLICATE_API_TOKEN in your GitHub repo's settings.

### Runners

This action works on GitHub's default runners, but **most models require more disk space and memory** than is available on the default runner. 

GitHub offers more powerful hosted runners that are easy to set up with just a few clicks, but you'll have to [Sign up for GitHub's Hosted Runners beta](https://github.com/features/github-hosted-runners/signup) before you can use them. Note that these runners are only available on paid Team or Enterprise plans, so you'll need to make sure your model repository is owned by a paid GitHub organization rather than an individual user account.

## Manual Deployment

Here's an example that allows you to [trigger a workflow run manually](https://docs.github.com/en/actions/managing-workflow-runs/manually-running-a-workflow) from the Actions tab in your repo:

```yml
name: Push to Replicate

on:
  workflow_dispatch:
    inputs:
      model_name:
        required: true
        description: "The name of the Replicate model to publish, e.g `username/modelname`. The model must already exist on Replicate."

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v3

      - name: Install Cog
        uses: replicate/cog-action@main
        with:
          replicate_api_token: ${{ secrets.REPLICATE_API_TOKEN }}

      - name: Push to Replicate
        run: |
          cog push r8.im/${{ inputs.model_name }}}
```

## Continuous Deployment

Here's an example that pushes to Replicate whenever your repo's `main` branch is updated:

```yml
name: Push to Replicate

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v3

      - name: Install Cog
        uses: replicate/cog-action@main
        with:
          replicate_api_token: ${{ secrets.REPLICATE_API_TOKEN }}

      - name: Push to Replicate
        run: |
          cog push r8.im/zeke/hello-world
```
