# Submitter:   Wessel Dirksen "p-we" <wdirksen at gmail dot com>
# Contributor: Tycho LÃ1⁄4rsen "bas-t" (responsible for hosting and development of FFdecsawrapper and more)
# Contributor: Luis Alves (responsible for original development of linux_media TBS OS lib's)
# Contributor: Petr Vacek "vaca" (providing cardslot.conf for serial port SC readers)
# Contributor: J.P. van Best (implementing new procfs API for kernels >= 3.10 in FFdecsawrapper kernel module)
# Contributor: "Sunday" (tweak for speeding up gzip compression)
# Contributor: Oliver Endress (providing improved mutex patch for dvbdev.c)

# !! This package installs the TBS drivers for you. They should not be pre-installed.

# Important:
# 1) If you use more than 4 DVB tuners, you must specify the number of actual DVB tuners you have below:
# 2) If you use TBS 6281 or 6285, option below to define to use DVB-C mode only

_dvbtuners=4
_DVB_C_only=no ## yes or no

pkgname=ffdecsawrapper-git-tbs-unofficial
pkgver=v150429
pkgrel=1
pkgdesc="FFdecsa empowered softcam - Compiled with open source TBS DVB drivers + firmware from official TBS source"
url="https://github.com/bas-t/ffdecsawrapper.git"
arch=('i686' 'x86_64')
license=('GPLv3')
depends=('v4l-utils')
optdepends=('oscam-svn: smartcard reader support' 'linuxtv-dvb-apps: handy DVB tools')
makedepends=('git' 'patchutils' 'perl-proc-processtable' 'linux-headers')
conflicts=('ffdecsawrapper' 'sasc-ng' 'tbs-dvb-drivers' 'tbs-linux-drivers')
provides=('ffdecsawrapper')
backup=('etc/camdir/cardclient.conf' 'etc/conf.d/ffdecsawrapper' 'etc/camdir/cardslot.conf')
install='ffdecsawrapper.install'

_tbsver=v150429

## Alternately you can use "oldstable" or "master" branch by changing line below:

source=('git://github.com/bas-t/ffdecsawrapper.git#branch=stable' \
	'git://github.com/bas-t/media_tree.git#branch=arch' \
	'git://github.com/bas-t/media_build.git#branch=arch' \
	"http://www.tbsdtv.com/download/document/common/tbs-linux-drivers_$_tbsver.zip" \
	'cardclient.conf' 'cardslot.conf' 'ffdecsawrapper.conf' \
	'ffdecsawrapper.install' 'ffdecsawrapper.lr' 'ffdecsawrapper.service' \
	'ffdecsawrapper.rc' 'v4l-mutex-tbs-new3.patch')
	 
sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'fdc905866a01231595e23c53b7b7b5e81428c10844215c1be1231c4a1297f743'
            '5c23db2b93d1accdc0b3f1612766de38bf7ede5658f6ef973706988dd71d1b81'
            '436eb5a612aa3cb9e45bb2031429f3d41eb596ed65d18659d3bd708919c61253'
            '3e4c28a68d312783761150c9bc5e8239261f9fba11f76f1ab8d072b71391c55c'
            'd6ceef7a559ad49722663b4f0b2762ff1ebce3801dd56cd801c24112c9a0cba9'
            'f435344dc9f1c0ed7c2e0de74ec434cd73e2130a0d7589a4d38338e45925d8db'
            'e798aacd050c078083a477b1fc393fa2dacf9b413ab8fea5a80449d3423c2b22'
            '1dcc2a7002805f09f8f2e582102bc5f3159d0d34d73c49b433872694ed633e8c'
            'e81e69af5e0722cc41dd1baa04edfb25f81198e3616d64b99629ce10f1886e1a')

pkgver() {

	_kernel=`uname -r | sed -r 's/-/_/g'`

	cd $srcdir/ffdecsawrapper
	_gitffdecsawrapper=`git describe --always | sed 's|-|.|g'`
	cd $srcdir/media_tree
	_gitmedia_tree=`git describe --always | sed 's|-|.|g'`
	cd $srcdir/media_build
	_gitmedia_build=`git describe --always | sed 's|-|.|g'`

	echo "$_gitffdecsawrapper"_"$_gitmedia_tree"_"$_gitmedia_build"_"$_tbsver"_"$_kernel"
}
            
prepare() {

	    if [ "$_DVB_C_only" == "yes" ]; then
		   sed -i 's/SYS_DVBT, SYS_DVBT2, SYS_DVBC_ANNEX_A/SYS_DVBC_ANNEX_A/' $srcdir/media_tree/drivers/media/dvb-frontends/si2168.c
		   echo "Compiling TBS-6285 and 6281 drivers in DVB-C mode only..."
		   sleep 4
	    fi

	cd $srcdir
	rm -rf /linux-tbs-drivers
	tar xjvf linux-tbs-drivers.tar.bz2
	chmod 777 -R $srcdir/linux-tbs-drivers

	_v4lconfig="s/CONFIG_DVB_MAX_ADAPTERS=8/CONFIG_DVB_MAX_ADAPTERS=$(($_dvbtuners * 2))/"
	
	cd $srcdir/media_build
	make distclean
	make dir DIR=../media_tree
	make allyesconfig
	sed -i 's/CONFIG_MEDIA_CONTROLLER_DVB=y/# CONFIG_MEDIA_CONTROLLER_DVB is not set/' v4l/.config
	
	cd $srcdir
	msg "Compiling with number of DVB tuners set to $(($_dvbtuners * 2))..."
	msg "Applying v4l-mutex patch..."
	patch -p0 < $srcdir/v4l-mutex-tbs-new3.patch
	sleep 4
}

build() {
	cd $srcdir/media_build
	make -j3
	
	cd $srcdir/ffdecsawrapper      
	./configure --dvb_dir=$srcdir/media_build/linux --update=no
}

package() {

	mkdir -p $pkgdir/usr/bin
	mkdir -p $pkgdir/usr/lib/modules/`uname -r`/updates/{v4l-tbs,ffdecsawrapper}
	mkdir -p $pkgdir/usr/lib/firmware
	mkdir -p $pkgdir/etc/conf.d
	mkdir -p $pkgdir/etc/rc.d
	mkdir -p $pkgdir/etc/camdir
	mkdir -p $pkgdir/etc/logrotate.d
	mkdir -p $pkgdir/usr/lib/systemd/system

	install -m0644 $srcdir/cardclient.conf  $pkgdir/etc/camdir/cardclient.conf
	install -m0644 $srcdir/cardslot.conf  $pkgdir/etc/camdir/cardslot.conf
	install -m0755 $srcdir/ffdecsawrapper.rc  $pkgdir/etc/rc.d/ffdecsawrapper
	install -m0644 $srcdir/ffdecsawrapper.conf  $pkgdir/etc/conf.d/ffdecsawrapper
	install -m0644 $srcdir/ffdecsawrapper.lr  $pkgdir/etc/logrotate.d/ffdecsawrapper-git-tbs-unofficial.lr
	install -m0644 $srcdir/ffdecsawrapper.service  $pkgdir/usr/lib/systemd/system/ffdecsawrapper.service      

	install -m0755 $srcdir/ffdecsawrapper/ffdecsawrapper  $pkgdir/usr/bin
	install -m0755 $srcdir/ffdecsawrapper/dvbloopback.ko  $pkgdir/usr/lib/modules/`uname -r`/updates/ffdecsawrapper
	find    "$srcdir/media_build" -name '*.ko' -exec cp {} $pkgdir/usr/lib/modules/`uname -r`/updates/v4l-tbs \;
	install -m0644 $srcdir/*dvb*.fw  $pkgdir/usr/lib/firmware
	install -m0644 $srcdir/media_tree/firmware_extra/*dvb*.fw  $pkgdir/usr/lib/firmware

	msg "Compressing modules, this will take awhile..."	
	find "$pkgdir" -name '*.ko' -print0 | xargs -0 -P`nproc` -n10 gzip -9
	
	chmod 755 -R $pkgdir/usr/lib/modules/`uname -r`/updates
}

