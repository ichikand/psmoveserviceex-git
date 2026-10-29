# Maintainer: ichikand <344514765+ichikand@users.noreply.github.com>
pkgname=psmoveserviceex-git
pkgver=r1.a196b8a
pkgrel=1
pkgdesc="PS Move Service Ex - Modernized C++17 fork with systemd, udev and BlueZ support"
arch=('x86_64')
url="https://github.com/ichikand/PSMoveServiceEx"
license=('Apache-2.0')
depends=(
  'boost-libs'
  'opencv'
  'sdl2'
  'libusb'
  'bluez'
  'bluez-utils'
  'bluez-libs'
  'systemd-libs'
)
makedepends=(
  'git'
  'cmake'
  'boost'
  'protobuf'
  'glm'
  'eigen'
)
provides=('psmoveserviceex')
conflicts=('psmoveserviceex' 'psmoveservice')
source=("git+${url}.git#branch=linux-cxx17-modernization")
sha256sums=('SKIP')

pkgver() {
  cd "PSMoveServiceEx"
  printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

prepare() {
  cd "$srcdir/PSMoveServiceEx"
  git submodule update --init --recursive --depth 1 -j"$(nproc)"
}

build() {
  cmake -B build -S "PSMoveServiceEx" \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_CXX_STANDARD=17 \
    -DBUILD_TESTS=OFF \
    -Wno-dev
  cmake --build build -j"$(nproc)"
}

package() {
  cd "PSMoveServiceEx"

  install -Dm755 "../build/src/psmoveservice/PSMoveService" "$pkgdir/usr/bin/PSMoveService"
  install -Dm755 "../build/src/psmoveconfigtool/PSMoveConfigTool" "$pkgdir/usr/lib/psmoveserviceex/PSMoveConfigTool"

  install -d "$pkgdir/usr/share/psmoveserviceex"
  cp -r src/psmoveconfigtool/assets "$pkgdir/usr/share/psmoveserviceex/" 2>/dev/null || cp -r assets "$pkgdir/usr/share/psmoveserviceex/"

  install -d "$pkgdir/usr/bin"
  cat <<'EOF' > "$pkgdir/usr/bin/PSMoveConfigTool"
#!/bin/sh
cd /usr/share/psmoveserviceex
exec /usr/lib/psmoveserviceex/PSMoveConfigTool "$@"
EOF
  chmod 755 "$pkgdir/usr/bin/PSMoveConfigTool"

  install -Dm755 "../build/src/psmoveclient/libPSMoveClient_CAPI.so" "$pkgdir/usr/lib/libPSMoveClient_CAPI.so"

  install -Dm644 "misc/linux/99-pseye-exclusive.rules" "$pkgdir/usr/lib/udev/rules.d/99-pseye-exclusive.rules"
  install -Dm644 "misc/linux/50-psmoveservice-bluetooth.rules" "$pkgdir/usr/share/polkit-1/rules.d/50-psmoveservice-bluetooth.rules"
  install -Dm644 "misc/linux/psmoveservice.sysusers" "$pkgdir/usr/lib/sysusers.d/psmoveservice.conf"
  install -Dm644 "misc/linux/psmoveservice.service" "$pkgdir/usr/lib/systemd/system/psmoveservice.service"
  install -Dm644 "misc/linux/psmoveservice.conf" "$pkgdir/usr/lib/tmpfiles.d/psmoveservice.conf"

  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}