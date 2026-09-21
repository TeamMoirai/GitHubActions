# GitHub Actions

> OSS repository uses and maintained GitHub Actions "reusable workflows" and "composite actions".
>

[![actions lint](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_actions-lint.yaml/badge.svg)](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_actions-lint.yaml)
[![Readme_CN](https://img.shields.io/badge/README-中文-red)](https://github.com/TeamMoirai/GitHubActions/blob/master/README.md)

[![Test benchmark-runnable](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-benchmark-runnable.yaml/badge.svg?event=pull_request)](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-benchmark-runnable.yaml)
[![Test check-metas](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-check-metas.yaml/badge.svg?event=pull_request)](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-check-metas.yaml)
[![Test checkout](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-checkout.yaml/badge.svg?event=pull_request)](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-checkout.yaml)
[![Test clean-packagejson-branch](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-clean-packagejson-branch.yaml/badge.svg?event=pull_request)](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-clean-packagejson-branch.yaml)
[![Test create-release](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-create-release.yaml/badge.svg?event=pull_request)](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-create-release.yaml)
[![Test setup-dotnet](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-setup-dotnet.yaml/badge.svg?event=pull_request)](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-setup-dotnet.yaml)
[![Test update-packagejson](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-update-packagejson.yaml/badge.svg?event=pull_request)](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_test-update-packagejson.yaml)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
# 📖 Table of Contents

- [🔑 Required Secrets](#-required-secrets)
- [♻️ Reusable workflows](#-reusable-workflows)
  - [clean-packagejson-branch](#clean-packagejson-branch)
  - [create-release](#create-release)
  - [dd-event-post](#dd-event-post)
  - [increment-version](#increment-version)
  - [prevent-github-change](#prevent-github-change)
  - [stale-issue](#stale-issue)
  - [update-packagejson](#update-packagejson)
  - [validate-tag](#validate-tag)
- [🎬 Actions](#-actions)
  - [benchmark-runnable](#benchmark-runnable)
  - [check-metas](#check-metas)
  - [checkout](#checkout)
  - [download-artifact](#download-artifact)
  - [setup-dotnet](#setup-dotnet)
  - [unity-builder](#unity-builder)
  - [upload-artifact](#upload-artifact)
- [🤝 Special Thanks](#-special-thanks)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 🔑 Required Secrets

Some workflows require GitHub Secrets to be configured in the repository settings. Go to **Settings → Secrets and variables → Actions** to add them.

| Secret Name | Used By | Description |
| ---- | ---- | ---- |
| `NUGET_KEY` | `create-release` | NuGet.org API key for pushing packages |
| `DD_API_KEY` | `dd-event-post` | Datadog API key for posting events |
| `ACTIONBOT_APPID` | `update-packagejson` | GitHub App ID for bot token authentication |
| `ACTIONBOT_PRIVATE_KEY` | `update-packagejson` | GitHub App private key for bot token authentication |
| `AZURE_OIDC_CLIENTID` | `benchmark-cleanup`, `benchmark-execute` | Azure AD application client ID (OIDC) |
| `AZURE_OIDC_TENANTID` | `benchmark-cleanup`, `benchmark-execute` | Azure AD tenant ID (OIDC) |
| `AZURE_OIDC_SUBSCRIPTIONID` | `benchmark-cleanup`, `benchmark-execute` | Azure subscription ID (OIDC) |
| `BENCHMARK_VM_SSH_KEY` | `benchmark-execute` | SSH private key for connecting to benchmark VMs |

> [!NOTE]
> Secrets are only required when the corresponding workflow feature is used. For example, `NUGET_KEY` is only needed when `nuget-push: true` is passed to `create-release`.

> [!TIP]
> When calling reusable workflows, use `secrets: inherit` to pass repository secrets to the called workflow.

# ♻️ Reusable workflows

## clean-packagejson-branch

> [See workflow](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/workflows/clean-packagejson-branch.yaml)

Delete specic github branch. Mainly used for cleanup branch created by [update-packagejson](#update-packagejson) workflow. Action has following limitation to prevent accidental deletion.

1. Branch is NOT default branch.
2. Branch is created & commited by github-actions[bot].

**Sample usage**

```yaml
name: Build-Release

on:
  workflow_dispatch:

jobs:
  cleanup:
    permissions:
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/clean-packagejson-branch.yaml@master
    with:
      branch: branch_name_to_delete
```

## create-release

> [See workflow](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/workflows/create-release.yaml)

Create GitHub Release, upload NuGet and upload artifact to release assets. Mainly used for NuGet and Unity release workflow.

Required secrets.

| SecretKey | When | Description |
| ---- | ---- | ---- |
| `NUGET_KEY` | `with.nuget-push` is true | This secret is required to push nupkg, snupkg to NuGet.org |

**Sample usage**

Create release only.

```yaml
name: Build-Release

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "tag: git tag you want create. (sample 1.0.0)"
        required: true
      dry-run:
        description: "dry_run: true will never create release/nuget."
        required: true
        default: false
        type: boolean

jobs:
  create-release:
    uses: TeamMoirai/GitHubActions/.github/workflows/create-release.yaml@master
    with:
      commit-id: ''
      tag: ${{ inputs.tag }}
      dry-run: ${{ inputs.dry-run }} # if true, delete tag after Release creation & 60s later.
      nuget-push: false
      release-upload: false
    secrets: inherit
```

Change release name not to use `Ver.` prefix.

```yaml
name: Build-Release

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "tag: git tag you want create. (sample 1.0.0)"
        required: true
      dry-run:
        description: "dry_run: true will never create release/nuget."
        required: true
        default: false
        type: boolean

jobs:
  create-release:
    uses: TeamMoirai/GitHubActions/.github/workflows/create-release.yaml@master
    with:
      commit-id: ''
      tag: ${{ inputs.tag }}
      dry-run: ${{ inputs.dry-run }} # if true, delete tag after Release creation & 60s later.
      nuget-push: false
      release-upload: false
      release-format: '{0}'
    secrets: inherit
```

Download other workflows artifacts to upload to release assets.

```yaml
name: Build-Release

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "tag: git tag you want create. (sample 1.0.0)"
        required: true
      dry-run:
        description: "dry_run: true will never create release/nuget."
        required: true
        default: false
        type: boolean

jobs:
  create-release:
    uses: TeamMoirai/GitHubActions/.github/workflows/create-release.yaml@master
    with:
      commit-id: ''
      tag: ${{ inputs.tag }}
      dry-run: ${{ inputs.dry-run }} # if true, delete tag after Release creation & 60s later.
      nuget-push: false
      release-upload: true
      release-asset-path: |
        ./FooBar/win-amd64/FooBar.pdb
        ./FooBar/win-arm64/FooBar.pdb
      download-run-id: '123456789' # specify run id to download artifacts from.
    secrets: inherit
```

Build .NET then create release. `create-release` will push nuget packages.

```yaml
name: Build-Release

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "tag: git tag you want create. (sample 1.0.0)"
        required: true
      dry-run:
        description: "dry_run: true will never create release/nuget."
        required: true
        default: false
        type: boolean

jobs:
  build-dotnet:
    runs-on: ubuntu-24.04
    timeout-minutes: 3
    defaults:
      run:
        working-directory: ./Sandbox
    steps:
      - uses: actions/checkout@v4
      - uses: TeamMoirai/GitHubActions/.github/actions/setup-dotnet@master
      - run: dotnet build -c Release -p:Version=${{ inputs.tag }}
      - run: dotnet pack --no-build -c Release -p:Version=${{ inputs.tag }} -p:IncludeSymbols=true -p:SymbolPackageFormat=snupkg -o ./publish
      - name: upload artifacts
        uses: TeamMoirai/GitHubActions/.github/actions/upload-artifact@master
        with:
          name: nuget
          path: ./Sandbox/publish
          retention-days: 1

  create-release:
    needs: [build-dotnet]
    uses: TeamMoirai/GitHubActions/.github/workflows/create-release.yaml@master
    with:
      commit-id: ''
      tag: ${{ inputs.tag }}
      dry-run: ${{ inputs.dry-run }} # if true, delete tag after Release creation & 60s later.
      nuget-push: true
      release-upload: false
    secrets: inherit                 # to allow workflow to access NUGET_KEY secret
```

Build .NET and Unity, then create release. `create-release` will push nuget packages and upload unitypackage to release assets.

```yaml
name: Build-Release

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "tag: git tag you want create. (sample 1.0.0)"
        required: true
      dry-run:
        description: "dry_run: true will never create release/nuget."
        required: true
        default: false
        type: boolean

jobs:
  update-packagejson:
    if: ${{ github.actor != 'dependabot[bot]' }}
    permissions:
      actions: read
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/update-packagejson.yaml@master
    with:
      file-path: |
        ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo/package.json
        ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo.Plugin/package.json
        ./Sandbox/Sandbox.Godot/addons/Foo/plugin.cfg
        ./Sandbox/Directory.Build.props
      tag: ${{ inputs.tag }}
      use-bot-token: false # false to use GITHUB_TOKEN, true to use GitHub App token
      dry-run: false

  build-dotnet:
    runs-on: ubuntu-24.04
    timeout-minutes: 3
    defaults:
      run:
        working-directory: ./Sandbox
    steps:
      - uses: actions/checkout@v4
      - uses: TeamMoirai/GitHubActions/.github/actions/setup-dotnet@master
      - run: dotnet build -c Release -p:Version=${{ inputs.tag }}
      - run: dotnet pack --no-build -c Release -p:Version=${{ inputs.tag }} -p:IncludeSymbols=true -p:SymbolPackageFormat=snupkg -o ./publish
      - name: upload artifacts
        uses: TeamMoirai/GitHubActions/.github/actions/upload-artifact@master
        with:
          name: nuget
          path: ./Sandbox/publish
          retention-days: 1

  build-unity:
    needs: [update-packagejson]
    runs-on: ubuntu-24.04
    timeout-minutes: 15
    steps:
      - run: echo ${{ needs.update-packagejson.outputs.sha }}
      - uses: actions/checkout@v4
        with:
          ref: ${{ needs.update-packagejson.outputs.sha }}
      # Store artifacts.
      - uses: TeamMoirai/GitHubActions/.github/actions/upload-artifact@master
        with:
          name: Sandbox.Unity.unitypackage
          path: ./Sandbox/Sandbox.Unity/output/Sandbox.Unity.unitypackage
          if-no-files-found: error
      - uses: TeamMoirai/GitHubActions/.github/actions/upload-artifact@master
        with:
          name: Sandbox.Unity.Plugin.unitypackage
          path: ./Sandbox/Sandbox.Unity/output/Sandbox.Unity.Plugin.unitypackage
          if-no-files-found: error

  create-release:
    needs: [update-packagejson, build-dotnet, build-unity]
    uses: TeamMoirai/GitHubActions/.github/workflows/create-release.yaml@master
    with:
      commit-id: ${{ needs.update-packagejson.outputs.sha }}
      tag: ${{ inputs.tag }}
      dry-run: ${{ inputs.dry-run }} # if true, delete tag after Release creation & 60s later.
      nuget-push: true
      release-upload: true
      release-asset-path: |
        ./Sandbox.Unity.unitypackage/Sandbox.Unity.unitypackage
        ./Sandbox.Unity.Plugin.unitypackage/Sandbox.Unity.Plugin.unitypackage
        ./nuget/ClassLibrary.${{ inputs.tag }}.nupkg
        ./nuget/ClassLibrary.${{ inputs.tag }}.snupkg
    secrets: inherit                 # to allow workflow to access NUGET_KEY secret

  cleanup:
    if: ${{ needs.update-packagejson.outputs.is-branch-created == 'true' }}
    needs: [update-packagejson]
    permissions:
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/clean-packagejson-branch.yaml@master
    with:
      branch: ${{ needs.update-packagejson.outputs.branch-name }}
```


## dd-event-post

> [See workflow](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/workflows/dd-event-post.yaml)

Post Datadog event.

Required secrets.

| SecretKey | Description |
| ---- | ---- |
| `DD_API_KEY` | Datadog API key for posting events |

1. Use for Pull Request Merge event.

**Sample usage**

```yaml
name: PR Merged

on:
  pull_request:
    types: [closed]

jobs:
  post:
    if: ${{ github.event.pull_request.merged == true }}
    uses: TeamMoirai/GitHubActions/.github/workflows/dd-event-post.yaml@master
    secrets: inherit
```

## increment-version

> [See workflow](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/workflows/increment-version.yaml)

Update specified version file with incremented version. Mainly used for [post-release workflow](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/workflows/_post-release.yaml).

**Sample usage**

Following workflow will increment patch version from released tag and update specified package.json, plugin.cfg and Directory.Build.props files with new version with `-dev` suffix.

```yaml
name: Post Release

on:
  release:
    types: [published]

jobs:
  new-version:
    permissions:
      actions: read
      contents: read
    uses: TeamMoirai/GitHubActions/.github/workflows/increment-version.yaml@master
    with:
      ref: ${{ github.event.repository.default_branch }}
      tag: ${{ github.ref_name }} # tag value will here. 1.2.1
      type: patch
      suffix: "-dev"

  update-packagejson:
    needs: [new-version]
    permissions:
      actions: read
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/update-packagejson.yaml@master
    with:
      ref: ${{ github.event.repository.default_branch }}
      file-path: |
        ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo/package.json
        ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo.Plugin/package.json
        ./Sandbox/Sandbox.Godot/addons/Foo/plugin.cfg
        ./Sandbox/Directory.Build.props
      tag: ${{ needs.new-version.outputs.version }}
      use-bot-token: false # false to use GITHUB_TOKEN, true to use GitHub App token
      dry-run: false

```

## prevent-github-change

> [See workflow](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/workflows/prevent-github-change.yaml)

Prevent fork users to change files triggered by. Only Organization contributors can change these files.

**Sample usage**

```yaml
name: Prevent github change
on:
  pull_request:
    paths:
      - ".github/**/*.yaml"
      - ".github/**/*.yml"

jobs:
  detect:
    permissions:
      contents: read
    uses: TeamMoirai/GitHubActions/.github/workflows/prevent-github-change.yaml@master
```


## stale-issue

> [See workflow](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/workflows/stale-issue.yaml)

Stale issue and PRs.
Mainly used for Issue/PR management.

**Sample usage**


```yaml
name: "Close stale issues"

on:
  schedule:
    - cron: "0 0 * * *"

jobs:
  stale:
    permissions:
      contents: read
      pull-requests: write
      issues: write
    uses: TeamMoirai/GitHubActions/.github/workflows/stale-issue.yaml@master
```

## update-packagejson

> [See workflow](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/workflows/update-packagejson.yaml)

Update specified `Unity package.json` and `Godot plugin.cfg` version with tag version. Mainly used for UPM and Godot plugin release workflow.

Required secrets (when `use-bot-token` is true).

| SecretKey | Description |
| ---- | ---- |
| `ACTIONBOT_APPID` | GitHub App ID for bot token authentication |
| `ACTIONBOT_PRIVATE_KEY` | GitHub App private key for bot token authentication |

Consider use GitHub App token, when your repository want's restrict direct push to default branch.

```yaml
jobs:
  use-bot-token:
    permissions:
      actions: read
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/update-packagejson.yaml@master
    with:
      file-path: ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo/package.json
      tag: ${{ inputs.tag }}
      use-bot-token: true # <-- Use GitHub App token
      dry-run: ${{ inputs.dry-run }}
```

If you want to run dotnet run during workflow, you can specify `dotnet-run-path` input. Make sure arguments are always `--version "{tag}"`.

```yaml
jobs:
  update-packagejson:
    permissions:
      actions: read
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/update-packagejson.yaml@master
    with:
      file-path: ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo/package.json
      # you can write multi path.
      dotnet-run-path: |
        ./Sandbox/Sandbox.Console/Sandbox.Console.csproj
      tag: ${{ inputs.tag }}
      use-bot-token: false
      dry-run: ${{ inputs.dry-run }}
```

**Sample usage**

```yaml
name: Build-Release

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "tag: git tag you want create. (sample 1.0.0)"
        required: true
      dry-run:
        description: "dry_run: true will never create release/nuget."
        required: true
        default: false
        type: boolean

jobs:
  update-packagejson:
    permissions:
      actions: read
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/update-packagejson.yaml@master
    with:
      # you can write multi path.
      file-path: |
        ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo/package.json
        ./Sandbox/Sandbox.Godot/addons/Foo/plguin.cfg
        ./Sandbox/Directory.Build.props
      tag: ${{ inputs.tag }}
      use-bot-token: false # false to use GITHUB_TOKEN, true to use GitHub App token
      dry-run: ${{ inputs.dry-run }}

  build-unity:
    needs: [update-packagejson]
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ needs.update-packagejson.outputs.sha }}  # use updated package.json

  # use clean-packagejson-branch.yaml to delete dry-run branch.
  cleanup:
    if: ${{ needs.update-packagejson.outputs.is-branch-created == 'true' }}
    needs: [update-packagejson]
    permissions:
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/clean-packagejson-branch.yaml@master
    with:
      branch: ${{ needs.update-packagejson.outputs.branch-name }}
```

## validate-tag

> [See implementation](https://github.com/TeamMoirai/GitHubActions/blob/master/src/TeamMoiraiActions/Commands/ValidateTagCommand.cs)

Validate tag is newer than latest release tag.

Provided by the `validate-tag` subcommand of the `TeamMoiraiActions` CLI and called internally by the [create-release](#create-release) workflow. It is **not** a standalone reusable workflow.

**Command line usage**

```bash
dotnet run --project ./src/TeamMoiraiActions/TeamMoiraiActions.csproj --no-launch-profile -- validate-tag --tag "1.0.0" --require-validation
```

| Argument | Description |
| ---- | ---- |
| `--tag` | Git tag to validate. (sample `1.0.0`). A leading `v` is stripped. |
| `--require-validation` | When omitted the tag is only normalized, not compared. When set, exits 1 if the tag is older than the latest release. |

**Outputs**

| Name | Description |
| ---- | ---- |
| `tag` | The tag as supplied. |
| `normalized-tag` | The tag with any `v` prefix removed. |

Version comparison handles segment-wise numeric ordering (`1.0.9` vs `1.0.10`) and pre-release ordering (`alpha < beta < preview < rc < release`).

# 🎬 Actions

## benchmark-runnable

> [See action](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/actions/benchmark-runnable/action.yaml)

Check if GitHub User is allow to run benchmark.
Mainly used for benchmark CI workflow.

> [!NOTE]
> This action is workaround for current `github.event.comment.author_association` inconsistence behavior.
> `github.event.comment.author_association` should return `OWNER`, `MEMBER` or `CORABORATOR` for organization member, however currently it returns `CONTRIBUTOR` even actor is Org member.
> It means `github.event.comment.author_association` can't be used to check if actor is Org member == "benchmark command allowed user" or not.
> This action checks if actor is Benchmark allowed by the file list at `.github/benchmark-allowed-users.txt`.
> One username per line; blank lines and lines starting with `#` are ignored, and matching is case-insensitive.
> The list file requires `actions/checkout` first. When the file is missing the action **denies** (fail closed) and emits a warning.

**sample usage**

```yaml
name: benchmark

jobs:
  # is actor is benchmarkable
  verify:
    if: ${{ github.event_name == 'workflow_dispatch' || contains(github.event.comment.body, '/benchmark') }}
    outputs:
      is-benchmarkable: ${{ steps.is-benchmarkable.outputs.authorized }} # true or false
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      # the allowlist file requires the repository to be checked out
      - uses: actions/checkout@8e8c483db84b4bee98b60c0593521ed34d9990e8 # v6.0.1
      - name: Check actor is benchmarkable
        id: is-benchmarkable
        uses: TeamMoirai/GitHubActions/.github/actions/benchmark-runnable@master
        with:
          username: ${{ github.actor }}
          allowed-users-file: .github/benchmark-allowed-users.txt # default, can be omitted

  # run benchmark
  benchmark:
    needs: [verify]
    if: ${{ needs.verify.outputs.is-benchmarkable == 'true' }}
    environment: benchmark # required for Azure login
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - run: echo "run benchmark"
```

## check-metas

> [See action](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/actions/check-metas/action.yaml)

Check Unity .meta files are not generated.
Mainly used for Unity CI workflow.

**Sample usage**

```yaml
name: build-debug

on:
  push:
    branches:
      - main

jobs:
  build-unity:
    name: "Build Unity package"
    runs-on: ubuntu-24.04
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      # Any actions that create .meta when it was not comitted.
      - name: Unity Build
        run: touch ./Sandbox/Sandbox.Unity/Assets/Scene1.unity.meta
      - name: Check all .meta is comitted
        uses: TeamMoirai/GitHubActions/.github/actions/check-metas@master
        with:
          directory: ./Sandbox/Sandbox.Unity
```

## checkout

> [See action](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/actions/checkout/action.yaml)

Wrapper of [actions/checkout](https://github.com/actions/checkout/tree/main) to offer centrlral managed checkout by sha pinning.

**Sample usage**

```yaml
name: build-debug

on:
  push:
    branches:
      - main

jobs:
  build-unity:
    name: "Build Unity package"
    runs-on: ubuntu-24.04
    timeout-minutes: 15
    steps:
      # - uses: actions/checkout@v4
      - use: TeamMoirai/GitHubActions/.github/actions/checkout@master
      # Any actions that create .meta when it was not comitted.
      - name: Unity Build
        run: touch ./Sandbox/Sandbox.Unity/Assets/Scene1.unity.meta
      - name: Check all .meta is comitted
        uses: TeamMoirai/GitHubActions/.github/actions/check-metas@master
        with:
          directory: ./Sandbox/Sandbox.Unity
```


## download-artifact

> [See action](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/actions/download-artifact/action.yaml)

Wrapper of [actions/download-artifact](https://github.com/actions/download-artifact/tree/main) to offer default value and consistent action versioning. Mainly used for Release artifact.

> [!TIP]
> See [upload-artifact](#upload-artifact) for upload.

**Sample usage**

```yaml
name: build-debug

on:
  push:
    branches:
      - main

jobs:
  # must prepare upload-artifact

  download-artifact:
    needs: [upload-artifact]
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: TeamMoirai/GitHubActions/.github/actions/download-artifact@master
        with:
          name: my-artifact
      - name: Display structure of downloaded files
        run: ls -R
```


## setup-dotnet

> [See action](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/actions/setup-dotnet/action.yaml)

Wrapper of [actions/setup-dotnet](https://github.com/actions/setup-dotnet) to offer default value and consistent action versioning and Environment variables. Mainly used for .NET CI workflow.

**Sample usage**

```yaml
name: build-debug

on:
  push:
    branches:
      - main

jobs:
  dotnet-build:
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: TeamMoirai/GitHubActions/.github/actions/setup-dotnet@master
```

## unity-builder

> [See action](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/actions/unity-builder/action.yaml)

Build Unity projects for different platforms.

**Sample usage**

```yaml
name: build-debug

on:
  push:
    branches:
      - main

jobs:
  dotnet-build:
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      # execute scripts/Export Package
      # /opt/Unity/Editor/Unity -quit -batchmode -nographics -silent-crashes -logFile -projectPath . -executeMethod PackageExporter.Export
      - name: Build Unity (.unitypacakge)
        uses: TeamMoirai/GitHubActions/.github/actions/unity-builder@master
        with:
          projectPath: src/MyProject.Unity
          unityVersion: "2020.3.33f1"
          targetPlatform: StandaloneLinux64
          buildMethod: PackageExporter.Export
          versioning: None
```

## upload-artifact

> [See action](https://github.com/TeamMoirai/GitHubActions/blob/master/.github/actions/upload-artifact/action.yaml)

Wrapper of [actions/upload-artifact](https://github.com/actions/upload-artifact/tree/main) to offer default value and consistent action versioning. Mainly used for Release artifact.

> [!TIP]
> See [download-artifact](#download-artifact) for download.

**Sample usage**

```yaml
name: build-debug

on:
  push:
    branches:
      - main

jobs:
  upload-artifact:
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - run: mkdir -p path/to/artifact
      - run: echo hello > path/to/artifact/world.txt
      - uses: TeamMoirai/GitHubActions/.github/actions/upload-artifact@master
        with:
          name: my-artifact
          path: path/to/artifact/world.txt
```

# 🤝 Special Thanks

This project is based on **Cysharp**'s [Actions](https://github.com/Cysharp/Actions) with modifications.
