# httpd-plus

Add-ons for the [OpenBSD](https://www.openbsd.org) [`httpd(8)`](http://man.openbsd.org/httpd) web server applicable to the lastest `-current` branch.

Other branches available:
* [6.8-stable](https://github.com/mpfr/httpd-plus/tree/6.8-stable)
* [6.7-stable](https://github.com/mpfr/httpd-plus/tree/6.7-stable)

## List of add-ons

The former `location-access-tests` add-on was removed from this list as it was [imported](https://github.com/openbsd/src/commit/e96b74b9e3e44aa22060826006547b90ccc38faa#diff-ed9bfab4d87ea6df040a9696cb1860f82d56e70486351f950b3fca91eab7175d) into `-current`. The note regarding its usage with [WordPress](https://wordpress.org), however, still applies:
> Even with this add-on installed, WordPress is unable to discover that the OpenBSD web server is now capable to perform required URL rewrites.  This will make the [Permalink Settings Screen](https://wordpress.org/support/article/settings-permalinks-screen/) not behave as expected. Luckily, and for this case exactly, the [got_url_rewrite hook](https://developer.wordpress.org/reference/hooks/got_url_rewrite/) exists.  Adding the following line of code into the current theme's `functions.php` file will straighten things out:
> ```
> add_filter('got_url_rewrite', '__return_true');
> ```

### updates

Bug fixes:
* Failing `directory auto index` of `location` in case enclosing `server` specifies `directory no index` (see on [tech@](https://marc.info/?l=openbsd-tech&m=160293921708844&w=2))

### cache-control-headers

Optional HTTP `Cache-Control` headers via `httpd.conf(5)`.

```
types {
	...
	image/jpeg  { cache "max-age=2592000, public" }             jpeg jpg
	text/css    { cache "max-age=86400, private" }              css
	text/html   { cache "no-store, no-cache, must-revalidate" } html
	...
}
```

### fastcgi-script-overrides

Definition of `script` overrides for `fastcgi` environments via `httpd.conf(5)`. This is mainly intended to be used as a shortcut avoiding unnecessary evaluation rounds for the server.

```
server "www.example.com" {
	...
	location not found "/*" {
		fastcgi {
			socket "/run/php-fpm.sock"
			script "/index.php"
		}
	}
	...
}
```

### client-ip-filters

Client IP matching (`from` or `not from`) for `location` sections in `httpd.conf(5)`.

```
server "www.example.com" {
	listen on * port www

	location "/intranet*" not from "10.0.0/24" { block }
	...
}
```

## How to install

`httpd-plus` is a series of consecutive patch files which may be applied easily by following the steps below.

Make sure your user has sufficient `doas` permissions. To start, `cd` into the user's home directory, for example `/home/mpfr`.

```
$ cat /etc/doas.conf
permit nopass mpfr
$ cd
$ pwd
/home/mpfr
```

Download and extract patch files and installation script.

```
$ rm -rf httpd-plus-current/
$ ftp -Vo - https://codeload.github.com/mpfr/httpd-plus/tar.gz/current | tar xzvf -
httpd-plus-current
httpd-plus-current/00-updates.patch
httpd-plus-current/01-cache-control-headers.patch
httpd-plus-current/02-fastcgi-script-overrides.patch
httpd-plus-current/03-client-ip-filters.patch
httpd-plus-current/LICENSE
httpd-plus-current/README.md
httpd-plus-current/install
```

Apply the patch files by running the installation script. This will build and install the `httpd-plus` binary. After the build process, the original source is restored.

```
$ doas ksh httpd-plus-current/install 2>&1 | tee httpd-plus-install.log
Backing up original sources ... Done.
Applying patch files ...
... 00-updates ...
Hmm...  Looks like a unified diff to me...
The text leading up to this was:
--------------------------
.
.
.
done
... 01-cache-control-headers ...
Hmm...  Looks like a unified diff to me...
.
.
.
done
Building and installing httpd-plus binary and manpage ...
.
.
.
Restoring original sources ... Done.

Installing httpd-plus binary and manpage completed successfully.
Please consult 'man httpd.conf' for further information on new features.
```

Adapt your `httpd.conf` for the newly added features. For further information, have a look at the updated `httpd.conf(5)` manpage via `man httpd.conf`. Make sure your new configuration is valid.

```
$ doas vi /etc/httpd.conf
...
$ doas httpd -n
configuration OK
```

Restart the `httpd` daemon.

```
$ doas rcctl restart httpd
httpd(ok)
httpd(ok)
```

## How to uninstall

The original version of `httpd` can easily be restored by performing a fresh rebuild and reinstall.

```
$ cd /usr/src/usr.sbin/httpd
$ doas make obj
$ doas make clean
$ doas make
$ doas make install
```

Remove `httpd-plus` related features from your configuration file and make sure it is valid. Don't forget to restart the `httpd` daemon.

```
$ doas vi /etc/httpd.conf
...
$ doas httpd -n
configuration OK
$ doas rcctl restart httpd
httpd(ok)
httpd(ok)
```
