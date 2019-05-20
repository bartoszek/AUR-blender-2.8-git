# Maintainer : bartus <arch-user-repoᘓbartus.33mail.com>

#to enforce cuda verison uncomment this line and update value of sm_xx model accordingly
_cuda_capability="sm_50"

pkgname=blender-2.8-git
pkgver=2.8_r82468.ed1ee89288e
pkgrel=1
pkgdesc="Development version of Blender 2.8 branch"
arch=('i686' 'x86_64')
url="http://blender.org/"
depends=('alembic' 'libgl' 'python' 'python-numpy' 'openjpeg' 'desktop-file-utils' 'hicolor-icon-theme'
         'ffmpeg' 'fftw' 'openal' 'freetype2' 'libxi' 'openimageio' 'opencolorio'
         'openvdb' 'opencollada' 'opensubdiv' 'openshadinglanguage' 'libtiff' 'libpng')
optdepends=('cuda: CUDA support in Cycles')
makedepends=('pacman-contrib' 'git' 'cmake' 'cuda' 'boost' 'mesa' 'llvm')
options=(!strip)
provides=('blender-2.8')
conflicts=('blender-2.8')
license=('GPL')
install=blender.install
# NOTE: the source array has to be kept in sync with .gitmodules
# the submodules has to be stored in path ending with git to match
# the path in .gitmodules.
# More info:
#   http://wiki.blender.org/index.php/Dev:Doc/Tools/Git
source=('git://git.blender.org/blender.git#branch=master' \
        'blender-addons.git::git://git.blender.org/blender-addons.git#branch=blender2.8' \
        'blender-addons-contrib.git::git://git.blender.org/blender-addons-contrib.git#branch=blender2.8' \
        'blender-translations.git::git://git.blender.org/blender-translations.git' \
        'blender-dev-tools.git::git://git.blender.org/blender-dev-tools.git' \
        blender-2.8.desktop \
        ffmpeg.patch \
        )
md5sums=('SKIP'
         'SKIP'
         'SKIP'
         'SKIP'
         'SKIP'
         'cd108dca1c77607c6a7cc45aa284ea97'
         '9d4bfb5b3dd33e95b13cc6c7d9d2d2e1')

pkgver() {
  cd "$srcdir/blender"
  printf "2.8_r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

prepare() {
  cd "$srcdir/blender"
  # update the submodules
  git submodule update --init --recursive --remote
#  git submodule foreach git checkout master
#  git submodule foreach git pull --rebase origin master
  git apply ${srcdir}/ffmpeg.patch
}

build() {

  mkdir -p "$srcdir/blender-build"
  cd "$srcdir/blender-build"
  
  _pyver=$(python -c "from sys import version_info; print(\"%d.%d\" % (version_info[0],version_info[1]))")
  msg "python version detected: ${_pyver}"

  # determine whether we can precompile CUDA kernels
  _CUDA_PKG=`pacman -Qq cuda 2>/dev/null` || true
  if [ "$_CUDA_PKG" != "" ]; then
      _EXTRAOPTS=(-DWITH_CYCLES_CUDA_BINARIES=ON \
                  -DCUDA_TOOLKIT_ROOT_DIR=/opt/cuda)
      if [ "$_cuda_capability" != "" ]; then
        _EXTRAOPTS+=(-DCYCLES_CUDA_BINARIES_ARCH:STRING="${_cuda_capability}")
      fi
  fi

  export CFLAGS="${CFLAGS} -DOPENVDB_3_ABI_COMPATIBLE"
  export CXXFLAGS="${CXXFLAGS} -DOPENVDB_3_ABI_COMPATIBLE"

  cmake "$srcdir/blender" \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DWITH_INSTALL_PORTABLE=OFF \
        -DWITH_CXX11=ON \
        -DWITH_ALEMBIC=NO \
        -DWITH_OPENCOLORIO=ON \
        -DWITH_FFTW3=ON \
        -DWITH_SYSTEM_GLEW=ON \
        -DWITH_CODEC_FFMPEG=ON \
        -DWITH_PYTHON_INSTALL=OFF \
        -DPYTHON_VERSION=${_pyver} \
        -DWITH_MOD_OCEANSIM=ON \
        -DWITH_CYCLES_OPENSUBDIV=ON \
        -DWITH_CYCLES_OSL=ON \
        -DWITH_LLVM=ON \
        -DWITH_IMAGE_OPENEXR=ON \
        -DWITH_OPENSUBDIV=ON \
        -DWITH_OPENVDB=ON \
        -DWITH_OPENVDB_BLOSC=ON \
        -DWITH_OPENCOLLADA=ON \
        ${_EXTRAOPTS[@]}
  make
}

package() {
  cd "$srcdir/blender-build"
  make DESTDIR="$pkgdir" install
  
  msg "add -2.8 sufix to desktop shortcut"
  sed -i 's/=blender/=blender-2.8/g' ${pkgdir}/usr/share/applications/blender.desktop
  sed -i 's/=Blender/=Blender-2.8/g' ${pkgdir}/usr/share/applications/blender.desktop
  mv ${pkgdir}/usr/share/applications/blender.desktop ${pkgdir}/usr/share/applications/blender-2.8.desktop

  msg "add -2.8 sufix to binaries"
  mv ${pkgdir}/usr/bin/blender ${pkgdir}/usr/bin/blender-2.8
  mv ${pkgdir}/usr/bin/blender-thumbnailer.py ${pkgdir}/usr/bin/blender-2.8-thumbnailer.py
#  mv ${pkgdir}/usr/bin/blenderplayer ${pkgdir}/usr/bin/blenderplayer-2.8

  msg "mv doc/blender to doc/blender-2.8"
  mv ${pkgdir}/usr/share/doc/blender ${pkgdir}/usr/share/doc/blender-2.8

  msg "add -2.8 sufix to all icons"  
  for icon in `find ${pkgdir}/usr/share/icons -type f`
  do
    # ${filename##/*.} extra extenssion from path
    # ${filename%.*} extract filename form path
    # look at bash "manipulatin string" 
    mv $icon ${icon%.*}-2.8.${icon##/*.}
  done

  ## not needed when using options=(!strip)?
  #if [ -e "$pkgdir"/usr/share/blender/*/scripts/addons/cycles/lib/ ] ; then
  #  # make sure the cuda kernels are not stripped
  #  chmod 444 "$pkgdir"/usr/share/blender/*/scripts/addons/cycles/lib/*
  #fi
}
# vim:set sw=2 ts=2 et:
