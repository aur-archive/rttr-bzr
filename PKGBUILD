# Maintainer: Vertoe <vertoe at qhor dot net>
# Contributor: Mineo  <themineo+aur at gmail dot com>

pkgname=rttr-bzr
pkgver=9527
_pkgdate=20141201
pkgrel=1
pkgdesc="Return to the Roots is a remake of Blue Bytes 'The Settlers II'. This is the nightly version."
arch=('i686' 'x86_64')
url="http://rttr.info/"
license=('GPL')
depends=('curl' 'libgl' 'bzip2' 'miniupnpc' 'sdl_mixer' 'lua')
makedepends=('bzr' 'cmake')

_bzrtrunk=lp:s25rttr
_bzrmod=s25rttr
_s2files=/opt/S2

prepare() {
  if [[ -d "$_s2files" ]]; then
    echo "Found game files in $_s2files/."
  else
    echo ""
    echo "++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++"
    echo "ATTENTION: This requires the original Settlers 2 game files in $_s2files/. +"
    echo "    Please make sure you have them located there or modify the PKGBUILD. +"
    echo "++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++"
    echo ""
    exit 1
  fi
}

build() {
  cd "$srcdir"
  echo "Connecting to Bazaar server...."

  if [[ -d "$_bzrmod" ]]; then
    cd "$_bzrmod" && bzr pull "$_bzrtrunk" -r "$pkgver"
    echo "The local files are updated."
  else
    bzr branch "$_bzrtrunk" "$_bzrmod" -r "$pkgver"
  fi

  # Fix CMakeLists
  sed -i '10,13d' $srcdir/$_bzrmod/s-c/src/CMakeLists.txt
  sed -i '9,12d' $srcdir/$_bzrmod/s-c/resample-1.8.1/src/CMakeLists.txt
  sed -i '57,60d' $srcdir/$_bzrmod/s25update/src/CMakeLists.txt
  sed -i '302,309d' $srcdir/$_bzrmod/build/postinstall.sh.cmake

  echo "Bazaar checkout done."
  echo "Starting build..."

  cp -rf "$srcdir/$_bzrmod" "$srcdir/$_bzrmod-build"
  cd "$srcdir/$_bzrmod-build/build"

  ./cmake.sh -DCMAKE_INSTALL_PREFIX=/usr --prefix=/usr

  # Fix client version
  sed -i "5c\
\#define WINDOW\_VERSION \"$_pkgdate\"" $srcdir/$_bzrmod-build/build/build_version.h
  sed -i "6c\
\#define WINDOW\_REVISION \"$pkgver\"" $srcdir/$_bzrmod-build/build/build_version.h
  sed -i "7i\
\#define FORCE true" $srcdir/$_bzrmod-build/build/build_version.h
  cp $srcdir/$_bzrmod-build/build/build_version.h $srcdir/$_bzrmod-build/build/build_version.h.force -vf

  make -j $(nproc)
}

package() {
  cd "$srcdir/$_bzrmod-build/build"
  make DESTDIR="$pkgdir/" install
  rm -rf "$pkgdir/dbg"
  cp -r "$_s2files/" "$pkgdir/usr/share/s25rttr/"
}
