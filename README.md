<p style="text-align: center;"><img src=".github/assets/nx.png" 
width="100%" alt="Nx - Smart, Extensible Build Framework"></p>

<h1 align="center">Nx Cloud Github Workflows</h2>

✨ Github Workflows for easily configuring distributed CI pipelines powered by the speed and intelligence of Nx Cloud.

- [Example Usage](#example-usage)
- [Configuration Options for the Main Job](#configuration-options-for-the-main-job-nx-cloud-mainyml)
- [Configuration Options for the Agent Jobs](#configuration-options-for-agent-jobs-nx-cloud-agentsyml)

## Example Usage

The following will configure a CI workflow which runs on the `main` branch, and on all pull requests into the `main` branch. The CI workflow will do the following:

- Checkout your repo at the appropriate depth for determining affected projects
- Determine the `head` and `base` SHAs to use for `nx affected`
- Install nodejs
- Use an appropriate node_module caching strategy for either `yarn` or `npm` (depending on which one is configured in your repo)
- Install dependencies with either `yarn` or `npm` (depending on which one is configured in your repo)
- Spawn 3 agents ready to receive tasks/targets to run
- Initiate all the provided commands in parallel, with those defined in `parallel-commands` being executed on the main job, and those in `parallel-commands-on-agents` being intelligently distributed across the 3 agents by Nx Cloud
- Shut down all agents when all parallel tasks have completed

**.github/workflows/ci.yml**

<!-- start example-usage -->

```yaml
name: CI

on:
  push:
    branches:
      - main
  pull_request:

concurrency:
  group: ${{ github.workflow }}-${{ github.event.number || github.ref }}
  cancel-in-progress: true

jobs:
  main:
    name: Nx Cloud - Main Job
    uses: jameshenry/ci/.github/workflows/nx-cloud-main.yml@0.1
    with:
      parallel-commands: |
        npx nx workspace-lint
        npx nx format:check
      parallel-commands-on-agents: |
        npx nx affected --target=lint --parallel=3
        npx nx affected --target=test --parallel=3 --ci --code-coverage
        npx nx affected --target=build --parallel=3

  agents:
    name: Nx Cloud - Agents
    uses: jameshenry/ci/.github/workflows/nx-cloud-agents.yml@0.1
    with:
      number-of-agents: 3
```

<!-- end example-usage -->

## Configuration Options for the Main Job (nx-cloud-main.yml)

<!-- start configuration-options-for-the-main-job -->

```yaml
- uses: jameshenry/ci/.github/workflows/nx-cloud-main.yml@0.1
  with:
    # [OPTIONAL] A multi-line string representing any bash commands (separated by new lines) which should
    # run sequentially, directly on the main job BEFORE executing any of the parallel commands which
    # may be specified via parallel-commands and/or parallel-commands-on-agents
    init-commands: |
      ""

    # [OPTIONAL] A multi-line string representing any bash commands (separated by new lines) which should
    # run in parallel, directly on the main job at the same time as any commands which may have beeen
    # specified via parallel-commands-on-agents
    #
    # NOTE: Due to how each stringified command gets interpreted in order to parallelize it, there may be
    # some limitations in terms of quote escaping and variable assignments within the command itself.
    parallel-commands: |
      ""

    # [OPTIONAL] A multi-line string representing any bash commands (separated by new lines) which should
    # run in parallel, preferentially distributed across any available agents at the same time as any
    # commands which may have been specified via parallel-commands
    #
    # NOTE: Due to how each stringified command gets interpreted in order to parallelize it, there may be
    # some limitations in terms of quote escaping and variable assignments within the command itself.
    parallel-commands-on-agents: |
      ""

    # [OPTIONAL] A multi-line string representing any bash commands (separated by new lines) which should
    # run sequentially, directly on the main job AFTER executing any of the parallel commands which
    # may be specified via parallel-commands and/or parallel-commands-on-agents
    final-commands: |
      ""

    # [OPTIONAL] The "main" branch of your repository (the base branch which you target with PRs).
    # Common names for this branch include main and master.
    #
    # Default: main
    main-branch-name: ""

    # [OPTIONAL] If you want to provide a specific node-version to use you can do that here.
    # If you do not specify one, it will respect an optional volta config you might have in
    # your package.json, otherwise it will simply install the latest LTS version of node.
    node-version: ""

    # [OPTIONAL] If you want to provide a specific npm-version to use you can do that here.
    # If you do not specify one, it will respect an optional volta config you might have in
    # your package.json, otherwise it will simply install the version of npm which is bundled
    # with the latest LTS version of node.
    npm-version: ""

    # [OPTIONAL] If you want to provide a specific yarn v1 version to use you can do that here.
    # If you do not specify one, it will respect an optional volta config you might have in
    # your package.json, otherwise it will simply install the latest v1 version of yarn available.
    yarn-version: ""

    # [OPTIONAL] If you want to provide a specific install command to use when installing dependencies
    # you can do that here. If you do not specify one, it will check for the presence of a yarn.lock
    # file in your repo, and if it finds one, run `yarn install --frozen-lockfile`. Otherwise it will
    # run `npm ci`.
    install-command: ""
```

<!-- end configuration-options-for-the-main-job -->

## Configuration Options for Agent Jobs (nx-cloud-agents.yml)

<!-- start configuration-options-for-agent-jobs -->

```yaml
- uses: jameshenry/ci/.github/workflows/nx-cloud-agents.yml@0.1
  with:
    # [REQUIRED] The number of agents which should be created as part of the workflow in order to
    # allow Nx Cloud to intelligently distribute tasks in parallel.
    number-of-agents: 3

    # [OPTIONAL] If you want to provide a specific node-version to use you can do that here.
    # If you do not specify one, it will respect an optional volta config you might have in
    # your package.json, otherwise it will simply install the latest LTS version of node.
    node-version: ""

    # [OPTIONAL] If you want to provide a specific npm-version to use you can do that here.
    # If you do not specify one, it will respect an optional volta config you might have in
    # your package.json, otherwise it will simply install the version of npm which is bundled
    # with the latest LTS version of node.
    npm-version: ""

    # [OPTIONAL] If you want to provide a specific yarn v1 version to use you can do that here.
    # If you do not specify one, it will respect an optional volta config you might have in
    # your package.json, otherwise it will simply install the latest v1 version of yarn available.
    yarn-version: ""

    # [OPTIONAL] If you want to provide a specific install command to use when installing dependencies
    # you can do that here. If you do not specify one, it will check for the presence of a yarn.lock
    # file in your repo, and if it finds one, run `yarn install --frozen-lockfile`. Otherwise it will
    # run `npm ci`.
    install-command: ""
```

<!-- end configuration-options-for-agent-jobs -->
