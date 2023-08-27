# Maintainer: Laurent Carlier <lordheavym@gmail.com>
# Contributor: Evangelos Foutras <foutrelis@gmail.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>

pkgname=('lib32-llvm' 'lib32-llvm-libs')
pkgver=16.0.6
pkgrel=1
arch=('x86_64')
url="https://llvm.org/"
license=('custom:Apache 2.0 with LLVM Exception')
makedepends=('cmake' 'ninja' 'lib32-libffi' 'lib32-zlib' 'lib32-zstd' 'python'
             'gcc-multilib' 'lib32-libxml2')
options=('staticlibs' '!lto') # https://github.com/llvm/llvm-project/issues/57740
_source_base=https://github.com/llvm/llvm-project/releases/download/llvmorg-$pkgver
source=($_source_base/llvm-$pkgver.src.tar.xz{,.sig}
        $_source_base/cmake-$pkgver.src.tar.xz{,.sig}
        $_source_base/third-party-$pkgver.src.tar.xz{,.sig})
sha256sums=('e91db44d1b3bb1c33fcea9a7d1f2423b883eaa9163d3d56ca2aa6d2f0711bc29'
            'SKIP'
            '39d342a4161095d2f28fb1253e4585978ac50521117da666e2b1f6f28b62f514'
            'SKIP'
            '15f5b9aeeba938530af977d5f9205612737a091a7f0f6c8075df8723b7713f70'
            'SKIP')
validpgpkeys=('474E22316ABF4785A88C6E8EA2C794A986419D8A'  # Tom Stellard <tstellar@redhat.com>
              'D574BD5D1D0E98895E3BF90044F2485E45D59042') # Tobias Hieta <tobias@hieta.se>

# Utilizing LLVM_DISTRIBUTION_COMPONENTS to avoid
# installing static libraries; inspired by Gentoo
_get_distribution_components() {
  local target
  ninja -t targets | grep -Po 'install-\K.*(?=-stripped:)' | while read -r target; do
    case $target in
      llvm-libraries|distribution)
        continue
        ;;
      # shared libraries
      LLVM|LLVMgold)
        ;;
      # libraries needed for clang-tblgen
      LLVMDemangle|LLVMSupport|LLVMTableGen)
        ;;
      # exclude static libraries
      LLVM*)
        continue
        ;;
      # exclude llvm-exegesis (doesn't seem useful without libpfm)
      llvm-exegesis)
        continue
        ;;
    esac
    echo $target
  done
}
prepare() {
  rename -v -- "-$pkgver.src" '' {cmake,third-party}-$pkgver.src
  cd llvm-$pkgver.src
  mkdir build
}

build() {
  cd llvm-$pkgver.src/build

  export PKG_CONFIG="i686-pc-linux-gnu-pkg-config"

  # Build only minimal debug info to reduce size
  CFLAGS+=' -g1'
  CXXFLAGS+=' -g1'

  local cmake_args=(
    -G Ninja
    -DCMAKE_BUILD_TYPE=Release
    -DCMAKE_CXX_FLAGS:STRING=-m32
    -DCMAKE_C_FLAGS:STRING=-m32
    -DCMAKE_INSTALL_PREFIX=/usr
    -DCMAKE_SKIP_RPATH=ON
    -DLLVM_BINUTILS_INCDIR=/usr/include
    -DLLVM_BUILD_DOCS=OFF
    -DLLVM_BUILD_LLVM_DYLIB=ON
    -DLLVM_DEFAULT_TARGETS_TRIPLE="i686-pc-linux-gnu"
    -DLLVM_ENABLE_BINDINGS=OFF
    -DLLVM_ENABLE_FFI=ON
    -DLLVM_ENABLE_RTTI=ON
    -DLLVM_ENABLE_SPHINX=OFF
    -DLLVM_HOST_TRIPLE=$CHOST
    -DLLVM_INCLUDE_BENCHMARKS=OFF
    -DLLVM_LIBDIR_SUFFIX=32
    -DLLVM_LINK_LLVM_DYLIB=ON
    -DLLVM_TARGET_ARCH:STRING=i686
    -DLLVM_USE_PERF=ON
  )

  cmake .. "${cmake_args[@]}"
  local distribution_components=$(_get_distribution_components | paste -sd\;)
  test -n "$distribution_components"
  cmake_args+=(-DLLVM_DISTRIBUTION_COMPONENTS="$distribution_components")

  cmake .. "${cmake_args[@]}"
  ninja
}

package_lib32-llvm() {
  pkgdesc="Compiler infrastructure (32-bit)"
  depends=('lib32-llvm-libs' 'llvm')

  cd llvm-$pkgver.src/build

  DESTDIR="$pkgdir" ninja install-distribution

  # The runtime library goes into lib32-llvm-libs
  mv "$pkgdir"/usr/lib32/lib{LLVM,LTO,Remarks}*.so* "$srcdir"
  mv -f "$pkgdir"/usr/lib32/LLVMgold.so "$srcdir"

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib32/*.a

  mv "$pkgdir/usr/bin/llvm-config" "$pkgdir/usr/lib32/llvm-config"

  rm -rf "$pkgdir"/usr/{bin,include,share/{doc,man,llvm,opt-viewer}}

  mkdir "$pkgdir"/usr/bin
  mv "$pkgdir/usr/lib32/llvm-config" "$pkgdir/usr/bin/llvm-config32"

  install -Dm644 ../LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_lib32-llvm-libs() {
  pkgdesc="LLVM runtime libraries (32-bit) "
  depends=('lib32-libffi' 'lib32-zlib' 'lib32-zstd' 'lib32-ncurses'
           'lib32-libxml2' 'lib32-gcc-libs')

  install -d "$pkgdir/usr/lib32"

  cp -P \
    "$srcdir"/lib{LLVM,LTO,Remarks}*.so* \
    "$srcdir"/LLVMgold.so \
    "$pkgdir/usr/lib32/"

  # Symlink LLVMgold.so from /usr/lib/bfd-plugins
  # https://bugs.archlinux.org/task/28479
  install -d "$pkgdir/usr/lib32/bfd-plugins"
  ln -s ../LLVMgold.so "$pkgdir/usr/lib32/bfd-plugins/LLVMgold.so"

  install -Dm644 llvm-$pkgver.src/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
