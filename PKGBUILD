# Maintainer: Christer Solskogen <christer.solskogen@gmail.com>
# Build order: armv7l-binutils -> armv7l-linux-api-headers -> armv7l-gcc-bootstrap -> armv7l-glibc -> armv7l-gcc -> armv7l-glibc (again)
_arch=armv7l
_target=$_arch-unknown-linux-gnueabihf
pkgname=$_arch-glibc
pkgver=2.36
pkgrel=3
_commit=4f4d7a13edfd2fdc57c9d76e1fd6d017fb47550c
pkgdesc="GNU C Library ARM64 target"
arch=(any)
url='https://www.gnu.org/software/libc/'
license=('GPL' 'LGPL')
depends=()
makedepends=(git $_arch-gcc $_arch-linux-api-headers python)
options=(!strip staticlibs)
source=(git+https://sourceware.org/git/glibc.git#commit=${_commit})

validpgpkeys=(7273542B39962DF7B299931416792B4EA25340F8  # "Carlos O'Donell <carlos@systemhalted.org>"
              BC7C7372637EC10C57D7AA6579C43DFBF1CF2187) # Siddhesh Poyarekar
b2sums=('SKIP')

prepare() {
  mkdir -p glibc-build
}

build() {
  cd glibc-build
  echo "build-programs=no" > configparms
  echo "cross-compiling=yes" >> configparms
  echo "slibdir=/usr/lib" >> configparms
  echo "rtlddir=/usr/lib" >> configparms

#Use CFLAGS/CXXFLAGS from Arch Linux ARM
CFLAGS="-march=armv7-a -mfloat-abi=hard -mfpu=neon -O2 -pipe -fstack-protector-strong -fno-plt -fexceptions \
-Wformat -Werror=format-security \
-fstack-clash-protection"
CXXFLAGS="$CFLAGS -Wp,-D_GLIBCXX_ASSERTIONS"

  ../glibc/configure \
      --prefix=/usr \
      --host=$_target \
      --libdir=/usr/lib \
      --with-bugurl=https://aur.archlinux.org/packages/aarch64-glibc \
      --enable-kernel=5.10 \
      --enable-bind-now \
      --disable-multi-arch \
      --enable-stack-protector=strong \
      --disable-profile \
      --disable-werror \
      --disable-timezone-tools 

  make
}

package() {
  cd glibc-build
  make DESTDIR="$pkgdir"/usr/$_target/sys-root install 

  #we don't want static libraries. Only keep the one that we really need.
  find "$pkgdir"/usr/$_target/sys-root  -name '*.a' -and -not -name libc_nonshared.a -delete
  
  #Remove files we don't need in a cross compilation environment 
  rm -r "$pkgdir"/usr/$_target/sys-root/{etc,usr/share,var}

  #strip manually
  find "$pkgdir"/usr/$_target/sys-root -name '*.so' -and ! -name 'libc.so' -print0 | xargs -0 $_target-strip --strip-all

  #create symlink to the shared library since we don't have the static one (gcc needs this)
  ln -s libpthread.so.0 "$pkgdir"/usr/$_target/sys-root/usr/lib/libpthread.so
}
