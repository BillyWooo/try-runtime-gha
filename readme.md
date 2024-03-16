# Try Runtime GHA

Try Runtime GHA makes it easy to add [try-runtime](https://paritytech.github.io/try-runtime-cli/try_runtime/) checks to your CI.

## Motivation

`try-runtime` enforces critical invariants such as:

- Migrations execute correctly, including `pre_upgrade` and `post_upgrade` hooks
- Migration execution does not exceed the block weight limit
- Migrations are idempotent
- Try State invariants pass
- Post-Migration, all runtime state is decodable

Given that code changes can break any of the above invariants, running `try-runtime` checks is as important as regularly running cargo tests in your CI to prevent accidental regressions.

## Usage

Create a new [Github Action Workflow](https://docs.github.com/en/actions/using-workflows) in the repo containing your runtime(s).

The simplest usage only requires two parameters `runtime-package`, and `node-uri`:

```yaml
name: Check Migrations

on:
  push:
    branches: ["main"]
  pull_request:

jobs:
  check-migrations:
    steps:
      - name: Checkout sources
        uses: actions/checkout@v3

      - name: Run Try Runtime Checks
        uses: "paritytech/try-runtime-gha@v0.1.0"
        with:
          runtime-package: "my-runtime"        # The runtime package name
          node-uri: "wss://my-runtime-uri:443" # Extra args to pass to the ''on-runtime-upgrade'' subcommand. For example, "--disable-idempotency-check --no-weight-warnings". See all options at https://paritytech.github.io/try-runtime-cli/try_runtime_core/commands/on_runtime_upgrade/struct.Command.html.
          checks: "all"                        # OPTIONAL: Types of checks to run. For example, "pre-and-post". See all options at https://paritytech.github.io/try-runtime-cli/frame_support/traits/enum.UpgradeCheckSelect.html. Default: "all".
          extra-args: "--arg --other-arg"      # OPTIONAL: Extra args to pass to the 'on-runtime-upgrade' subcommand. For example, "--disable-idempotency-check --no-weight-warnings". See all options at https://paritytech.github.io/try-runtime-cli/try_runtime_core/commands/on_runtime_upgrade/struct.Command.html.
```

When triggered, the GHA will

1. Build the `runtime-package` with flags `--production` and `--features try-runtime`
2. Download [try-runtime-cli](https://github.com/paritytech/try-runtime-cli)
3. Exec `try-runtime on-runtime-upgrade --checks [checks] [extra-args] live [node-uri]`

If there is an error, you can check the logs for the exact command that failed to help reproduce the issue in your local environment.

See more examples at [`.github/workflows/examples.yml`](https://github.com/paritytech/try-runtime-gha/blob/main/.github/workflows/examples.yml).
