# setup-cog

[Cog](https://github.com/replicate/cog) is an open-source tool that lets you package machine learning models in a standard, production-ready container. To learn more about Cog, visit [cog.run](https://cog.run).

This repo is a GitHub Action that makes it easy to install Cog on a GitHub-hosted runner so you can run, test, and push machine learning models to Replicate, or to your own infrastructure. This action takes care of setting up prerequisites like Dockerx and NVIDIA's CUDA drivers, so you can focus on building and deploying your models without the headache.

**⚡️ [Cog supports building and pushing GPU models from machines that don't have GPUs](https://github.com/replicate/cog/pull/1069) ⚡️**. So in many cases you can use this action to push your GPU model to Replicate, even if you're using a GitHub-hosted runner that doesn't have a GPU.

## Usage

Add a step to your workflow that uses this action:

```yml
- name: Setup Cog
  uses: replicate/setup-cog@v1
  with:
    token: ${{ secrets.REPLICATE_API_TOKEN }}

- name: Download weights
  run: cog run script/download-weights

- name: Push to Replicate
  run: cog push r8.im/zeke/hello-world
```

Now you can run the `cog` CLI in subsequent steps to `cog run`, `cog push`, etc.

### Inputs

This following inputs can be specified using the `with` keyword:

- `cog-version`: The version of Cog to install in the form `vX.Y.Z`. See [Cog releases] for a list of available versions. Defaults to `v0.8.0-beta3` (a prerelease version with support for pushing GPU models from runners that don't have a GPU!)
- `token`: Your Replicate API token. If set, the Action will automatically authenticate to Replicate using `cog login`. To use this feature, get a token from [replicate.com/account](https://replicate.com/account), then create a repository secret named `REPLICATE_API_TOKEN` in your GitHub repo's settings.
- `install-cuda`: Install NVIDIA CUDA Toolkit. Defaults to `true`. This flag can be set to `false` when building Cog containers that don't require GPU.

## Example: Manual Deployment

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

      - name: Setup Cog
        uses: replicate/setup-cog@v1
        with:
          token: ${{ secrets.REPLICATE_API_TOKEN }}

      - name: Push to Replicate
        run: |
          cog push r8.im/${{ inputs.model_name }}}
```

## Example: Continuous Deployment

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

      - name: Setup Cog
        uses: replicate/setup-cog@v1
        with:
          token: ${{ secrets.REPLICATE_API_TOKEN }}

      - name: Push to Replicate
        run: |
          cog push r8.im/zeke/hello-world
```

## Limitations

GitHub's hosted runners don't have GPUs (yet?), so you can't currently run GPU models unless you [self-host your own runner](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners).

That said, Cog can still be used to `build` and `push` GPU models, even from a Linux environment that doesn't have a GPU. So you can push GPU models to Replicate, but you can't run them as part of your workflow.

### Using custom hosted runners

This action works on GitHub's default runners, but **most models require more disk space and memory** than is available on the default runner. 

GitHub offers more powerful hosted runners that are easy to set up with just a few clicks, but you'll have to [Sign up for GitHub's Hosted Runners beta](https://github.com/features/github-hosted-runners/signup) before you can use them. Note that these runners are only available on paid Team or Enterprise plans, so you'll need to make sure your model repository is owned by a paid GitHub organization rather than an individual user account.

Once you've set up a hosted runner, you can use it by adding the [`runs-on`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idruns-on) key to your job:

```yml
jobs:
  do_cog_stuff:
    runs-on: my-fancy-runner-with-more-disk-and-memory
    steps:
      - name: Check out code
        uses: actions/checkout@v3

      - name: Setup Cog
        uses: replicate/setup-cog@v1
```