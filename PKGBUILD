# php7 update: BlackIkeEagle <ike DOT devolder AT gmail DOT com>
# Contributor: Pierre Schmitz <pierre@archlinux.de>

pkgbase=php
pkgname=('php'
         'php-cgi'
         'php-apache'
         'php-fpm'
         'php-embed'
         'php-pear'
         'php-enchant'
         'php-gd'
         'php-imap'
         'php-intl'
         'php-ldap'
         'php-mcrypt'
         'php-odbc'
         'php-pgsql'
         'php-pspell'
         'php-snmp'
         'php-sqlite'
         'php-tidy'
         'php-xsl')
_pkgver=7.0.0
_rcver=RC7
pkgver=${_pkgver}$_rcver
pkgrel=1
arch=('i686' 'x86_64')
license=('PHP')
url='http://www.php.net'
makedepends=('apache' 'c-client' 'postgresql-libs' 'libldap' 'postfix'
             'sqlite' 'unixodbc' 'net-snmp' 'libzip' 'enchant' 'file' 'freetds'
             'libmcrypt' 'tidyhtml' 'aspell' 'libltdl' 'gd' 'icu'
             'curl' 'libxslt' 'openssl' 'db' 'gmp' 'systemd' 'libedit' 'wget')
checkdepends=('procps-ng')
if [[ "$_rcver" != "" ]]; then
    source=(
        "http://downloads.php.net/~ab/${pkgbase}-${pkgver}.tar.xz"
    )
    md5sums=(
        '5bb4e6b8bf969d2bd5ab222c60c52dad'
    )
else
    source=(
        "http://www.php.net/distributions/${pkgbase}-${pkgver}.tar.xz"
        "http://www.php.net/distributions/${pkgbase}-${pkgver}.tar.xz.asc"
    )
    md5sums=(
        'a490f25243a19dc6f577dcbc539b97ff'
        'SKIP'
    )
fi

source+=(
    'php.ini.patch'
    'apache.conf'
    'logrotate.d.php-fpm'
    'php-fpm.service'
    'php-fpm.tmpfiles'
)
md5sums+=(
    'fe290cd3bd61a794f14472afde7ad282'
    '00071bea18deee67a3b3dc636630823f'
    '25bc67ad828e8147a817410b68d8016c'
    'cc2940f5312ba42e7aa1ddfab74b84c4'
    'c60343df74f8e1afb13b084d5c0e47ed'
)
validpgpkeys=('6E4F6AB321FDC07F2C332E3AC2BF0BC433CFC8B3'
              '0BD78B5F97500D450838F95DFE857D9A90D90EC1')

prepare() {
	cd ${srcdir}/${pkgbase}-${pkgver}

	patch -p0 -i ${srcdir}/php.ini.patch
	# Just because our Apache 2.4 is configured with a threaded MPM by default does not mean we want to build a ZTS PHP.
	# Let's supress this behaviour and build a SAPI that works fine with the prefork MPM.
	sed '/APACHE_THREADED_MPM=/d' -i sapi/apache2handler/config.m4 -i configure
}

build() {
	local _phpconfig="--srcdir=../${pkgbase}-${pkgver} \
		--config-cache \
		--prefix=/usr \
		--sbindir=/usr/bin \
		--sysconfdir=/etc/php \
		--localstatedir=/var \
		--with-layout=GNU \
		--with-config-file-path=/etc/php \
		--with-config-file-scan-dir=/etc/php/conf.d \
		--disable-rpath \
		--mandir=/usr/share/man \
		--with-kerberos \
		--with-libedit \
		"

	local _phpextensions="--enable-bcmath=shared \
		--enable-calendar=shared \
		--enable-dba=shared \
		--enable-exif=shared \
		--enable-ftp=shared \
		--enable-gd-native-ttf \
		--enable-intl=shared \
		--enable-mbstring \
		--enable-mysqlnd \
		--enable-opcache \
		--enable-phar=shared \
		--enable-posix=shared \
		--enable-shmop=shared \
		--enable-soap=shared \
		--enable-sockets=shared \
		--enable-sysvmsg=shared \
		--enable-sysvsem=shared \
		--enable-sysvshm=shared \
		--enable-zip=shared \
		--with-bz2=shared \
		--with-curl=shared \
		--with-db4=/usr \
		--with-enchant=shared,/usr \
		--with-fpm-systemd \
		--with-freetype-dir=/usr \
		--with-xpm-dir=/usr \
		--with-gd=shared,/usr \
		--with-gdbm \
		--with-gettext=shared \
		--with-gmp=shared \
		--with-iconv=shared \
		--with-icu-dir=/usr \
		--with-imap-ssl \
		--with-imap=shared \
		--with-jpeg-dir=/usr \
		--with-vpx-dir=/usr \
		--with-ldap=shared \
		--with-ldap-sasl \
		--with-libzip \
		--with-mcrypt=shared \
		--with-mhash \
		--with-mysql-sock=/run/mysqld/mysqld.sock \
		--with-mysqli=shared,mysqlnd \
		--with-openssl=shared \
		--with-pcre-regex=/usr \
		--with-pdo-mysql=shared,mysqlnd \
		--with-pdo-odbc=shared,unixODBC,/usr \
		--with-pdo-pgsql=shared \
		--with-pdo-sqlite=shared,/usr \
		--with-pgsql=shared \
		--with-png-dir=/usr \
		--with-pspell=shared \
		--with-snmp=shared \
		--with-sqlite3=shared,/usr \
		--with-tidy=shared \
		--with-unixODBC=shared,/usr \
		--with-xmlrpc=shared \
		--with-xsl=shared \
		--with-zlib \
		"

	EXTENSION_DIR=/usr/lib/php/modules
	export EXTENSION_DIR
	PEAR_INSTALLDIR=/usr/share/pear
	export PEAR_INSTALLDIR

	cd ${srcdir}/${pkgbase}-${pkgver}

	# php
	mkdir ${srcdir}/build-php
	cd ${srcdir}/build-php
	ln -s ../${pkgbase}-${pkgver}/configure
    ./configure ${_phpconfig} \
        --without-pear \
        --disable-cgi \
        --enable-pcntl \
        ${_phpextensions}
    make

	# cgi and fcgi
	# reuse the previous run; this will save us a lot of time
	cp -a ${srcdir}/build-php ${srcdir}/build-cgi
	cd ${srcdir}/build-cgi
	./configure ${_phpconfig} \
		--disable-cli \
		--enable-cgi \
		${_phpextensions}
	make

	# apache
	cp -a ${srcdir}/build-php ${srcdir}/build-apache
	cd ${srcdir}/build-apache
	./configure ${_phpconfig} \
		--disable-cli \
		--with-apxs2 \
		${_phpextensions}
	make

	# fpm
	cp -a ${srcdir}/build-php ${srcdir}/build-fpm
	cd ${srcdir}/build-fpm
	./configure ${_phpconfig} \
		--disable-cli \
		--enable-fpm \
		--with-fpm-user=http \
		--with-fpm-group=http \
		${_phpextensions}
	make

	# embed
	cp -a ${srcdir}/build-php ${srcdir}/build-embed
	cd ${srcdir}/build-embed
	./configure ${_phpconfig} \
		--disable-cli \
		--enable-embed=shared \
		${_phpextensions}
	make

	# pear
	cp -a ${srcdir}/build-php ${srcdir}/build-pear
	cd ${srcdir}/build-pear
	./configure ${_phpconfig} \
		--disable-cgi \
		--enable-pcntl \
		--with-pear \
		${_phpextensions}
	make
}

check() {
	# tests on i686 fail
	[[ $CARCH == 'i686' ]] && return

    # in general the tests fail
	#cd ${srcdir}/build-php

	#export REPORT_EXIT_STATUS=1
	#export NO_INTERACTION=1
	#export SKIP_ONLINE_TESTS=1
	#export SKIP_SLOW_TESTS=1

	#sapi/cli/php -n \
		#${srcdir}/${pkgbase}-${pkgver}/run-tests.php -n -P \
		#${srcdir}/${pkgbase}-${pkgver}/{Zend,ext/{date,pcre,spl,standard},sapi/cli}

    echo
}

package_php() {
	pkgdesc='An HTML-embedded scripting language'
	depends=('pcre' 'libxml2' 'curl' 'libzip' 'libedit' 'openssl')
	backup=('etc/php/php.ini')

	cd ${srcdir}/build-php
	make -j1 INSTALL_ROOT=${pkgdir} install
	install -dm755 ${pkgdir}/usr/share/pear
	# install php.ini
	install -Dm644 ${srcdir}/${pkgbase}-${pkgver}/php.ini-production ${pkgdir}/etc/php/php.ini
	install -dm755 ${pkgdir}/etc/php/conf.d/

	# remove static modules
	rm -f ${pkgdir}/usr/lib/php/modules/*.a
	# remove modules provided by sub packages
	rm -f ${pkgdir}/usr/lib/php/modules/{enchant,gd,imap,intl,ldap,mcrypt,odbc,pdo_odbc,pgsql,pdo_pgsql,pspell,snmp,sqlite3,pdo_sqlite,tidy,xsl}.so
	# remove empty directory
	rmdir ${pkgdir}/usr/include/php/include
    # remove conflicting /var/run folder
    rmdir ${pkgdir}/var/run
	# fix broken link
	ln -sf phar.phar ${pkgdir}/usr/bin/phar

    _install_extensions bz2 curl gettext gmp exif openssl soap sockets xmlrpc \
        mysqli pdo_mysql bcmath calendar dba ftp iconv phar posix shmop \
        sysvmsg sysvsem sysvshm zip
    _install_zend_extensions opcache

    # sane default extensions ?
    ln -sf /etc/php/extensions/curl.ini ${pkgdir}/etc/php/conf.d/curl.ini
    ln -sf /etc/php/extensions/phar.ini ${pkgdir}/etc/php/conf.d/phar.ini
    ln -sf /etc/php/extensions/openssl.ini ${pkgdir}/etc/php/conf.d/openssl.ini
}

package_php-cgi() {
	pkgdesc='CGI and FCGI SAPI for PHP'
	depends=('php')

	install -Dm755 ${srcdir}/build-cgi/sapi/cgi/php-cgi ${pkgdir}/usr/bin/php-cgi
}

package_php-apache() {
	pkgdesc='Apache SAPI for PHP'
	depends=('php' 'apache')
	backup=('etc/httpd/conf/extra/php7_module.conf')

	install -Dm755 ${srcdir}/build-apache/libs/libphp7.so ${pkgdir}/usr/lib/httpd/modules/libphp7.so
	install -Dm644 ${srcdir}/apache.conf ${pkgdir}/etc/httpd/conf/extra/php7_module.conf
}

package_php-fpm() {
	pkgdesc='FastCGI Process Manager for PHP'
	depends=('php' 'systemd')
	backup=('etc/php/php-fpm.conf')
	install='php-fpm.install'

	install -Dm755 ${srcdir}/build-fpm/sapi/fpm/php-fpm ${pkgdir}/usr/bin/php-fpm
	install -Dm644 ${srcdir}/build-fpm/sapi/fpm/php-fpm.8 ${pkgdir}/usr/share/man/man8/php-fpm.8
	install -Dm644 ${srcdir}/build-fpm/sapi/fpm/php-fpm.conf ${pkgdir}/etc/php/php-fpm.conf
	install -Dm644 ${srcdir}/logrotate.d.php-fpm ${pkgdir}/etc/logrotate.d/php-fpm
	install -dm755 ${pkgdir}/etc/php/fpm.d
	install -Dm644 ${srcdir}/php-fpm.tmpfiles ${pkgdir}/usr/lib/tmpfiles.d/php-fpm.conf
	install -Dm644 ${srcdir}/php-fpm.service ${pkgdir}/usr/lib/systemd/system/php-fpm.service
}

package_php-embed() {
	pkgdesc='Embedded PHP SAPI library'
	depends=('php')

	install -Dm755 ${srcdir}/build-embed/libs/libphp7.so ${pkgdir}/usr/lib/libphp7.so
	install -Dm644 ${srcdir}/${pkgbase}-${pkgver}/sapi/embed/php_embed.h ${pkgdir}/usr/include/php/sapi/embed/php_embed.h
}

package_php-pear() {
	pkgdesc='PHP Extension and Application Repository'
	depends=('php')
	backup=('etc/php/pear.conf')

	cd ${srcdir}/build-pear
	make install-pear INSTALL_ROOT=${pkgdir}
	rm -rf ${pkgdir}/usr/share/pear/.{channels,depdb,depdblock,filemap,lock,registry}
}

package_php-enchant() {
	pkgdesc='enchant module for PHP'
	depends=('php' 'enchant')

	install -Dm755 ${srcdir}/build-php/modules/enchant.so ${pkgdir}/usr/lib/php/modules/enchant.so

    _install_extensions enchant
}

package_php-gd() {
	pkgdesc='gd module for PHP'
	depends=('php' 'gd')

	install -Dm755 ${srcdir}/build-php/modules/gd.so ${pkgdir}/usr/lib/php/modules/gd.so

    _install_extensions gd
}

package_php-imap() {
	pkgdesc='imap module for PHP'
	depends=('php' 'c-client')

	install -Dm755 ${srcdir}/build-php/modules/imap.so ${pkgdir}/usr/lib/php/modules/imap.so

    _install_extensions imap
}

package_php-intl() {
	pkgdesc='intl module for PHP'
	depends=('php' 'icu')

	install -Dm755 ${srcdir}/build-php/modules/intl.so ${pkgdir}/usr/lib/php/modules/intl.so

    _install_extensions intl
}

package_php-ldap() {
	pkgdesc='ldap module for PHP'
	depends=('php' 'libldap')

	install -Dm755 ${srcdir}/build-php/modules/ldap.so ${pkgdir}/usr/lib/php/modules/ldap.so

    _install_extensions ldap
}

package_php-mcrypt() {
	pkgdesc='mcrypt module for PHP'
	depends=('php' 'libmcrypt' 'libltdl')

	install -Dm755 ${srcdir}/build-php/modules/mcrypt.so ${pkgdir}/usr/lib/php/modules/mcrypt.so

    _install_extensions mcrypt
}

package_php-odbc() {
	pkgdesc='ODBC modules for PHP'
	depends=('php' 'unixodbc')

	install -Dm755 ${srcdir}/build-php/modules/odbc.so ${pkgdir}/usr/lib/php/modules/odbc.so
	install -Dm755 ${srcdir}/build-php/modules/pdo_odbc.so ${pkgdir}/usr/lib/php/modules/pdo_odbc.so

    _install_extensions odbc pdo_odbc
}

package_php-pgsql() {
	pkgdesc='PostgreSQL modules for PHP'
	depends=('php' 'postgresql-libs')

	install -Dm755 ${srcdir}/build-php/modules/pgsql.so ${pkgdir}/usr/lib/php/modules/pgsql.so
	install -Dm755 ${srcdir}/build-php/modules/pdo_pgsql.so ${pkgdir}/usr/lib/php/modules/pdo_pgsql.so

    _install_extensions pgsql pdo_pgsql
}

package_php-pspell() {
	pkgdesc='pspell module for PHP'
	depends=('php' 'aspell')

	install -Dm755 ${srcdir}/build-php/modules/pspell.so ${pkgdir}/usr/lib/php/modules/pspell.so

    _install_extensions pspell
}

package_php-snmp() {
	pkgdesc='snmp module for PHP'
	depends=('php' 'net-snmp')

	install -Dm755 ${srcdir}/build-php/modules/snmp.so ${pkgdir}/usr/lib/php/modules/snmp.so

    _install_extensions snmp
}

package_php-sqlite() {
	pkgdesc='sqlite module for PHP'
	depends=('php' 'sqlite')

	install -Dm755 ${srcdir}/build-php/modules/sqlite3.so ${pkgdir}/usr/lib/php/modules/sqlite3.so
	install -Dm755 ${srcdir}/build-php/modules/pdo_sqlite.so ${pkgdir}/usr/lib/php/modules/pdo_sqlite.so

    _install_extensions sqlite3 pdo_sqlite
}

package_php-tidy() {
	pkgdesc='tidy module for PHP'
	depends=('php' 'tidyhtml')

	install -Dm755 ${srcdir}/build-php/modules/tidy.so ${pkgdir}/usr/lib/php/modules/tidy.so

    _install_extensions tidy
}

package_php-xsl() {
	pkgdesc='xsl module for PHP'
	depends=('php' 'libxslt')

	install -Dm755 ${srcdir}/build-php/modules/xsl.so ${pkgdir}/usr/lib/php/modules/xsl.so

    _install_extensions xsl
}

_install_extensions() {
    # extensions
    install -dm755 ${pkgdir}/etc/php/extensions/

    for extension in $@; do
        echo "extension=${extension}.so" > ${pkgdir}/etc/php/extensions/${extension}.ini
    done
}

_install_zend_extensions() {
    # extensions
    install -dm755 ${pkgdir}/etc/php/extensions/

    for zend_extension in $@; do
        echo "zend_extension=${zend_extension}.so" > ${pkgdir}/etc/php/extensions/${zend_extension}.ini
    done
}
