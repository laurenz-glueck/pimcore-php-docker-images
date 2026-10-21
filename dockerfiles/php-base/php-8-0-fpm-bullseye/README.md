# php 8.0-fpm bullseye base (patched)

## Latest version: 8.0.30-p1

PHP 8.0 is EOL and gets no official releases anymore. This image is the official docker-library
`php:8.0.30-fpm` (bullseye) build, vendored from
[docker-library/php@df4636c](https://github.com/docker-library/php/tree/df4636cac888ec860ccbf0f200637b88a2fb22ab/8.0/bullseye/fpm),
with the php.net source backport for CVE-2026-109618 / CVE-2026-109619 applied to the verified
source tarball. `php -v` still reports `8.0.30`; the patch is visible through
`PHP_SECURITY_PATCHES` and the `io.github.laurenz-glueck.php.security-patches` label.

There is no bookworm variant: PHP 8.0 does not support OpenSSL 3.

Used as base for `pimcore-php/php-8-0-fpm` (from `1.6`).

Patch: `PHP-8.0-CVE-2026-109618-109619.patch`
- Source: php-src branch `PHP-8.0-CVE-2026-109618-109619`, commit
  [`82f8f31b16a035c1b67cc253e79f7cc288f4cc20`](https://github.com/php/php-src/commit/82f8f31b16a035c1b67cc253e79f7cc288f4cc20)
- SHA256: `534478c973cef6f493605b7af907f639a364912ee92c24598ddae44c3bc72d1c`
- Refresh: `gh api -H 'Accept: application/vnd.github.patch' repos/php/php-src/commits/<sha> > PHP-8.0-CVE-2026-109618-109619.patch`

A new backport means a new tag (`8.0.30-p2`, ...). Never re-push an existing tag.

Build image: 
```
docker build --platform linux/amd64 -t ghcr.io/laurenz-glueck/pimcore-php-docker-images/php-base/php-8-0-fpm-bullseye:8.0.30-p1 .
```

You might want to use the `--no-cache` option for a fresh build without cache.

Push image:
```
docker push ghcr.io/laurenz-glueck/pimcore-php-docker-images/php-base/php-8-0-fpm-bullseye:8.0.30-p1
```
