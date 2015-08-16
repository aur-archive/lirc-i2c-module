# Maintainer: graysky <graysky AT archlinux DOT us>
# Maintainer: Paul Mattal <paul@archlinux.org>

pkgname=lirc-i2c-module
pkgver=0.9.0
pkgrel=4
_extramodules=extramodules-3.15-ARCH
pkgdesc="Missing i2c module for PVR-250 and its kin for those of us too lazy to write configs for ir-kbd-i2c."
arch=('i686' 'x86_64')
url="http://www.lirc.org/"
license=('GPL')
makedepends=('help2man' 'linux>=3.15' 'linux<3.16' 'linux-headers>=3.15'
'linux-headers<3.16' 'alsa-lib' 'libx11' 'libftdi' 'libirman' 'python2')
deps=("lirc=$pkgver")
options=('!makeflags' '!strip')
install=lirc.install
source=(http://prdownloads.sourceforge.net/lirc/lirc-${pkgver}.tar.bz2
lirc_wpc8769l.patch
lircd-handle-large-config.patch
lirc_atiusb-kfifo.patch
kernel-2.6.39.patch
linux-3.8.patch)

sha256sums=('6323afae6ad498d4369675f77ec3dbb680fe661bea586aa296e67f2e2daba4ff'
'137b1169810d1b66c5fe058aaffc2043ecbb4ef6cfce62050f9b418fa924b9ba'
'474b5709e6604ef2815e6e1a611d77665e3d33be05cd09110330a81a846bc69f'
'f2a83e2a32c8eb963453214d0337589a293b2327291290ec047f4d78782fb310'
'3dddd4e9f093ee6fe75b3408da269744a4ffcd5255ea2382f077fb32079a2352'
'f1de4e8f14b68f1993dcb2489ae55f2f991f1c4a31db02ee8f6931b0f96baa2e')

build() {
	_kernver="$(cat /usr/lib/modules/${_extramodules}/version)"
	cd "${srcdir}/lirc-${pkgver}"
	patch -Np1 -i "${srcdir}/lirc_wpc8769l.patch"
	patch -Np1 -i "${srcdir}/lircd-handle-large-config.patch"
	patch -Np1 -i "${srcdir}/lirc_atiusb-kfifo.patch"
	patch -Np1 -i "${srcdir}/kernel-2.6.39.patch"
	patch -Np1 -i "${srcdir}/linux-3.8.patch"

	# use fixed instead of Courier w/xmode2, should be more prevalent on linux boxen
	sed -i -e 's|char.*font1_name.*Courier.*$|char font1_name[]="-misc-fixed-*-r-*-*-12-*-*-*-*-*-iso8859-1";|g' tools/xmode2.c

	# use /dev/lirc0 by default instead of /dev/lirc
	sed -i -e 's|#define DEV_LIRC "lirc"|#define DEV_LIRC "lirc0"|' config.h.in

	sed -i '/AC_PATH_XTRA/d' configure.ac
	sed -e 's/@X_CFLAGS@//g' \
		-e 's/@X_LIBS@//g' \
		-e 's/@X_PRE_LIBS@//g' \
		-e 's/@X_EXTRA_LIBS@//g' -i Makefile.am tools/Makefile.am
	# fix for new automake #33497
	sed -i 's/AM_CONFIG_HEADER/AC_CONFIG_HEADER/' configure.ac
	libtoolize
	autoreconf

	PYTHON=python2 ./configure --enable-sandboxed --prefix=/usr \
		--with-driver=all --with-kerneldir=/usr/lib/modules/${_kernver}/build/ \
		--with-moduledir=/usr/lib/modules/${_kernver}/kernel/drivers/misc \
		--sbindir=/usr/bin --with-transmitter

	# Remove drivers already in kernel
	sed -e "s:lirc_dev::" -e "s:lirc_bt829::" -e "s:lirc_igorplugusb::" \
		-e "s:lirc_imon::" -e "s:lirc_parallel::" -e "s:lirc_sasem::" \
		-e "s:lirc_serial::" -e "s:lirc_sir::" -e "s:lirc_ttusbir::" \
		-e "s:lirc_atiusb::" \
		-i Makefile drivers/Makefile drivers/*/Makefile tools/Makefile
	make
}

package() {
	cd "lirc-${pkgver}/drivers"
	make DESTDIR="${pkgdir}" moduledir="/usr/lib/modules/${_extramodules}" install

	# set the kernel we've built for inside the install script
	sed -i -e "s/EXTRAMODULES=.*/EXTRAMODULES=${_extramodules}/g" "${startdir}/lirc.install"
	# gzip -9 modules
	find "${pkgdir}" -name '*.ko' -exec gzip -9 {} \;

	# remove wpc module
	rm "$pkgdir/usr/lib/modules/$_extramodules/lirc_wpc8769l.ko.gz"
}
