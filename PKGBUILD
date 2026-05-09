# This is an example PKGBUILD file. Use this as a start to creating your own,
# and remove these comments. For more information, see 'man PKGBUILD'.
# NOTE: Please fill out the license field for your package! If it is unknown,
# then please put 'unknown'.

# Maintainer: Your Name <youremail@domain.com>
pkgname=battery-limit
pkgver=1
pkgrel=1
epoch=
pkgdesc="Limits laptop battery charging to 80% of maximum"
arch=(any)
url=""
license=('unknown')
groups=()
depends=()
makedepends=()
checkdepends=()
optdepends=()
provides=()
conflicts=()
replaces=()
backup=()
options=()
install="battery-limit.install"
changelog=
source=("battery-limit.service")
noextract=()
cksums=("SKIP")
validpgpkeys=()

package() {
	install -Dm644 "${srcdir}/battery-limit.service" "${pkgdir}/etc/systemd/system/battery-limit.service"
}
