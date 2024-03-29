#https://github.com/calamares/calamares/releases

pkgname=salientos-calamares
_pkgname=calamares
pkgver=3.2.61
pkgrel=1
pkgdesc='Distribution-independent installer framework'
arch=('i686' 'x86_64')
license=(GPL)
url="https://github.com/calamares/calamares/releases"
_url="https://raw.githubusercontent.com/salientos/$pkgname/main"
license=('LGPL')
depends=('kdbusaddons' 'kconfig' 'kcoreaddons' 'kiconthemes' 'ki18n' 'kio' 'solid' 'yaml-cpp' 'kpmcore>=4.2.0' 'mkinitcpio-openswap'
         'boost-libs' 'ckbcomp' 'hwinfo' 'qt5-svg' 'polkit-qt5' 'gtk-update-icon-cache' 'plasma-framework'
         'qt5-xmlpatterns' 'squashfs-tools' 'libpwquality' 'efibootmgr' 'icu')
conflicts=('arco-calamares' 'arco-calamares-comp' 'arco-calamares-dev')
makedepends=('extra-cmake-modules' 'qt5-tools' 'qt5-translations' 'git' 'boost')
backup=('usr/share/calamares/modules/bootloader.conf'
        'usr/share/calamares/modules/displaymanager.conf'
        'usr/share/calamares/modules/initcpio.conf'
        'usr/share/calamares/modules/unpackfs.conf')

source=(
	"$_pkgname-$pkgver::$url/download/v$pkgver/$_pkgname-$pkgver.tar.gz"
	$_url/calamares.desktop
	$_url/calamares_polkit
	$_url/49-nopasswd-calamares.rules
)

sha256sums=('7591b9b60738bdba7b9de2b8da5462ab21006db06a006f0dd21ac5b832711dd2'
            '427d7617a9b1210d0b15503c0d625b8465c34591cd16543c116ed4b223173158'
            '966c5f0834039dc6a7529e75f70417ba2510b1e643ffb49fd25855ce9dc244b4'
            '56d85ff6bf860b9559b8c9f997ad9b1002f3fccc782073760eca505e3bddd176')

pkgver() {
	cd ${srcdir}/$_pkgname-$pkgver
	_ver="$(cat CMakeLists.txt | grep -m3 -e "  VERSION" | grep -o "[[:digit:]]*" | xargs | sed s'/ /./g')"
	_git=".r$(git rev-list --count HEAD).$(git rev-parse --short HEAD)"
	printf '%s%s' "${_ver}" "${_git}"
}

prepare() {
	cd ${_pkgname}-${pkgver}
	sed -i -e 's/"Install configuration files" OFF/"Install configuration files" ON/' "$srcdir/${_pkgname}-${pkgver}/CMakeLists.txt"
	sed -i -e 's/# DEBUG_FILESYSTEMS/DEBUG_FILESYSTEMS/' "$srcdir/${_pkgname}-${pkgver}/CMakeLists.txt"
	sed -i -e "s/desired_size = 512 \* 1024 \* 1024  \# 512MiB/desired_size = 512 \* 1024 \* 1024 \* 4  \# 2048MiB/" "$srcdir/${_pkgname}-${pkgver}/src/modules/fstab/main.py"

	# add pkgrelease to patch-version
	_patchver="$(cat CMakeLists.txt | grep -m3 -e CALAMARES_VERSION_PATCH | grep -o "[[:digit:]]*" | xargs)"
	sed -i -e "s|CALAMARES_VERSION_PATCH $_patchver|CALAMARES_VERSION_PATCH $_patchver-$pkgrel|g" CMakeLists.txt
		
	# change branding
	sed -i -e "s/default/salientos/g" src/branding/CMakeLists.txt
		
	# modify desktop file
	sed -i -e 's#Exec=sh.*#Exec=sh -c "/etc/calamares/launch.sh"#g' "$srcdir/${_pkgname}-${pkgver}/calamares.desktop"
	sed -i -e 's#Name=.*#Name=Install SalientOS#g' "$srcdir/${_pkgname}-${pkgver}/calamares.desktop"
	sed -i -e 's#GenericName=.*#GenericName=System Installer#g' "$srcdir/${_pkgname}-${pkgver}/calamares.desktop"
	sed -i -e 's#Comment=.*#Comment=SalientOS System Installer#g' "$srcdir/${_pkgname}-${pkgver}/calamares.desktop"
}

build() {
	cd $_pkgname-$pkgver

	mkdir -p build
	cd build
	cmake .. \
	-DCMAKE_BUILD_TYPE=Release \
	-DCMAKE_INSTALL_PREFIX=/usr \
	-DCMAKE_INSTALL_LIBDIR=lib \
	-DWITH_PYTHONQT=OFF \
	-DWITH_KF5DBus=OFF \
	-DBoost_NO_BOOST_CMAKE=ON \
	-DWEBVIEW_FORCE_WEBKIT=OFF \
	-DSKIP_MODULES="webview tracking interactiveterminal initramfs \
	initramfscfg dracut dracutlukscfg \
	dummyprocess dummypython dummycpp \
	dummypythonqt services-openrc \
	keyboardq localeq welcomeq"
	make
}

package() {
	cd ${_pkgname}-${pkgver}/build
	make DESTDIR="$pkgdir" install
	install -Dm644 "${srcdir}/calamares.desktop" "$pkgdir/usr/share/applications/calamares.desktop"
	install -Dm755 "${srcdir}/calamares_polkit" "$pkgdir/usr/bin/calamares_polkit"
	install -Dm644 "${srcdir}/49-nopasswd-calamares.rules" "$pkgdir/etc/polkit-1/rules.d/49-nopasswd-calamares.rules"
	chmod 750 "$pkgdir"/etc/polkit-1/rules.d
}
