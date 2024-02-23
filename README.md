![version](https://img.shields.io/badge/version-20%2B-E23089)
![platform](https://img.shields.io/static/v1?label=platform&message=mac-intel%20|%20mac-arm%20|%20win-64&color=blue)
[![license](https://img.shields.io/github/license/miyako/4d-topic-cicd)](LICENSE)
![downloads](https://img.shields.io/github/downloads/miyako/4d-topic-cicd/total)

A CI/CD template project.

# Points of Interest

A [Run Tests](https://github.com/miyako/4d-topic-cicd/blob/main/.github/workflows/run-tests.yml) workflow is automatically triggered when code is changed in the `main` branch. 

This workflow does the following:

1. Launch 2 GitHub hosted runners: `windows-latest` `macos-latest`
2. Checkout the current repository
3. Checkout the latest [`compiler`](https://github.com/miyako/4d-class-compiler) project from releases
4. Download [`tool4d`](https://developer.4d.com/docs/Admin/cli/#using-tool4d)
5. Run a specific [`--startup-method`](https://developer.4d.com/docs/Admin/cli/#launch-a-4d-application) with `tool4d` and `compiler` project 

Sample result: https://github.com/miyako/4d-topic-cicd/actions/runs/8025901243

Only changes relevant to code execution will trigger the workflow:

```yml
on:
  push:
    branches:
    - main
    paths:
      - '*/Project/Sources/**/*.4dm'
      - '*/Project/Sources/*/*.4DForm'
      - '*/Project/Sources/*.4DCatalog' 
```

A specific branch, product and build of `tool4d` is used:

```yml
jobs:     

  get:
    strategy:
      fail-fast: false
      matrix:
        TOOL4D_PLATFORM: ["windows-latest", "macos-latest"]
        TOOL4D_BRANCH: [20.x]
        TOOL4D_VERSION: [20.2]
        TOOL4D_BUILD: [latest] 
    runs-on: ${{ matrix.TOOL4D_PLATFORM }}
    steps:

      - name: checkout 
        uses: actions/checkout@v4
    
      - name: get tool4d
        id: get
        uses: miyako/4D/.github/actions/get-tool@v1
        with:
          platform: ${{ matrix.TOOL4D_PLATFORM }}
          branch: ${{ matrix.TOOL4D_BRANCH }}
          version: ${{ matrix.TOOL4D_VERSION }}
          build: ${{ matrix.TOOL4D_BUILD }}
```

Any combination of runners, `tool4d`, projects, `--startup-method` can be specified in a strategy matrix:

```yml
jobs:     

  get:
    strategy:
      fail-fast: false
      matrix:
        TOOL4D_BUILD: [latest] 
        TOOL4D_STARTUP_METHOD: [test] 
        TOOL4D_STARTUP_PROJECT_PATH: [./application/Project/application.4DProject] 
    runs-on: ${{ matrix.TOOL4D_PLATFORM }}
    steps:

      - name: run tests
        uses: ./.github/actions/run-tests
        with:
          tool4d_download_url: ${{ steps.get.outputs.tool4d_download_url }}
          tool4d_executable_path: ${{ steps.get.outputs.tool4d_executable_path }}
          startup_method: ${{ matrix.TOOL4D_STARTUP_METHOD }}
          project_path: ${{ matrix.TOOL4D_STARTUP_PROJECT_PATH }}        
```

A [Publish](https://github.com/miyako/4d-topic-cicd/blob/main/.github/workflows/publish.yml) workflow can be trigged manually.

This workflow does the following:

1. Prompt to choose the kind of version bump: `patch`, `minor`, `major`
2. Update `package.json` at the root of the project (the version information in this file is incorporated in the build process)
3. Create a release that corresponds to the new version
4. Connect to a self-hosted runner (build must always take place on a self-hosted runner with licenses installed)
5. (skip units tests, which would have been done already on GitHub hosted runners, as described above)
6. Checkout the current repository
7. Checkout the latest [`compiler`](https://github.com/miyako/4d-class-compiler) project from releases
8. Download [`tool4d`](https://developer.4d.com/docs/Admin/cli/#using-tool4d)
9. Build, sign for distribtuion (Develooper ID Application), archive, notarise, staple the product
10. Uploaded .dmg to the release created earlier

---

# workflows

## [Test](https://github.com/miyako/4d-topic-cicd/blob/main/.github/workflows/test.yml)

* trigger: any time there is a push to the `main` branch, or manually
* runners: github hosted `macos-latest` and/or `windows-latest`
* always use the latest `tool4d`

## Build 

* trigger: any time there is a push to the `main` branch, or manually
* runners: self-hosted, `macOS`
* when triggered automatically, sign to run locally for testing
* when triggered manually, sign for distribtuion, notarise, staple

### 資料

* [セルフホステッド ランナーの概要](https://docs.github.com/ja/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)
* [セルフホステッド ランナーを追加する](https://docs.github.com/ja/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners)
* [セルフホストランナーアプリケーションをサービスとして設定する](https://docs.github.com/ja/actions/hosting-your-own-runners/managing-self-hosted-runners/configuring-the-self-hosted-runner-application-as-a-service?platform=mac)

* [A Tool for 4D Code Execution in CLI](https://blog.4d.com/a-tool-for-4d-code-execution-in-cli/)

* [Workflow syntax for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idif)
* [Automatic token authentication](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)
* [Contexts](https://docs.github.com/en/actions/learn-github-actions/contexts)
* [Variables](https://docs.github.com/en/actions/learn-github-actions/variables)
* [Expressions](https://docs.github.com/en/actions/learn-github-actions/expressions)
* [Workflow commands for GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#environment-files)
