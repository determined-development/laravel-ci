# laravel-ci

Docker images for running Laravel CI tasks

Should require minimal effort to test Laravel projects. Comes with most common PHP extensions, and nvm for building node
assets.

Built on `ubuntu:22.04`

## Usage

### ENV

The following environment variables can be used to modify the container setup

| ENV                | Description                                  | Default         |
|--------------------|----------------------------------------------|-----------------|
| `CACHE_DRIVER`     | Cache Driver for testing                     | `array`         |
| `SESSION_DRIVER`   | Session Driver for testing                   | `file`          |
| `QUEUE_CONNECTION` | Queue driver for testing                     | `sync`          |
| `MAIL_MAILER`      | Mailer for testing                           | `array`         |
| `SRC_PATH`         | The directory that contains the laravel root | `/var/www/html` |

Other Laravel environment variables can also be passed as required, or set with `.env` for running tests.

### Custom containers

You can create your own docker image using one of the `onbuild` images as a base

* `ddsam/laravel-ci:latest-onbuild`
* `ddsam/laravel-ci:<version>-onbuild`
* `ddsam/laravel-ci:prerelease-onbuild`
* `ddsam/laravel-ci:<version>-prerelease-onbuild`

#### ARGS

Your own container can be built from the relevant onbuild image with the following build args:

| ARG                | Description                            | Default            |
|--------------------|----------------------------------------|--------------------|
| `WWWUSER`          | The user ID to run as internally.      | `$UID` or `1000`   |
| `WWWGROUP`         | The user group to run as internally.   | `$GROUP` or `1000` |
| `PHP_VERSION`      | The PHP version to install             | `8.4`              |
| `NODE_VERSION`     | Node version to install                | `20`               |
| `POSTGRES_VERSION` | The postgres client version to install | `15`               |

#### Example

```dockerfile
ARG PHP_VERSION=8.4
ARG NODE_VERSION=21

FROM ddsam/laravel-ci:latest-onbuild

CMD ["build-env", "-c", "-k"]
```

### Examples

#### Bitbucket Pipelines

```yaml
image: 'ddsam/laravel-ci:php8.3-latest'

definitions:
  caches:
    vendor:
      key:
        files:
          - composer.json
          - composer.lock
      path: vendor

  services:
    mysql:
      image: mysql/mysql-server:8.0
      variables:
        MYSQL_DATABASE: 'testing'
        MYSQL_USER: 'laravel'
        MYSQL_PASSWORD: 'password'

pipelines:
  pull-requests:
    '**':
      - step:
          name: Prepare environment for testing
          caches:
            - composer
            - vendor
          script:
            - build-env -c -k
          artifacts:
            - '.env'
      - parallel:
          - step:
              name: PHPStan
              caches:
                - vendor
              script:
                - php vendor/bin/phpstan analyse
          - step:
              name: PHPCS
              caches:
                - vendor
              script:
                - php vendor/bin/phpcs ./
          - step:
              name: Pint
              caches:
                - vendor
              script:
                - php vendor/bin/pint --test
          - step:
              name: PHPunit
              caches:
                - vendor
                - node
              env:
                - MYSQL_DATABASE: 'testing'
                - MYSQL_USER: 'laravel'
                - MYSQL_PASSWORD: 'password'
              script:
                - build-env -n -a
                - php artisan test
              services:
                - mysql
```

## Versions

The current supported versions are:

* `ddsam/laravel-ci:php8.0-latest` - PHP 8.0, Node 16 (default)
* `ddsam/laravel-ci:php8.1-latest` - PHP 8.1, Node 16 (default)
* `ddsam/laravel-ci:php8.2-latest` - PHP 8.2, Node 22 (default)
* `ddsam/laravel-ci:php8.3-latest` - PHP 8.3, Node 22 (default)
* `ddsam/laravel-ci:php8.4-latest` - PHP 8.4, Node 22 (default)

### Pre-release versions

Pre-releases get published as:

* `ddsam/laravel-ci:php8.0-prerelease` - PHP 8.0, Node 16 (default)
* `ddsam/laravel-ci:php8.1-prerelease` - PHP 8.1, Node 16 (default)
* `ddsam/laravel-ci:php8.2-prerelease` - PHP 8.2, Node 22 (default)
* `ddsam/laravel-ci:php8.3-prerelease` - PHP 8.3, Node 22 (default)
* `ddsam/laravel-ci:php8.4-prerelease` - PHP 8.4, Node 22 (default)