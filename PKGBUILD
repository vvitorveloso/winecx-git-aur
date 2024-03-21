# Maintainer : Vitor Veloso <vitorveloso@iid.org.br>
# Based on wine-git: 
## Maintainer : Daniel Bermond <dbermond@archlinux.org>
## Contributor: Sidney Crestani <sidneycrestani@archlinux.net>
## Contributor: sxe <sxxe@gmx.de> 

pkgname=winecx-git
pkgver=crossover.wine.24.0.0.r0.g5321115d
pkgrel=1
pkgdesc='crossover wine'
arch=('x86_64')
url='https://github.com/Gcenx/winecx'
license=('LGPL')
depends=(
    'desktop-file-utils'
    'fontconfig'      'lib32-fontconfig'
    'freetype2'       'lib32-freetype2'
    'gcc-libs'        'lib32-gcc-libs'
    'gettext'         'lib32-gettext'
    'libpcap'         'lib32-libpcap'
    'libunwind'       'lib32-libunwind'
    'libxcursor'      'lib32-libxcursor'
    'libxkbcommon'    'lib32-libxkbcommon'
    'libxi'           'lib32-libxi'
    'libxrandr'       'lib32-libxrandr'
    'wayland'         'lib32-wayland'
)
makedepends=('git' 'perl' 'mingw-w64-gcc'
    'alsa-lib'              'lib32-alsa-lib'
    'gnutls'                'lib32-gnutls'
    'gst-plugins-base-libs' 'lib32-gst-plugins-base-libs'
    'libcups'               'lib32-libcups'
    'libgphoto2'
    'libpulse'              'lib32-libpulse'
    'libxcomposite'         'lib32-libxcomposite'
    'libxcomposite'         'lib32-libxcomposite'
    'libxinerama'           'lib32-libxinerama'
    'libxxf86vm'            'lib32-libxxf86vm'
    'mesa'                  'lib32-mesa'
    'mesa-libgl'            'lib32-mesa-libgl'
    'opencl-headers'
    'opencl-icd-loader'     'lib32-opencl-icd-loader'
    'pcsclite'              'lib32-pcsclite'
    'samba'
    'sane'
    'sdl2'                  'lib32-sdl2'
    'unixodbc'
    'v4l-utils'             'lib32-v4l-utils'
    'vulkan-headers'
    'vulkan-icd-loader'     'lib32-vulkan-icd-loader'
)
optdepends=(
    'alsa-lib'              'lib32-alsa-lib'
    'alsa-plugins'          'lib32-alsa-plugins'
    'cups'                  'lib32-libcups'
    'dosbox'
    'gnutls'                'lib32-gnutls'
    'gst-plugins-bad'
    'gst-plugins-base'      'lib32-gst-plugins-base'
    'gst-plugins-base-libs' 'lib32-gst-plugins-base-libs'
    'gst-plugins-good'      'lib32-gst-plugins-good'
    'gst-plugins-ugly'
    'libgphoto2'
    'libpulse'              'lib32-libpulse'
    'libxcomposite'         'lib32-libxcomposite'
    'libxinerama'           'lib32-libxinerama'
    'opencl-icd-loader'     'lib32-opencl-icd-loader'
    'pcsclite'              'lib32-pcsclite'
    'samba'
    'sane'
    'sdl2'                  'lib32-sdl2'
    'unixodbc'
    'v4l-utils'             'lib32-v4l-utils'
    'wine-gecko'
    'wine-mono'

)
options=('staticlibs' '!lto')
provides=("winecx=${pkgver}" "bin32-winecx=${pkgver}" "winecx-wow64=${pkgver}")
conflicts=('winecx')
replaces=('winecx')
source=('git+https://github.com/Gcenx/winecx.git'
        'distversion.h'
	'30-win32-aliases.conf')
sha256sums=('SKIP'
	    '5d30dd4fa71b55ae5f3e30782a2c93ec506a6c6f65017c1a0995230ad7b39a82'
            '5d30dd4fa71b55ae5f3e30782a2c93ec506a6c6f65017c1a0995230ad7b39a82')

prepare() {
    cp "${srcdir}/distversion.h" "${srcdir}/winecx/programs/winedbg/distversion.h"
    rm -rf build-{32,64}
    mkdir -p build-{32,64}
}

pkgver() {
    git -C winecx describe --long --tags | sed 's/\([^-]*-g\)/r\1/;s/-/./g;s/^wine.//;s/^v//;s/\.rc/rc/'
}

build() {
    export CFLAGS+=' -ffat-lto-objects'
    
    # build wine 64-bit
    # (according to the wine wiki, this 64-bit/32-bit building order is mandatory)
    printf '%s\n' '  -> Building wine-64...'


    cd build-64
    ../winecx/configure \
        --prefix='/opt/winecx' \
        --libdir='/opt/winecx/lib' \
        --with-x \
        --with-wayland \
        --with-gstreamer \
        --with-vulkan \
        --enable-win64
    make
    
    # build wine 32-bit
    printf '%s\n' '  -> Building wine-32...'
    cd "${srcdir}/build-32"
    export PKG_CONFIG_PATH='/opt/winecx/lib32/pkgconfig'
    ../winecx/configure \
        --prefix='/opt/winecx' \
        --libdir='/opt/winecx/lib32' \
        --with-x \
        --with-wayland \
        --with-gstreamer \
        --with-vulkan \
        --with-wine64="${srcdir}/build-64"
    make
}

package() {
    # package wine 32-bit
    # (according to the wine wiki, this reverse 32-bit/64-bit packaging order is important)
    printf '%s\n' '  -> Packaging wine-32...'
    cd build-32
    make -j8 prefix="${pkgdir}/opt/winecx" \
         libdir="${pkgdir}/opt/winecx/lib32" \
         dlldir="${pkgdir}/opt/winecx/lib32/wine" \
         install
    
    # package wine 64-bit
    printf '%s\n' '  -> Packaging wine-64...'
    cd "${srcdir}/build-64"
    make -j8 prefix="${pkgdir}/opt/winecx" \
         libdir="${pkgdir}/opt/winecx/lib" \
         dlldir="${pkgdir}/opt/winecx/lib/wine" \
         install
    
    # font aliasing settings for win32 applications
    install -d -m755 "${pkgdir}/opt/winecx/share/fontconfig/conf.default"
    install -D -m644 "${srcdir}/30-win32-aliases.conf" -t "${pkgdir}/opt/winecx/share/fontconfig/conf.avail"
    ln -s ../conf.avail/30-win32-aliases.conf "${pkgdir}/opt/winecx/share/fontconfig/conf.default/30-win32-aliases.conf"
    
}
