# Plugin CI workflows

## Continuous integration workflow

This workflow will execute the following actions as long as they are available on the checked repository.

| Action | Required package | Required config file |
| -------- | -------- | -------- |
| PHP Parallel Lint | [php-parallel-lint/php-parallel-lint](https://packagist.org/packages/php-parallel-lint/php-parallel-lint) | |
| PHP CodeSniffer | [squizlabs/php_codesniffer](https://packagist.org/packages/squizlabs/php_codesniffer) | `.phpcs.xml` |
| PHPStan | [phpstan/phpstan](https://packagist.org/packages/phpstan/phpstan) | `phpstan.neon` |
| ESLint | [eslint](https://www.npmjs.com/package/eslint) | `.eslintrc.js` |
| Stylelint | [stylelint](https://www.npmjs.com/package/stylelint) | `.stylelintrc.js` |
| Licence headers check | [glpi-project/tools](https://packagist.org/packages/glpi-project/tools) | `tools/HEADER` |
| PHPUnit | [phpunit/phpunit](https://packagist.org/packages/phpunit/phpunit) | `phpunit.xml` |
| Jest | [jest](https://www.npmjs.com/package/jest) | `jest.config.js` |

You can use this workflow in any plugin using the following Github Actions configuration.
You will have to use your plugin system name in the `plugin-key` parameter.

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

jobs:
  ci:
    name: "GLPI ${{ matrix.glpi-version }} - php:${{ matrix.php-version }} - ${{ matrix.db-image }}"
    strategy:
      fail-fast: false
      matrix:
        include:
          - {glpi-version: "10.0.x", php-version: "7.4", db-image: "mysql:5.7"}
          - {glpi-version: "10.0.x", php-version: "8.2", db-image: "mariadb:10.6"}
    uses: "glpi-project/plugin-ci-workflows/.github/workflows/continuous-integration.yml@v1"
    with:
      plugin-key: "myplugin"
      glpi-version: "${{ matrix.glpi-version }}"
      php-version: "${{ matrix.php-version }}"
      db-image: "${{ matrix.db-image }}"
```

The available `glpi-version`/`php-version` combinations corresponds to the `ghcr.io/glpi-project/githubactions-glpi` images tags
that can be found [here](https://github.com/orgs/glpi-project/packages/container/githubactions-glpi/versions?filters%5Bversion_type%5D=tagged).

The `db-image` parameter is a combination of the DB server engine (`mysql`, `mariadb` or `percona`) and the server version.
- MariaDB available versions are listed [here](https://github.com/orgs/glpi-project/packages/container/githubactions-mariadb/versions?filters%5Bversion_type%5D=tagged)
- MySQL available versions are listed [here](https://github.com/orgs/glpi-project/packages/container/githubactions-mysql/versions?filters%5Bversion_type%5D=tagged).
- Percona available versions are listed [here](https://github.com/orgs/glpi-project/packages/container/githubactions-percona/versions?filters%5Bversion_type%5D=tagged).
