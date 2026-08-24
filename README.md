# secret-scanning-action

This action uses [gh-secure-kit](https://github.com/srz-zumix/gh-secure-kit) to run [local secret scanning](https://github.com/srz-zumix/gh-secure-kit#scan-local-git-content-for-secrets) in GitHub Actions.

## Features

- Scan local git content for secrets with `gh-secure-kit secret-scanning local check`
- Optionally install `pre-commit` and/or `pre-push` hooks with `gh-secure-kit secret-scanning local hook install`
- Automatically remove or restore hooks at the end of the job with `srz-zumix/post-run-action`
- Support hook-only setup by disabling the immediate scan
- Pass scan flags through the `args` input so you can choose staged, uncommitted, unpushed, or other supported scan targets

## Usage

### Scan staged changes

```yaml
name: Secret scan
on:
  pull_request:

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v5
      - uses: srz-zumix/secret-scanning-action@v0
        with:
          args: --staged
```

### Install pre-commit and pre-push hooks before scanning

```yaml
      - uses: srz-zumix/secret-scanning-action@v0
        with:
          hooks: |
            pre-commit
            pre-push
          args: --unpushed
```

### Install hooks only

```yaml
      - uses: srz-zumix/secret-scanning-action@v0
        with:
          hooks: pre-push
          run-scan: false
```

## Inputs

| Name | Description | Default | Required |
| ---- | ----------- | ------- | -------- |
| `args` | Additional arguments passed to `gh-secure-kit secret-scanning local check` | `''` | false |
| `hooks` | Optional hooks to install before scanning. Supported values: `pre-commit`, `pre-push` | `''` | false |
| `run-scan` | Run local secret scanning after optional hook installation | `true` | false |
| `working-directory` | Directory where the scan and optional hook installation run | `.` | false |
| `cache` | Whether or not to use aqua install cache | `true` | false |

## Notes

- The action runs `gh-secure-kit` directly from the release binary installed by `aqua`.
- Hooks installed by this action are cleaned up automatically at job teardown so the repository is returned to its previous state.
- Refer to the [gh-secure-kit README](https://github.com/srz-zumix/gh-secure-kit#scan-local-git-content-for-secrets) for supported local scan flags.
- Hook installation uses the repository configured in `working-directory`.
- Hook installation always runs with `--backup` and `--force`, and the backup files are removed during cleanup.
