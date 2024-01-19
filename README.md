# laravel-ci

Docker images for running Laravel CI tasks

Should require minimal effort to test Laravel projects. Comes with most common PHP extensions, and nvm for building node
assets.

Built on `ubuntu:22.04`

## Usage

### ARGS

Default build arguments are as follows:

| ARG                | Description                            | Default            |
|--------------------|----------------------------------------|--------------------|
| `WWWUSER`          | The user ID to run as internally.      | `$UID` or `1000`   |
| `WWWGROUP`         | The user group to run as internally.   | `$GROUP` or `1000` |
| `PHP_VERSION`      | The PHP version to install             | `8.3`              |
| `NODE_VERSION`     | Node version to install                | `20`               |
| `POSTGRES_VERSION` | The postgres client version to install | `15`               |

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
      - step:
          name: Prepare environment for testing
          caches:
            - composer
            - node
          script:
            - build-env -n
          artifacts:
            - vendor/**
            - '.env'
      - parallel:
          - step:
              name: PHPStan
              script:
                - php vendor/bin/phpstan analyse
          - step:
              name: PHPCS
              script:
                - php vendor/bin/phpcs ./
          - step:
              name: Pint
              script:
                - php vendor/bin/pint --test
          - step:
              name: PHPunit
              env:
                - MYSQL_DATABASE: 'testing'
                - MYSQL_USER: 'laravel'
                - MYSQL_PASSWORD: 'password'
              script:
                - build-env -c -k -a
                - php artisan test
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
* `ddsam/laravel-ci:php8.3-latest` - PHP 8.3, Node 20 (default)

### Pre-release versions

Pre-releases get published as:

* `ddsam/laravel-ci:php8.0-prerelease` - PHP 8.0, Node 16 (default)
* `ddsam/laravel-ci:php8.1-prerelease` - PHP 8.1, Node 16 (default)
* `ddsam/laravel-ci:php8.2-prerelease` - PHP 8.2, Node 18 (default)
* `ddsam/laravel-ci:php8.3-prerelease` - PHP 8.3, Node 20 (default)