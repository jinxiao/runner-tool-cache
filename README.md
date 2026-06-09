# setup-uv tool-cache release

This repository builds a `setup-uv` runner tool-cache archive and publishes it as a GitHub Release asset.

Before installing uv, the workflow requests the GitHub API for the latest `astral-sh/uv` releases, selects the latest three non-draft and non-prerelease versions, and replaces the existing `${{ runner.tool_cache }}` directory on the publishing runner. The release asset therefore contains only those three uv versions.

Run the workflow manually:

1. Open **Actions**.
2. Run **Release setup-uv tool cache**.
3. Leave `release-tag` empty to use `setup-uv-tool-cache-uv-<oldest>-<newest>`.

The generated asset is named like:

```text
setup-uv-tool-cache-Linux-X64-uv-0.11.19-0.11.18-0.11.17.tar.gz
```

The archive is created with `${{ runner.tool_cache }}` as the working directory, so it contains the full contents of the runner tool-cache directory after the uv setup steps have populated it.

## Restore on a GHES private runner

Restore the archive before running `astral-sh/setup-uv`:

```yaml
- name: Restore uv tool cache from release
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    CACHE_REPO: owner/repo
    CACHE_TAG: setup-uv-tool-cache-uv-0.11.17-0.11.19
    CACHE_ASSET: setup-uv-tool-cache-Linux-X64-uv-0.11.19-0.11.18-0.11.17.tar.gz
  shell: bash
  run: |
    set -euo pipefail
    mkdir -p "${RUNNER_TOOL_CACHE}" /tmp/setup-uv-tool-cache
    gh release download "${CACHE_TAG}" \
      --repo "${CACHE_REPO}" \
      --pattern "${CACHE_ASSET}" \
      --dir /tmp/setup-uv-tool-cache
    tar -C "${RUNNER_TOOL_CACHE}" -xzf "/tmp/setup-uv-tool-cache/${CACHE_ASSET}"

- name: Install uv
  uses: astral-sh/setup-uv@v8.2.0
  with:
    version: "0.11.19"
    enable-cache: false
    ignore-empty-workdir: true
```

The archive must be extracted directly into `RUNNER_TOOL_CACHE`.
