# GitHub Actions

> OSS 仓库使用的 GitHub Actions "可复用工作流" 和 "复合操作"。
>

[![actions lint](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_actions-lint.yaml/badge.svg)](https://github.com/TeamMoirai/GitHubActions/actions/workflows/_actions-lint.yaml)
[![English](https://img.shields.io/badge/README-English-blue)](https://github.com/TeamMoirai/GitHubActions/blob/main/README_EN.md)

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

- [🔑 必需密钥](#-%E5%BF%85%E9%9C%80%E5%AF%86%E9%92%A5)
- [♻️ 可复用工作流](#-%E5%8F%AF%E5%A4%8D%E7%94%A8%E5%B7%A5%E4%BD%9C%E6%B5%81)
  - [clean-packagejson-branch](#clean-packagejson-branch)
  - [create-release](#create-release)
  - [dd-event-post](#dd-event-post)
  - [increment-version](#increment-version)
  - [prevent-github-change](#prevent-github-change)
  - [stale-issue](#stale-issue)
  - [update-packagejson](#update-packagejson)
  - [validate-tag](#validate-tag)
- [🎬 复合操作](#-%E5%A4%8D%E5%90%88%E6%93%8D%E4%BD%9C)
  - [check-benchmarkable](#check-benchmarkable)
  - [check-metas](#check-metas)
  - [checkout](#checkout)
  - [download-artifact](#download-artifact)
  - [setup-dotnet](#setup-dotnet)
  - [unity-builder](#unity-builder)
  - [upload-artifact](#upload-artifact)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 🔑 必需密钥

部分工作流需要在仓库设置中配置 GitHub Secrets。请前往 **Settings → Secrets and variables → Actions** 进行添加。

| 密钥名称 | 使用方 | 说明 |
| ---- | ---- | ---- |
| `NUGET_KEY` | `create-release` | NuGet.org 的 API 密钥，用于推送包 |
| `DD_API_KEY` | `dd-event-post` | Datadog 的 API 密钥，用于发送事件 |
| `ACTIONBOT_APPID` | `update-packagejson` | GitHub App ID，用于机器人令牌认证 |
| `ACTIONBOT_PRIVATE_KEY` | `update-packagejson` | GitHub App 私钥，用于机器人令牌认证 |
| `AZURE_OIDC_CLIENTID` | `benchmark-cleanup`、`benchmark-execute` | Azure AD 应用程序客户端 ID（OIDC） |
| `AZURE_OIDC_TENANTID` | `benchmark-cleanup`、`benchmark-execute` | Azure AD 租户 ID（OIDC） |
| `AZURE_OIDC_SUBSCRIPTIONID` | `benchmark-cleanup`、`benchmark-execute` | Azure 订阅 ID（OIDC） |
| `BENCHMARK_VM_SSH_KEY` | `benchmark-execute` | 用于连接基准测试虚拟机的 SSH 私钥 |

> [!NOTE]
> 密钥仅在使用对应的工作流功能时才需要。例如，`NUGET_KEY` 仅在 `create-release` 传入 `nuget-push: true` 时才需要。

> [!TIP]
> 调用可复用工作流时，使用 `secrets: inherit` 将仓库密钥传递给被调用的工作流。

# ♻️ 可复用工作流

## clean-packagejson-branch

> [查看工作流](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/workflows/clean-packagejson-branch.yaml)

删除指定的 GitHub 分支。主要用于清理由 [update-packagejson](#update-packagejson) 工作流创建的分支。该操作有以下限制以防止误删：

1. 分支不是默认分支。
2. 分支由 github-actions[bot] 创建并提交。

**使用示例**

```yaml
name: Build-Release

on:
  workflow_dispatch:

jobs:
  cleanup:
    permissions:
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/clean-packagejson-branch.yaml@main
    with:
      branch: branch_name_to_delete
```

## create-release

> [查看工作流](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/workflows/create-release.yaml)

创建 GitHub Release，上传 NuGet 包和发布资产。主要用于 NuGet 和 Unity 的发布工作流。

必需密钥。

| 密钥名称 | 使用条件 | 说明 |
| ---- | ---- | ---- |
| `NUGET_KEY` | `with.nuget-push` 为 true | 推送 nupkg、snupkg 到 NuGet.org 所需的密钥 |

**使用示例**

仅创建 Release。

```yaml
name: Build-Release

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "tag: 要创建的 git tag（示例 1.0.0）"
        required: true
      dry-run:
        description: "dry_run: true 则不会创建 release/nuget"
        required: true
        default: false
        type: boolean

jobs:
  create-release:
    uses: TeamMoirai/GitHubActions/.github/workflows/create-release.yaml@main
    with:
      commit-id: ''
      tag: ${{ inputs.tag }}
      dry-run: ${{ inputs.dry-run }} # 为 true 时，Release 创建后 60 秒删除 tag
      nuget-push: false
      release-upload: false
    secrets: inherit
```

修改 Release 名称，不使用 `Ver.` 前缀。

```yaml
name: Build-Release

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "tag: 要创建的 git tag（示例 1.0.0）"
        required: true
      dry-run:
        description: "dry_run: true 则不会创建 release/nuget"
        required: true
        default: false
        type: boolean

jobs:
  create-release:
    uses: TeamMoirai/GitHubActions/.github/workflows/create-release.yaml@main
    with:
      commit-id: ''
      tag: ${{ inputs.tag }}
      dry-run: ${{ inputs.dry-run }}
      nuget-push: false
      release-upload: false
      release-format: '{0}'
    secrets: inherit
```

下载其他工作流的产物并上传到 Release 资产。

```yaml
name: Build-Release

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "tag: 要创建的 git tag（示例 1.0.0）"
        required: true
      dry-run:
        description: "dry_run: true 则不会创建 release/nuget"
        required: true
        default: false
        type: boolean

jobs:
  create-release:
    uses: TeamMoirai/GitHubActions/.github/workflows/create-release.yaml@main
    with:
      commit-id: ''
      tag: ${{ inputs.tag }}
      dry-run: ${{ inputs.dry-run }}
      nuget-push: false
      release-upload: true
      release-asset-path: |
        ./FooBar/win-amd64/FooBar.pdb
        ./FooBar/win-arm64/FooBar.pdb
      download-run-id: '123456789' # 指定要下载产物的运行 ID
    secrets: inherit
```

构建 .NET 然后创建 Release。`create-release` 将推送 NuGet 包。

```yaml
name: Build-Release

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "tag: 要创建的 git tag（示例 1.0.0）"
        required: true
      dry-run:
        description: "dry_run: true 则不会创建 release/nuget"
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
      - uses: TeamMoirai/GitHubActions/.github/actions/setup-dotnet@main
      - run: dotnet build -c Release -p:Version=${{ inputs.tag }}
      - run: dotnet pack --no-build -c Release -p:Version=${{ inputs.tag }} -p:IncludeSymbols=true -p:SymbolPackageFormat=snupkg -o ./publish
      - name: upload artifacts
        uses: TeamMoirai/GitHubActions/.github/actionsupload-artifact@main
        with:
          name: nuget
          path: ./Sandbox/publish
          retention-days: 1

  create-release:
    needs: [build-dotnet]
    uses: TeamMoirai/GitHubActions/.github/workflows/create-release.yaml@main
    with:
      commit-id: ''
      tag: ${{ inputs.tag }}
      dry-run: ${{ inputs.dry-run }}
      nuget-push: true
      release-upload: false
    secrets: inherit                 # 允许工作流访问 NUGET_KEY 密钥
```

构建 .NET 和 Unity，然后创建 Release。`create-release` 将推送 NuGet 包并上传 unitypackage 到 Release 资产。

```yaml
name: Build-Release

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "tag: 要创建的 git tag（示例 1.0.0）"
        required: true
      dry-run:
        description: "dry_run: true 则不会创建 release/nuget"
        required: true
        default: false
        type: boolean

jobs:
  update-packagejson:
    if: ${{ github.actor != 'dependabot[bot]' }}
    permissions:
      actions: read
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/update-packagejson.yaml@main
    with:
      file-path: |
        ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo/package.json
        ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo.Plugin/package.json
        ./Sandbox/Sandbox.Godot/addons/Foo/plugin.cfg
        ./Sandbox/Directory.Build.props
      tag: ${{ inputs.tag }}
      use-bot-token: false # false 使用 GITHUB_TOKEN，true 使用 GitHub App 令牌
      dry-run: false

  build-dotnet:
    runs-on: ubuntu-24.04
    timeout-minutes: 3
    defaults:
      run:
        working-directory: ./Sandbox
    steps:
      - uses: actions/checkout@v4
      - uses: TeamMoirai/GitHubActions/.github/actions/setup-dotnet@main
      - run: dotnet build -c Release -p:Version=${{ inputs.tag }}
      - run: dotnet pack --no-build -c Release -p:Version=${{ inputs.tag }} -p:IncludeSymbols=true -p:SymbolPackageFormat=snupkg -o ./publish
      - name: upload artifacts
        uses: TeamMoirai/GitHubActions/.github/actionsupload-artifact@main
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
      # 存储产物
      - uses: TeamMoirai/GitHubActions/.github/actions/upload-artifact@main
        with:
          name: Sandbox.Unity.unitypackage
          path: ./Sandbox/Sandbox.Unity/output/Sandbox.Unity.unitypackage
          if-no-files-found: error
      - uses: TeamMoirai/GitHubActions/.github/actions/upload-artifact@main
        with:
          name: Sandbox.Unity.Plugin.unitypackage
          path: ./Sandbox/Sandbox.Unity/output/Sandbox.Unity.Plugin.unitypackage
          if-no-files-found: error

  create-release:
    needs: [update-packagejson, build-dotnet, build-unity]
    uses: TeamMoirai/GitHubActions/.github/workflows/create-release.yaml@main
    with:
      commit-id: ${{ needs.update-packagejson.outputs.sha }}
      tag: ${{ inputs.tag }}
      dry-run: ${{ inputs.dry-run }}
      nuget-push: true
      release-upload: true
      release-asset-path: |
        ./Sandbox.Unity.unitypackage/Sandbox.Unity.unitypackage
        ./Sandbox.Unity.Plugin.unitypackage/Sandbox.Unity.Plugin.unitypackage
        ./nuget/ClassLibrary.${{ inputs.tag }}.nupkg
        ./nuget/ClassLibrary.${{ inputs.tag }}.snupkg
    secrets: inherit                 # 允许工作流访问 NUGET_KEY 密钥

  cleanup:
    if: ${{ needs.update-packagejson.outputs.is-branch-created == 'true' }}
    needs: [update-packagejson]
    permissions:
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/clean-packagejson-branch.yaml@main
    with:
      branch: ${{ needs.update-packagejson.outputs.branch-name }}
```


## dd-event-post

> [查看工作流](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/workflows/dd-event-post.yaml)

发送 Datadog 事件。

必需密钥。

| 密钥名称 | 说明 |
| ---- | ---- |
| `DD_API_KEY` | Datadog 的 API 密钥，用于发送事件 |

1. 用于 Pull Request 合并事件。

**使用示例**

```yaml
name: PR Merged

on:
  pull_request:
    types: [closed]

jobs:
  post:
    if: ${{ github.event.pull_request.merged == true }}
    uses: TeamMoirai/GitHubActions/.github/workflows/dd-event-post.yaml@main
    secrets: inherit
```

## increment-version

> [查看工作流](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/workflows/increment-version.yaml)

使用递增版本号更新指定的版本文件。主要用于[发布后工作流](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/workflows/post-release.yaml)。

**使用示例**

以下工作流将从已发布的 tag 递增补丁版本号，并使用带 `-dev` 后缀的新版本更新指定的 package.json、plugin.cfg 和 Directory.Build.props 文件。

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
    uses: TeamMoirai/GitHubActions/.github/workflows/increment-version.yaml@main
    with:
      ref: ${{ github.event.repository.default_branch }}
      tag: ${{ github.ref_name }} # tag 值将在此处。1.2.1
      type: patch
      suffix: "-dev"

  update-packagejson:
    needs: [new-version]
    permissions:
      actions: read
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/update-packagejson.yaml@main
    with:
      ref: ${{ github.event.repository.default_branch }}
      file-path: |
        ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo/package.json
        ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo.Plugin/package.json
        ./Sandbox/Sandbox.Godot/addons/Foo/plugin.cfg
        ./Sandbox/Directory.Build.props
      tag: ${{ needs.new-version.outputs.version }}
      use-bot-token: false # false 使用 GITHUB_TOKEN，true 使用 GitHub App 令牌
      dry-run: false

```

## prevent-github-change

> [查看工作流](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/workflows/prevent-github-change.yaml)

防止 fork 用户修改触发的文件。只有组织贡献者才能修改这些文件。

**使用示例**

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
    uses: TeamMoirai/GitHubActions/.github/workflows/prevent-github-change.yaml@main
```


## stale-issue

> [查看工作流](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/workflows/stale-issue.yaml)

标记过期的 Issue 和 PR。
主要用于 Issue/PR 管理。

**使用示例**


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
    uses: TeamMoirai/GitHubActions/.github/workflows/stale-issue.yaml@main
```

## update-packagejson

> [查看工作流](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/workflows/update-packagejson.yaml)

使用 tag 版本号更新指定的 `Unity package.json` 和 `Godot plugin.cfg`。主要用于 UPM 和 Godot 插件发布工作流。

必需密钥（当 `use-bot-token` 为 true 时）。

| 密钥名称 | 说明 |
| ---- | ---- |
| `ACTIONBOT_APPID` | GitHub App ID，用于机器人令牌认证 |
| `ACTIONBOT_PRIVATE_KEY` | GitHub App 私钥，用于机器人令牌认证 |

当仓库需要限制对默认分支的直接推送时，建议使用 GitHub App 令牌。

```yaml
jobs:
  use-bot-token:
    permissions:
      actions: read
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/update-packagejson.yaml@main
    with:
      file-path: ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo/package.json
      tag: ${{ inputs.tag }}
      use-bot-token: true # <-- 使用 GitHub App 令牌
      dry-run: ${{ inputs.dry-run }}
```

如果需要在工作流中运行 dotnet run，可以指定 `dotnet-run-path` 输入参数。请确保参数始终为 `--version "{tag}"`。

```yaml
jobs:
  update-packagejson:
    permissions:
      actions: read
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/update-packagejson.yaml@main
    with:
      file-path: ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo/package.json
      # 可以写多个路径
      dotnet-run-path: |
        ./Sandbox/Sandbox.Console/Sandbox.Console.csproj
      tag: ${{ inputs.tag }}
      use-bot-token: false
      dry-run: ${{ inputs.dry-run }}
```

**使用示例**

```yaml
name: Build-Release

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "tag: 要创建的 git tag（示例 1.0.0）"
        required: true
      dry-run:
        description: "dry_run: true 则不会创建 release/nuget"
        required: true
        default: false
        type: boolean

jobs:
  update-packagejson:
    permissions:
      actions: read
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/update-packagejson.yaml@main
    with:
      # 可以写多个路径
      file-path: |
        ./Sandbox/Sandbox.Unity/Assets/Plugins/Foo/package.json
        ./Sandbox/Sandbox.Godot/addons/Foo/plguin.cfg
        ./Sandbox/Directory.Build.props
      tag: ${{ inputs.tag }}
      use-bot-token: false # false 使用 GITHUB_TOKEN，true 使用 GitHub App 令牌
      dry-run: ${{ inputs.dry-run }}

  build-unity:
    needs: [update-packagejson]
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ needs.update-packagejson.outputs.sha }}  # 使用更新后的 package.json

  # 使用 clean-packagejson-branch.yaml 删除 dry-run 分支
  cleanup:
    if: ${{ needs.update-packagejson.outputs.is-branch-created == 'true' }}
    needs: [update-packagejson]
    permissions:
      contents: write
    uses: TeamMoirai/GitHubActions/.github/workflows/clean-packagejson-branch.yaml@main
    with:
      branch: ${{ needs.update-packagejson.outputs.branch-name }}
```

## validate-tag

> [查看工作流](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/workflows/validate-tag.yaml)

验证 tag 是否比最新的 release tag 更新。

**使用示例**

```yaml
name: "Validate release tag"

on:
  workflow_dispatch:
    inputs:
      tag:
        description: "tag: 要创建的 git tag（示例 1.0.0）"
        required: true
      require-validation:
        description: "require-validation: true 表示验证必须通过，false 表示即使验证失败也继续执行"
        required: false
        type: boolean
        default: true

jobs:
  validate:
    uses: TeamMoirai/GitHubActions/.github/workflows/validate-tag.yaml@main
    with:
      tag: ${{ inputs.tag }}
      require-validation: ${{ inputs.require-validation }} # true = tag 比当前 release 旧则退出 1。false = 即使失败也继续。

  test:
    needs: [validate]
    runs-on: ubuntu-24.04
    steps:
      - run: echo "${{ needs.validate.outputs.validated }}" # true 或 false

```

# 🎬 复合操作

## check-benchmarkable

> [查看操作](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/actions/check-benchmarkable/action.yaml)

检查 GitHub 用户是否被允许运行基准测试。
主要用于基准测试 CI 工作流。

> [!NOTE]
> 此操作是针对当前 `github.event.comment.author_association` 不一致行为的变通方案。
> `github.event.comment.author_association` 对于组织成员应该返回 `OWNER`、`MEMBER` 或 `COLLABORATOR`，但目前即使 actor 是组织成员也会返回 `CONTRIBUTOR`。
> 这意味着 `github.event.comment.author_association` 无法用于检查 actor 是否为组织成员（即"基准测试命令允许的用户"）。
> 此操作通过静态定义的列表检查 actor 是否被允许运行基准测试。

**使用示例**

```yaml
name: benchmark

jobs:
  # 检查 actor 是否可以运行基准测试
  verify:
    if: ${{ github.event_name == 'workflow_dispatch' || contains(github.event.comment.body, '/benchmark') }}
    outputs:
      is-benchmarkable: ${{ steps.is-benchmarkable.outputs.authorized }} # true 或 false
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - name: Check actor is benchmarkable
        id: is-benchmarkable
        uses: TeamMoirai/GitHubActions/.github/actions/check-benchmarkable@main
        with:
          username: ${{ github.actor }}

  # 运行基准测试
  benchmark:
    needs: [verify]
    if: ${{ needs.verify.outputs.is-benchmarkable == 'true' }}
    environment: benchmark # Azure 登录所需
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - run: echo "run benchmark"
```

## check-metas

> [查看操作](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/actions/check-metas/action.yaml)

检查 Unity 的 .meta 文件是否未被生成。
主要用于 Unity CI 工作流。

**使用示例**

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
      # 任何在未提交时创建 .meta 的操作
      - name: Unity Build
        run: touch ./Sandbox/Sandbox.Unity/Assets/Scene1.unity.meta
      - name: Check all .meta is comitted
        uses: TeamMoirai/GitHubActions/.github/actions/check-metas@main
        with:
          directory: ./Sandbox/Sandbox.Unity
```

## checkout

> [查看操作](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/actions/checkout/action.yaml)

[actions/checkout](https://github.com/actions/checkout/tree/main) 的封装，通过 SHA 固定提供集中管理的 checkout。

**使用示例**

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
      - use: TeamMoirai/GitHubActions/.github/actions/checkout@main
      # 任何在未提交时创建 .meta 的操作
      - name: Unity Build
        run: touch ./Sandbox/Sandbox.Unity/Assets/Scene1.unity.meta
      - name: Check all .meta is comitted
        uses: TeamMoirai/GitHubActions/.github/actions/check-metas@main
        with:
          directory: ./Sandbox/Sandbox.Unity
```


## download-artifact

> [查看操作](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/actions/download-artifact/action.yaml)

[actions/download-artifact](https://github.com/actions/download-artifact/tree/main) 的封装，提供默认值和一致的操作版本管理。主要用于发布产物。

> [!TIP]
> 上传请参见 [upload-artifact](#upload-artifact)。

**使用示例**

```yaml
name: build-debug

on:
  push:
    branches:
      - main

jobs:
  # 必须先准备 upload-artifact

  download-artifact:
    needs: [upload-artifact]
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: TeamMoirai/GitHubActions/.github/actions/download-artifact@main
        with:
          name: my-artifact
      - name: Display structure of downloaded files
        run: ls -R
```


## setup-dotnet

> [查看操作](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/actions/setup-dotnet/action.yaml)

[actions/setup-dotnet](https://github.com/actions/setup-dotnet) 的封装，提供默认值、一致的操作版本管理和环境变量。主要用于 .NET CI 工作流。

**使用示例**

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
      - uses: TeamMoirai/GitHubActions/.github/actions/setup-dotnet@main
```

## unity-builder

> [查看操作](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/actions/unity-builder/action.yaml)

为不同平台构建 Unity 项目。

**使用示例**

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
      # 执行 scripts/Export Package
      # /opt/Unity/Editor/Unity -quit -batchmode -nographics -silent-crashes -logFile -projectPath . -executeMethod PackageExporter.Export
      - name: Build Unity (.unitypacakge)
        uses: TeamMoirai/GitHubActions/.github/actions/unity-builder@main
        with:
          projectPath: src/MyProject.Unity
          unityVersion: "2020.3.33f1"
          targetPlatform: StandaloneLinux64
          buildMethod: PackageExporter.Export
          versioning: None
```

## upload-artifact

> [查看操作](https://github.com/TeamMoirai/GitHubActions/blob/main/.github/actions/upload-artifact/action.yaml)

[actions/upload-artifact](https://github.com/actions/upload-artifact/tree/main) 的封装，提供默认值和一致的操作版本管理。主要用于发布产物。

> [!TIP]
> 下载请参见 [download-artifact](#download-artifact)。

**使用示例**

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
      - uses: TeamMoirai/GitHubActions/.github/actions/upload-artifact@main
        with:
          name: my-artifact
          path: path/to/artifact/world.txt
```

# 🤝 特别鸣谢

该项目基于 **Cysharp** 的 [Actions](https://github.com/Cysharp/Actions) 修改。

