# php83 (Remi SCL) rebuilds for CentOS 7

Rebuilds Remi's `php83-php-*` Software Collection packages (installed under
`/opt/remi/php83`) from Remi's current source RPMs, for CentOS 7, inside a
CentOS 7 Docker container on this server.

Built and tested: **PHP 8.3.36**, plus PECL igbinary 3.2.16, msgpack 3.0.1,
memcached 3.4.0, rdkafka6 6.0.5, zip 1.22.8. PHP API stays `20230831-64`.

## Spec changes vs. Remi (and why)

PHP (`prepare-spec.sh`):
1. ICU `>= 74` lowered to `>= 73`: newest ICU for EL7 is Remi's `libicu73` (PHP needs >= 57).
2. `pkgconfig(odbc)` replaced by `unixODBC-devel`: EL7's unixODBC 2.3.1 has no `odbc.pc` (the image adds one).
3. `-D__NO_STRING_INLINES`: glibc 2.17 makes `strncmp()` a macro at `-O2`, which breaks `strncmp(x, ZEND_STRL(...))` in ext/ffi.
4. Patch900 `php-openssl102-get0data.patch`: 8.3.36's `xp_ssl.c` calls `ASN1_STRING_get0_data()` (OpenSSL >= 1.1) without the 1.0.2 shim `openssl.c` already has. PHP links the system OpenSSL 1.0.2 as Remi's EL7 builds did.

PECL:
- `%forgemeta` / `%forgesetup` (not in EL7 rpm) replaced with explicit `archivename`/`forgesource`/`%setup`.
- `-std=gnu99` (gcc 4.8 defaults to C90; rdkafka needs C99).
- memcached, rdkafka6 built with `--define 'vendeur remi'` so that they use `remi-libmemcached-awesome` / `remi-librdkafka`, like Remi's EL7 builds.
- zip built against `libzip5` >= 1.10.1 (spec wants 1.11.1, not available for EL7).

All builds use `--nocheck` (test suite skipped).

## Deploying to a CentOS 7 server

1. Copy the folder to the server (or serve it over HTTP) and add a repo file:
   ```ini
   [php83-local]
   name=php83 local rebuilds
   baseurl=file:///srv/php83-repo
   enabled=1
   gpgcheck=0
   ```
2. Runtime libraries come from Remi EL7 (`remi-safe`): `remi-libmemcached-awesome`,
   `remi-librdkafka`, `libzip5` (already installed if the Remi versions are).
3. Upgrade:
   ```bash
   systemctl stop php83-php-fpm
   yum update 'php83-php*'
   /opt/remi/php83/root/usr/sbin/php-fpm -t && systemctl start php83-php-fpm
   php83 -v; php83 -m
   ```

Tested: a clean CentOS 7 container with Remi's 8.3.8 packages (same set as
production) upgraded to these RPMs with `yum update`; all modules load,
`php-fpm -t` passes, and msgpack, igbinary, memcached (igbinary+msgpack), rdkafka,
zip, sodium and HTTPS (OpenSSL) work.
