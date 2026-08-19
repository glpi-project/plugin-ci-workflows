# Plugin CI workflows

## Continuous integration workflow

This workflow will execute the following actions as long as they are available on the plugin repository.
If the vendor package is not installed within the plugin repository, the CI workflow will attempt to use the vendor binaries provided by GLPI directly.

| Action                | Vendor package                                                                                            | Required config file                                                               |
|-----------------------|-----------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| PHP Parallel Lint     | [php-parallel-lint/php-parallel-lint](https://packagist.org/packages/php-parallel-lint/php-parallel-lint) |                                                                                    |
| PHP CodeSniffer       | [squizlabs/php_codesniffer](https://packagist.org/packages/squizlabs/php_codesniffer)                     | `.phpcs.xml`                                                                       |
| PHP-CS-Fixer          | [friendsofphp/php-cs-fixer](https://packagist.org/packages/friendsofphp/php-cs-fixer)                     | `.php-cs-fixer.php`                                                                |
| PHPStan               | [phpstan/phpstan](https://packagist.org/packages/phpstan/phpstan)                                         | `phpstan.neon`                                                                     |
| Psalm                 | [vimeo/psalm](https://packagist.org/packages/vimeo/psalm)                                                 | `psalm.xml`                                                                        |
| Rector                | [rector/rector](https://packagist.org/packages/rector/rector)                                             | `rector.php`                                                                       |
| ESLint                | [eslint](https://www.npmjs.com/package/eslint)                                                            | `eslint.config.js` or `eslint.config.mjs` or `eslint.config.cjs` or `.eslintrc.js` |
| Stylelint             | [stylelint](https://www.npmjs.com/package/stylelint)                                                      | `.stylelintrc.js`                                                                  |
| Licence headers check | _Natively since glpi 11.0.6_<br/> [glpi-project/tools](https://packagist.org/packages/glpi-project/tools) | `tools/HEADER`                                                                     |
| PHPUnit               | [phpunit/phpunit](https://packagist.org/packages/phpunit/phpunit)                                         | `phpunit.xml`                                                                      |
| Jest                  | [jest](https://www.npmjs.com/package/jest)                                                                | `jest.config.js`                                                                   |
| TwigCS                | [friendsoftwig/twigcs](https://github.com/friendsoftwig/twigcs)                                           | `.twig_cs.dist.php`                                                                |

During the `PHPUnit` tests execution, GLPI will be accessible over HTTP (`http://localhost/`).

```yaml
name: "Continuous integration"

on:
  pull_request:

jobs:
  ci:
    uses: "glpi-project/plugin-ci-workflows/.github/workflows/continuous-integration.yml@v1"
    with:
      # The plugin key (system name).
      plugin-key: "myplugin"

      # The version of GLPI on which to run the tests.
      glpi-version: "11.0.x"

      # The version of PHP on which to run the tests.
      php-version: "8.2"

      # The database docker image on which to run the tests.
      db-image: "mariadb:11.4"

      # Optional initialization script that can be used, for instance, to install additional system dependencies.
      init-script: "./.github/workflows/init-script.sh"

      # Optional extra services (possible values: "openldap").
      extra-services: "openldap"

      # Set to true to skip the CHANGELOG update check on pull requests.
      skip-changelog-check: true

      # Whether to enable code coverage generation (default: false).
      code-coverage: true
    secrets:
      # Optional newline-separated KEY=VALUE pairs, exposed as environment
      # variables during the PHPUnit step (e.g. tokens for tests hitting a
      # live external API).
      extra-env: |
        MY_API_TOKEN=${{ secrets.MY_API_TOKEN }}
```

The available `glpi-version`/`php-version` combinations corresponds to the `ghcr.io/glpi-project/githubactions-glpi-apache` images tags
that can be found [here](https://github.com/orgs/glpi-project/packages/container/githubactions-glpi-apache/versions?filters%5Bversion_type%5D=tagged).

The `db-image` parameter is a combination of the DB server engine (`mysql`, `mariadb` or `percona`) and the server version.

- MariaDB available versions are listed [here](https://github.com/orgs/glpi-project/packages/container/githubactions-mariadb/versions?filters%5Bversion_type%5D=tagged)
- MySQL available versions are listed [here](https://github.com/orgs/glpi-project/packages/container/githubactions-mysql/versions?filters%5Bversion_type%5D=tagged).
- Percona available versions are listed [here](https://github.com/orgs/glpi-project/packages/container/githubactions-percona/versions?filters%5Bversion_type%5D=tagged).

An optional `init-script` parameter can be used to define the path of an initialization script. This script will be executed with `bash`.
It can be used, for instance, to install a specific PHP extension.

An optional `extra-env` secret can be used to expose additional environment variables during the PHPUnit step, for instance credentials required by tests that hit a live external API.

On pull requests, the workflow checks that the `CHANGELOG` file has been updated. This check is automatically skipped for Dependabot PRs and when all changed files are in `locales/` or `.github/` (e.g. locale-update PRs). It can also be fully disabled via the `skip-changelog-check` parameter.

On pull requests that modify `plugin.xml` or `<plugin-key>.xml`, the workflow also validates that all URLs declared in the file are reachable. URLs inside `<download_url>` tags that are newly introduced by the PR only produce a warning (the release archive may not be published yet), while all other invalid URLs fail the check.

## Code coverage

Code coverage is automatically enabled when a `.glpi-coverage.json` configuration file is present at the root of the plugin directory.

If the file is not present, or if its `enabled` field is explicitly set to `false`, code coverage steps will be skipped entirely.

### `.glpi-coverage.json` format

All fields are optional. Default values are shown below:

```json
{
	"enabled": true,
	"only_list_changed_files": true,
	"badge": true,
	"overall_coverage_fail_threshold": 0,
	"file_coverage_error_min": 50,
	"file_coverage_warning_max": 75,
	"fail_on_negative_difference": false,
	"retention_days": 90
}
```

| Field                             | Default | Description                                                                                         |
|-----------------------------------|---------|-----------------------------------------------------------------------------------------------------|
| `enabled`                         | `true`  | Set to `false` to disable code coverage entirely.                                                   |
| `only_list_changed_files`         | `true`  | Only list files changed in the PR in the coverage report.                                           |
| `badge`                           | `true`  | Include a coverage badge in the report using shields.io.                                            |
| `overall_coverage_fail_threshold` | `0`     | Fail the workflow if overall coverage is below this percentage.                                     |
| `file_coverage_error_min`         | `50`    | Files with coverage below this percentage are marked as error (red).                                |
| `file_coverage_warning_max`       | `75`    | Files with coverage below this percentage are marked as warning (orange). Above is success (green). |
| `fail_on_negative_difference`     | `false` | Fail the workflow if any file coverage decreased compared to the base branch.                       |
| `retention_days`                  | `90`    | Number of days to retain coverage artifacts for base branch comparison.                             |

> **Tip:** To use as a reference without enabling coverage (e.g. for `glpi-empty`), create the file with `"enabled": false`.

### IDE Integration

The workflow produces a `coverage-report` artifact containing:

- `clover.xml`: Use this file to import coverage into PhpStorm or other IDEs. Paths are automatically sanitized to match `plugins/<plugin-key>/`.
- `cobertura.xml`: Used for the PR comment report.

### Coverage report workflow

The `coverage-report.yml` reusable workflow generates a PR comment with a coverage summary. It compares the coverage from the current PR against the base branch (using stored artifacts).

```yaml
  coverage-report:
    needs: "ci"
    uses: "glpi-project/plugin-ci-workflows/.github/workflows/coverage-report.yml@v1"
    with:
      plugin-key: "myplugin"
```

> **Note:** For base branch comparison to work, the CI workflow must run on every push to the default branch so that an up-to-date coverage artifact is always available.
> 
> A scheduled CI run (e.g., daily cron) is also recommended to ensure the artifact is regenerated before it expires.

## Generate CI matrix

This workflow can be used to generate a matrix that contains the default PHP/SQL versions that are supported by the target GLPI version.
You can use it in combination with the `Continuous Integration` and the `Coverage report` workflows, as shown in the example below.

```yaml
name: "Continuous integration"

on:
  push:
    branches:
      - "main"
    tags:
      - "*"
  pull_request:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:

concurrency:
  group: "${{ github.workflow }}-${{ github.ref }}"
  cancel-in-progress: true

jobs:
  generate-ci-matrix:
    name: "Generate CI matrix"
    uses: "glpi-project/plugin-ci-workflows/.github/workflows/generate-ci-matrix.yml@v1"
    with:
      glpi-version: "10.0.x"

      # Whether the complete compatibility matrix should be generated.
      # Default: false
      complete-matrix: true
  ci:
    name: "GLPI ${{ matrix.glpi-version }} - php:${{ matrix.php-version }} - ${{ matrix.db-image }}"
    needs: "generate-ci-matrix"
    strategy:
      fail-fast: false
      matrix: ${{ fromJson(needs.generate-ci-matrix.outputs.matrix) }}
    uses: "glpi-project/plugin-ci-workflows/.github/workflows/continuous-integration.yml@v1"
    with:
      plugin-key: "myplugin"
      glpi-version: "${{ matrix.glpi-version }}"
      php-version: "${{ matrix.php-version }}"
      db-image: "${{ matrix.db-image }}"

  coverage-report:
    if: github.event_name == 'pull_request'
    needs: "ci"
    uses: "glpi-project/plugin-ci-workflows/.github/workflows/coverage-report.yml@v1"
    with:
      plugin-key: "myplugin"
```
