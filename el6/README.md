# php73 (Remi SCL) 7.3.34 rebuild for CentOS 6, with the php2shell fix

Packages are versioned **7.3.34-1.el6**, and `php -v` / `PHP_VERSION` report
7.3.34, so it is easy to tell the fixed build apart. There is no upstream PHP
7.3.34: the code is 7.3.33 + Remi's security backports + the php2shell patch.

## Spec changes vs. Remi 7.3.33-22 (`prepare-spec.sh`)

1. `upver` 7.3.34, `Release: 1`; new `srcver` 7.3.33 for the tarball and `%setup`.
2. Patch900 `php-7.3-enhanced-remicollet.patch`, applied after Remi's CVE backports.
   Then `php-7.3.34-version.sh` (Source900) sets 7.3.34 in `main/php_version.h`
   and `configure.ac`.
3. New `%changelog` entry on top.
4. ICU: `libicu-devel = 69.1` replaced by `libicu-last-devel >= 50.1` (no libicu69 for EL6;
   Remi's last EL6 php73 builds used libicu-last 50).
5. GD: `gd-devel >= 2.3.3` replaced by `gd-last-devel >= 2.3.0` (newest for EL6).

Build flags: `--define "runselftest 0"` (EL6 rpm has no `--nocheck`) and
`--define "debug_package %{nil}"`; `--define "_scl_prefix /opt/remi" --define "scl_vendor remi"` so files land in `/opt/remi/php73` like Remi's own builds (EL6 scl-utils defaults to `/opt/rh`). EPEL's `autoconf268` replaces `autoconf` (buildconf needs >= 2.68).

## Deploying to a CentOS 6 server

Runtime libraries come from Remi EL6: `libicu-last`, `gd-last`, `libsodium23`,
`oniguruma5php`, `libargon2`, `libzip5` / `libzip-last` (already there if Remi's php73
was installed from Remi's EL6 repos). Copy `out/repo/` to the server, add a repo file

```ini
[php73-local]
name=php73 local rebuild
baseurl=file:///srv/php73-repo
enabled=1
gpgcheck=0
```

then:

```bash
service php73-php-fpm stop
yum update 'php73-php*'
/opt/remi/php73/root/usr/sbin/php-fpm -t && service php73-php-fpm start
php73 -v
```
