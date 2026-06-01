# Local CI testing with `act`

[`act`](https://github.com/nektos/act) runs GitHub Actions locally inside Docker. Use it to validate changes to these workflows before pushing to `v1`.

## Prerequisites

- Docker running
- `act` installed — see [installation](https://github.com/nektos/act?tab=readme-ov-file#installation)

## One-time setup

`act` prompts interactively for a Docker image on the first run. Pre-configure it to skip the prompt:

```bash
mkdir -p ~/.config/act
echo '-P ubuntu-latest=catthehacker/ubuntu:act-latest' >> ~/.config/act/actrc
```

## Running the CI

From the plugin directory, the command below redirects `uses: glpi-project/plugin-ci-workflows@v1` to your local checkout instead of fetching from GitHub.

Two flags are always required when running a `workflow_call`-based workflow with `act`:

- `-j ci` — the CI job depends on `generate-ci-matrix` for its matrix, which `act` cannot resolve dynamically; targeting the job directly bypasses that dependency
- `--env GITHUB_WORKSPACE=<plugins-dir>` — `${{ github.workspace }}` resolves incorrectly in container `options:` under `act` + `workflow_call`; this makes it point to the directory that contains the plugin folder

```bash
cd /path/to/plugins/myplugin

act push \
  --local-repository "glpi-project/plugin-ci-workflows@v1=/path/to/plugin-ci-workflows" \
  -W .github/workflows/continuous-integration.yml \
  -j ci \
  --matrix php-version:8.2 \
  --env GITHUB_WORKSPACE=/path/to/plugins \
  --pull=false \
  --artifact-server-path ~/.cache/act-artifacts
```

`--artifact-server-path` runs a local cache server pour les steps `actions/cache` — Composer et npm sont restaurés depuis le disque au lieu du réseau. Créer le répertoire une fois avant le premier run :

```bash
mkdir -p ~/.cache/act-artifacts
```

Adapt three paths to your setup:

| Placeholder | What it points to | Example |
|---|---|---|
| `/path/to/plugins/myplugin` | The plugin under test | `~/dev/GLPI/11.0-bf/plugins/fields` |
| `/path/to/plugin-ci-workflows` | Your local checkout of this repo | `~/dev/plugin-ci-workflows` |
| `/path/to/plugins` | The directory **containing** the plugin | `~/dev/GLPI/11.0-bf/plugins` |

### First run

The first run pulls the GLPI Docker image from `ghcr.io/glpi-project/githubactions-glpi-apache` (several GB). Drop `--pull=false` for that first run only, then add it back to reuse the cached image.

## Simulating a pull request event

Some steps only run on pull requests (CHANGELOG check, XML URL check). Pass a minimal event payload to trigger them:

```bash
cd /path/to/plugins/myplugin

act pull_request \
  --local-repository "glpi-project/plugin-ci-workflows@v1=/path/to/plugin-ci-workflows" \
  -W .github/workflows/continuous-integration.yml \
  -j ci \
  --matrix php-version:8.2 \
  --env GITHUB_WORKSPACE=/path/to/plugins \
  --pull=false \
  --artifact-server-path ~/.cache/act-artifacts \
  --eventpath <(echo '{"pull_request":{"number":1},"repository":{"full_name":"glpi-project/myplugin"}}')
```
