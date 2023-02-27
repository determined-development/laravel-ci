# laravel-ci
Docker images for running Laravel CI tasks

Should require minimal effort to test Laravel projects. Comes with most common PHP extensions, and nvm for building node
assets.

Built on `ubuntu:22.04`

## Usage

### ARGS
Default build arguments are as follows:

| ARG               | Description                          | Default            |
|-------------------|--------------------------------------|--------------------|
| `WWWUSER`         | The user ID to run as internally.    | `$UID` or `1000`   |
| `WWWGROUP`        | The user group to run as internally. | `$GROUP` or `1000` |

### ENV
The following environment variables can be used to modify the container setup

| ENV                | Description                | Default                             |
|--------------------|----------------------------|-------------------------------------|
| `CACHE_DRIVER`     | Cache Driver for testing   | `array`                             |
| `SESSION_DRIVER`   | Session Driver for testing | `file`                              |
| `QUEUE_CONNECTION` | Queue driver for testing   | `sync`                              |
| `MAIL_MAILER`      | Mailer for testing         | `array`                             |
| `NODE_VERSION`     | Node version to install    | `18` (php8.2) `16` (php8.0, php8.1) |

Other Laravel environment variables can also be passed as required, or set with `.env` for running tests.

### Preparing the environment
Running the script `build-env` will install composer, and ensure that an app-key is set. You can pass the flag `-a` to
install node, install npm packages, and attempt a production build.

If you just want node installed, you can run the `setup-node` script.

### Examples

#### Bitbucket Pipelines
```yaml
image: 'ddsam/laravel-ci:php8.2-latest'

pipelines:
  pull-requests:
    '**':
      - parallel:
          - step:
              name: PHPStan
              script:
                - build-env -n
                - mkdir -p test-reports
                - php vendor/bin/phpstan analyse --error-format=junit > ./test-reports/phpstan.xml
              caches:
                - composer
          - step:
              name: PHPCS
              script:
                - build-env -n
                - mkdir -p test-reports
                - php vendor/bin/phpcs ./ --report-junit=./test-reports/phpcs.xml
              caches:
                - composer
          - step:
              name: Pint
              script:
                - build-env -n
                - php vendor/bin/pint --test
              caches:
                - composer
          - step:
              name: PHPunit
              env:
                - MYSQL_DATABASE: 'testing'
                - MYSQL_USER: 'laravel'
                - MYSQL_PASSWORD: 'password'
              script:
                - build-env
                - mkdir -p test-reports
                - build-env -a
                - php artisan test --log-junit ./test-reports/phpunit.xml
              caches:
                - composer
              services:
                - mysql

definitions:
  services:
    mysql:
      image: mysql/mysql-server:8.0
      variables:
        MYSQL_DATABASE: 'testing'
        MYSQL_USER: 'laravel'
        MYSQL_PASSWORD: 'password'
```

## Versions
The current supported versions are:

* `ddsam/laravel-ci:php8.0-latest` - PHP 8.0, Node 16 (default)
* `ddsam/laravel-ci:php8.1-latest` - PHP 8.1, Node 16 (default)
* `ddsam/laravel-ci:php8.2-latest` - PHP 8.2, Node 18 (default)

### Pre-release versions
Pre-releases get published as:
* `ddsam/laravel-ci:php8.0-prerelease` - PHP 8.0, Node 16 (default)
* `ddsam/laravel-ci:php8.1-prerelease` - PHP 8.1, Node 16 (default)
* `ddsam/laravel-ci:php8.2-prerelease` - PHP 8.2, Node 18 (default)