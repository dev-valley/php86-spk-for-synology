# PHP 8.6 for Synology DSM 7 (Web Station backend)

Unofficial package of **PHP 8.6.0** for Synology NAS running **DSM 7.1 or newer**. After you install it,
**PHP 8.6** appears in Web Station next to Synology's own PHP versions, and you can switch the
extensions on and off per profile, the same way as with Synology's PHP.

Current release: **8.6.0-1** · Author: [dev-valley](https://github.com/dev-valley/)

**Download:** [github.com/dev-valley/php86-spk-for-synology/releases/latest](https://github.com/dev-valley/php86-spk-for-synology/releases/latest)
(all versions: [Releases](https://github.com/dev-valley/php86-spk-for-synology/releases))

php86 is the PHP 8.6 counterpart of [php85](https://github.com/dev-valley/php85-spk) (PHP 8.5) and is
built from the same recipe. The two are separate packages and can be installed side by side, like
Synology's own PHP 8.x packages. Each has its own Web Station backend, so you can move sites from
PHP 8.5 to PHP 8.6 one profile at a time.

> Not affiliated with or supported by Synology. Use at your own risk, and keep a backup.

## Which file for which NAS

Check your model's CPU or platform name at
[Synology: What kind of CPU does my NAS have?](https://kb.synology.com/en-global/DSM/tutorial/What_kind_of_CPU_does_my_NAS_have).
Package Center refuses a package built for the wrong architecture. Download the matching file from the
[latest release](https://github.com/dev-valley/php86-spk-for-synology/releases/latest).

| File | CPU | Platforms | Example models |
|---|---|---|---|
| `php86_x64-7.1_8.6.0-1.spk` | Intel / AMD 64-bit | apollolake, avoton, braswell, broadwell(nk/nkv2/ntbap), bromolow, cedarview, denverton, epyc7002/7003(ntb), geminilake(nk), grantley, icelaked, kvmx64, purley, r1000(nk), v1000(nk) | DS918+, DS920+, DS720+, DS923+, DS1517+, DS1621+, DS1821+, DS224+, DS423+, RS-series, Virtual DSM |
| `php86_aarch64-7.1_8.6.0-1.spk` | ARMv8 64-bit | rtd1296, rtd1619b, armada37xx | DS118, DS218, DS418, DS124, DS223, DS223j, DS423, DS120j, DS220j, DS420j |
| `php86_armv7-7.1_8.6.0-1.spk` | ARMv7 32-bit | alpine, alpine4k, armada370, armada375, armada38x, armadaxp, monaco | DS216j, DS218j, DS418j, DS416, DS216play, DS215j |

Model names are examples only. The platform name is what decides.

**Status:** new. php86 is built from the same recipe as php85, which runs on several x64 models with
DSM 7.2 to 7.4. php86 itself has been tested in containers on all three architectures but **not yet
on a real NAS**. Feedback is welcome. The **ARM packages** haven't been tested on real hardware, so
consider them beta.

SHA-256 (also attached to each release as `SHA256SUMS-<version>.txt`):
```
76199c88652a9d26107999d2e5602122e3694981ff9f6b014d1e24caa08a18fb  php86_x64-7.1_8.6.0-1.spk
c0b9d1be8e281fe4e9eb7c514296627c8978487c26270f72cd638d1350bf5dd1  php86_aarch64-7.1_8.6.0-1.spk
56a061ddb7dfe336f42a9c74be27cff1a0d26838898ca2d762e747c356a556c8  php86_armv7-7.1_8.6.0-1.spk
```

## Installation

1. Install **Web Station** from Package Center.
2. Package Center → **Manual Install** → choose the spk for your CPU. The package isn't signed, so DSM
   shows a warning about the publisher. Confirm it to continue.
3. Web Station → **Script Language Settings** → **Create** → choose **PHP 8.6**.
4. Tick the extensions you need. If an app needs **APCu**, tick **apcu** in the extension list. For the
   two switches at the top of the profile:
   - **"Enable PHP cache": leave it off.** OPcache is always built in and active since PHP 8.5, whatever
     this switch says. The switch loads APCu **and** tries to load OPcache as a separate module, which
     doesn't exist since 8.5. With the switch on, every PHP-FPM start writes a harmless
     `Failed loading Zend extension 'opcache.so'` warning to the log. Ticking apcu **and** turning on
     the switch also logs `Module "apcu" is already loaded`.
   - **"Enable Xdebug": leave it off.** Xdebug isn't included. Turning it on does nothing except write a
     `Failed loading Zend extension 'xdebug.so'` warning to the log.

   Neither warning is shown to visitors (`display_errors` is off).
5. Assign the profile to a web service or portal.

To update, install a newer spk over the existing one without uninstalling first. Your profiles and
their extension selection are kept. Extensions added in a new release show up unticked.

If PHP 8.6 doesn't appear in Web Station, re-register it via SSH:
`sudo /var/packages/WebStation/target/tools/synowebservice plugin-pkg sync php86`

### Moving a site from PHP 8.5 to PHP 8.6

Installing php86 doesn't touch [php85](https://github.com/dev-valley/php85-spk), or Synology's own PHP
packages.

1. Install php86 next to the PHP version the site uses now.
2. Create a new PHP 8.6 profile and tick the same extensions as in the current profile.
3. Assign the new profile to the site and test it. To go back, assign the old profile again.
4. Once no profile uses the old version any more, you can uninstall it.

## PHP 8.6 notes

- **Secure session defaults.** PHP 8.6 turns on `session.use_strict_mode`, `session.cookie_httponly`
  and `session.cookie_samesite = Lax` by default. php86 keeps these defaults in Web Station profiles
  (Synology's PHP 8.4 template, which the profile settings are based on, sets the old values
  explicitly). Apps that read the session cookie from JavaScript, or that need it on cross-site POST
  requests, may need one of these settings changed in the PHP 8.6 profile's settings in Web Station.
- **Deprecations.** PHP 8.6 deprecates a number of things (for example returning values from
  `__construct()`, `define()` with a third argument, `mysqli_get_charset()`, mbregex). Older apps may
  log deprecation notices; they aren't shown to visitors. See the
  [PHP 8.6 migration guide](https://www.php.net/manual/en/migration86.php).
- **PECL extensions.** apcu 5.1.28, imagick 3.8.1 and memcached 3.4.0 don't build with PHP 8.6 as
  released (two internal PHP functions were removed). php86 builds the released versions with the
  upstream fix commits applied, which aren't in a PECL release yet:
  apcu [d29e907](https://github.com/krakjoe/apcu/commit/d29e907791),
  [95f9ab8](https://github.com/krakjoe/apcu/commit/95f9ab828f),
  [9089968](https://github.com/krakjoe/apcu/commit/90899684be);
  imagick [b580a70](https://github.com/Imagick/imagick/commit/b580a705e2);
  memcached [3c73b4f](https://github.com/php-memcached-dev/php-memcached/commit/3c73b4f0ab).

## Troubleshooting

Web Station ties its PHP profiles to the package's run state. If Package Center shows PHP 8.6 as
**Stopped**, Web Station lists the PHP 8.6 backend as **Disabled**, portals using it show **Abnormal**,
and sites return **503**. Start the package again in Package Center.

If it keeps stopping, please open an [issue](https://github.com/dev-valley/php86-spk-for-synology/issues) with your
model, DSM version, package version, and the output of these commands (via SSH, as an admin user):

```
sudo cat /var/packages/php86/var/php86.log
sudo tail -n 50 /var/packages/php86/var/php-fpm.log
sudo tail -n 30 /var/log/packages/php86.log
sudo grep -i php86 /var/log/messages | tail -n 30
sudo /var/packages/php86/target/sbin/php-fpm -t -R --fpm-config /var/packages/php86/var/php-fpm.conf
```

The `-R` in the last command is needed when testing as root. The package itself runs PHP-FPM as its own
user, so without `-R` the test always fails with "'user' directive has not been specified".

Log files:

| File | Contents |
|---|---|
| `/var/packages/php86/var/php86.log` | Package start/stop, and restarts of the package's own PHP-FPM |
| `/var/packages/php86/var/php-fpm.log` | The package's own PHP-FPM (port 9086) |
| `/var/packages/php86/var/php-error.log` | PHP errors from the package's own PHP-FPM pool |
| `/var/log/messages` (ident `php86-fpm`) | Web Station's PHP profile instances |
| `/var/log/packages/php86.log` | DSM's record of package start/stop operations |

## What's included

- `php-fpm`, which Web Station uses (backend id 15, prefix `php86`)
- Commands `php86`, `php86-cgi`, `php-config86` and `phpize86`, installed in
  `/var/packages/php86/target`
- Besides Web Station, the package runs its own PHP-FPM pool on `127.0.0.1:9086` (localhost only), for
  reverse-proxy or CLI setups. It runs under a small supervisor (`bin/php86-fpm-supervise`): if this
  PHP-FPM is terminated by anything other than stopping the package, it's restarted within a few
  seconds and the restart is logged in `php86.log`.

**Compiled in (always on):** Core, ctype, date, dom, fileinfo, filter, hash, json, lexbor, libxml,
mbstring, mysqlnd, pcntl, pcre (with JIT), PDO, random, Reflection, session, SimpleXML, SPL, standard,
tokenizer, uri, xml, xmlreader, xmlwriter, Zend OPcache

**Shared extensions (switchable in Web Station's extension list, 38):** apcu 5.1.28, bcmath, bz2,
calendar, curl, dba (with cdb/flatfile/inifile), exif, ftp, gd, gettext, gmp, iconv, **imagick 3.8.1**,
intl, ldap, mailparse 3.2.0, memcached 3.4.0, mysqli, openssl, pdo_dblib, pdo_mysql, pdo_pgsql,
pdo_sqlite, pgsql, phar, posix, shmop, soap, sockets, sodium, sqlite3, ssh2 1.6.1, sysvmsg, sysvsem,
sysvshm, xsl, zip, zlib

Not included: readline (interactive shell only), xdebug (debugger, not meant for production servers), redis, igbinary.

### imagick / ImageMagick

- Bundled **ImageMagick 7.1.2 Q8** (same colour depth as Synology's own imagick), without OpenMP, so
  it's safe inside PHP-FPM workers.
- Formats: JPEG, PNG, GIF, WebP and TIFF can be read and written. **HEIC and AVIF can be read only**
  (decoders libde265 and dav1d, no encoders). Synology's own PHP 8.4 can't write them either.
- No PDF, PostScript or SVG (no Ghostscript or librsvg).
- **Security policy** (ImageMagick's "websafe" policy) in
  `/var/packages/php86/target/etc/ImageMagick-7/policy.xml`: external delegates and the MVG, MSL and
  URL coders are blocked. Limits: 256 MiB RAM, 16,000 px per side, 128 megapixels, 60 s per image.
  Larger images are rejected with a "resource limit" error. Adjust the file if you really need more.
  Updates overwrite it.

## How it was built

- **Framework:** [SynoCommunity spksrc](https://github.com/SynoCommunity/spksrc), with Synology's
  DSM 7.1 toolchains (GCC 8.5.0) for `x64-7.1`, `aarch64-7.1` and `armv7-7.1`
- **Flags:** `-O2` for PHP and every bundled library, without `-march`, so it runs on every CPU of the
  architecture. Release build: no debug, NTS (non-thread-safe), like Synology's own PHP.
- **Extensions** are built with `=shared`. PECL extensions (APCu, imagick, mailparse, memcached, ssh2) are
  built in-tree from their release sources, with the upstream PHP 8.6 fixes listed above.
  memcached is built without SASL, igbinary or msgpack.
- **Libraries are bundled** in `target/lib` and found via RPATH, so the package doesn't depend on DSM's
  system libraries (or on php85):
  OpenSSL 3.5.9, ICU 78.3, curl 8.22.0, libxml2 2.15.4, libxslt 1.1.45, libsodium 1.0.20,
  libzip 1.12.1, SQLite 3.53, ImageMagick 7.1.2, libheif 1.20.2, libde265, dav1d, lcms2, libpq
  (PostgreSQL 17), OpenLDAP (without SASL), libmemcached-awesome, libssh2, FreeTDS, GMP, libgcrypt,
  libjpeg, libpng, libwebp, freetype, libtiff
- **Web Station integration:** INFO `install_provide_packages="PHP"` plus `target/misc/` (`meta.json`,
  `php-fpm.ini`, `default_settings.json`, `extension_list.json`), modeled on Synology's PHP 8.4
  package. `opcache.jit` is disabled by default, as in Synology's packages.

**Performance:** in container benchmarks (function calls, arrays, strings, regex, JSON), PHP 8.6.0 is
as fast as PHP 8.5.13 on all three architectures. On real hardware, php85 runs as fast as Synology's
PHP 8.4.

## Changelog

- **8.6.0-1:** first release, PHP 8.6.0. Same extensions, bundled libraries, Web Station integration
  and supervised PHP-FPM as [php85](https://github.com/dev-valley/php85-spk) 8.5.13-1, installable
  alongside it (backend id 15, FPM port 9086). PHP 8.6's secure session defaults are kept in Web
  Station profiles. apcu, imagick and memcached include upstream fixes for PHP 8.6; ssh2 1.6.1.

## Licenses

PHP is distributed under the PHP License. The bundled extensions and libraries are under their own
licenses (OpenSSL: Apache 2.0, ICU: Unicode, curl: curl license, ImageMagick: ImageMagick license,
GMP/libheif/libde265/FreeTDS: LGPL, etc.).
