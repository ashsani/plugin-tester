# humanmade/plugin-tester

[![Docker Pulls](https://img.shields.io/docker/pulls/humanmade/plugin-tester)](https://hub.docker.com/repository/docker/humanmade/plugin-tester) [![Docker Image Size (latest by date)](https://img.shields.io/docker/image-size/humanmade/plugin-tester)](https://hub.docker.com/repository/docker/humanmade/plugin-tester)

Simple Docker image for running unit tests for WordPress plugins.

To run the tests for your plugin, run this in your plugin directory:

```sh
docker run --rm -v "$PWD:/code" humanmade/plugin-tester
```

You will need `phpunit/phpunit` specified as a Composer dependency of your plugin. Additional arguments can be passed to PHPUnit on the CLI directly, e.g.:

```sh
docker run --rm -v "$PWD:/code" humanmade/plugin-tester --stop-on-error
```

## Configuration

To configure PHPUnit, place a `phpunit.xml.dist` in the plugin root. You can alternatively use the command line arguments for PHPUnit for simpler tests.

Typically your `tests` directory in your plugin should include a `bootstrap.php` including at least the following:

```php
<?php
require '/wp-phpunit/includes/functions.php';

tests_add_filter( 'muplugins_loaded', function () {
	require dirname( __FILE__ ) . '/../your-plugin-entry-point.php';
} );

require '/wp-phpunit/includes/bootstrap.php';
```

## Continuous Integration with Travis

For Travis, the following minimal configuration will get your tests running:

```yaml
services:
  - docker

before_script:
  - composer install

script:
  - docker run --rm -v "$PWD:/code" humanmade/plugin-tester
```

We recommend also [caching the vendor directory](https://docs.travis-ci.com/user/caching/#arbitrary-directories):

```yaml
cache:
  timeout: 1000
  directories:
    - vendor
```

## Continuous Integration with GitHub Actions

You can also use GitHub Actions for continuous integration by using the provided `.github/workflows/image.yml` configuration. Here is an example:

```yaml
name: ci

on:
  push:
    branches:
      - 'master'

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v2
      -
        name: Set up QEMU
        uses: docker/setup-qemu-action@v1
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1
      -
        name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: Build and push latest
        uses: docker/build-push-action@v2
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: humanmade/plugin-tester:latest
      -
        name: Build and push 5.4
        uses: docker/build-push-action@v2
        with:
          context: .
          build-args: |
            WP_VERSION=5.4
          platforms: linux/amd64,linux/arm64
          push: true
          tags: humanmade/plugin-tester:wp-5.4
      -
        name: Build and push 5.5
        uses: docker/build-push-action@v2
        with:
          context: .
          build-args: |
            WP_VERSION=5.5
          platforms: linux/amd64,linux/arm64
          push: true
          tags: humanmade/plugin-tester:wp-5.5
      -
        name: Build and push 5.6
        uses: docker/build-push-action@v2
        with:
          context: .
          build-args: |
            WP_VERSION=5.6
          platforms: linux/amd64,linux/arm64
          push: true
          tags: humanmade/plugin-tester:wp-5.6
```

## Code Coverage

Plugin Tester includes [pcov](https://github.com/krakjoe/pcov) for test coverage, which is natively supported by PHPUnit 8+.

WordPress requires PHPUnit 7, so slight adjustments need to be made to PHPUnit to fix compatibility. Plugin Tester will do this automatically for you, provided you have [pcov-clobber](https://github.com/krakjoe/pcov-clobber) installed via Composer:

```sh
composer require --dev pcov/clobber
```

You can then set up coverage in your `phpunit.xml.dist`, or use the command-line flags:

```sh
docker run --rm -v "$PWD:/code" humanmade/plugin-tester --coverage-text --whitelist inc/
```
