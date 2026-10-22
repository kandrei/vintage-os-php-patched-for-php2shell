# PHP patched for php2shell vulnerability for old OSs

Various build for PHP used on older servers:

- el7/php83 - Remi's PHP 8.3.36 rebuilt for CentOS 7 with full fixes for php2shell
- el6/php73 - Remi's PHP 7.3.33 + php2shell patch from https://github.com/Anton-Barinov/php2shell-patches/blob/main/patches/php-7.3-enhanced-remicollet.patch rebuilt for CentOS 6 (new version 7.3.34)