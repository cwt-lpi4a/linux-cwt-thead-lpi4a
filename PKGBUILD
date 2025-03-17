# Maintainer: Chaiwat Suttipongsakul <cwt@bashell.com>

pkgbase=linux-cwt-510-thead-lpi4a
_variant=cwt
_commit=b9cf70c75d2b7482195a94e754d59f8cfc9dda2c
pkgver=r278.b9cf70c75
pkgrel=1
_desc='Linux 5.10.x (-cwt) for Sipeed Lichee Pi 4A'
_srcname=thead-kernel
url="https://github.com/revyos/thead-kernel"
arch=(riscv64)
license=('GPL2')
makedepends=(bc libelf pahole cpio perl tar xz gcc13 inetutils)
options=('!strip')
source=("git+https://github.com/revyos/thead-kernel.git#commit=$_commit"
  '1-double_size_of_KSYM_NAME_LEN_to_support_very_long_T-HEAD_ISA_extensions.patch'
  '2-add_TH1520_C910_CFLAGS_AFLAGS.patch'
  '3-disable-writeable-clk-debugfs.patch'
  'config'
  'linux.preset'
  '90-linux.hook')

b2sums=('d99a28af7efba717cf495797dd38d11ed4723f6c0c8ca217432ed4965240ab9bfa7c551c3cbb6722b5addb74ae0575761bab35220bf4b310d86b632812c70065'
        'b0641363a1319c9cacef9f850198a4e4f6e7f4ab79f0db536e5d04f60468ccf072d13d398da398016fc0c47b54610acc7629796decfc39c815352d321ce8b2a3'
        '3b40ea9e2da9d96dcb4cef03fb162a77fd4c79dc2ba943ef4c1f1996b9e4854fa1a6ac2d2bfa17b988f114220e887fcc0da5b8f1e2c279b3c2c720d686fae2e2'
        'f9d7989fea320e9ddef43886d261036c96d3dc57f6d727a07a3bf98b8aed53c18f1166bae4c4c1281ae805388e5044f5404315c8ebbd8d2a54decdf25df2d1d4'
        '910c0c213c597d1d83a8269290fc79bcdc67eabd8ef949bab0354a2651abbbf57a6074cce68009254849c6933da4cae66f952a159ddf1d6131fc6690e912f5be'
        'ee004ec1cf9161a960cb9273dd9446bd072c0ff36ccd843e9ed8fed70e3290a5450641798d060f1a20d5a6578e1b47f0b789480aacba7ae0c04ef37c6dbeff0b'
        '0b805af326bbd66b22a8ba0a474f255c4199746160d3ff387a508c0dfc3696cb8e5a459edc325f4adbb5e12ae0167bad1fe9b5b6dbcc1a6a115d40f0363d7a56')

pkgver() {
    cd "$srcdir/$_srcname"
    printf 'r%s.%s' "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

prepare() {
  cd $_srcname

  local src
  for src in $(ls ../*.patch); do
    echo "Applying patch $src..."
    patch -Np1 <"../$src"
  done

  echo "Setting version..."
  scripts/setlocalversion --save-scmversion
  echo "-${_variant}" >localversion.10-variant
  echo "-${pkgver}" >localversion.20-pkgver
  echo "-$pkgrel" >localversion.30-pkgrel

  echo "Setting config..."
  cp ../config .config
  unset CFLAGS
  CCACHE=$(which ccache 2>/dev/null)
  if [ "$CCACHE" != "" ]; then
    gcc="${CCACHE} gcc-13"
  else
    gcc="gcc-13"
  fi
  make HOSTCC="${gcc}" CC="${gcc}" -j $(nproc) olddefconfig
  cp .config ../../config.new

  make HOSTCC="${gcc}" CC="${gcc}" -j $(nproc) -s kernelrelease >version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  cd $_srcname
  unset CFLAGS
  CCACHE=$(which ccache 2>/dev/null)
  if [ "$CCACHE" != "" ]; then
    gcc="${CCACHE} gcc-13"
  else
    gcc="gcc-13"
  fi
  make HOSTCC="${gcc}" CC="${gcc}" -j $(nproc) all
}

_package() {
  pkgdesc="The $_desc kernel and modules"
  depends=(coreutils kmod mkinitcpio)
  optdepends=('wireless-regdb: to set the correct wireless channels of your country'
    'linux-firmware: firmware images needed for some devices')
  provides=("linux=${pkgver}" "WIREGUARD-MODULE")
  conflicts=('linux')

  cd $_srcname
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  echo "Installing boot image..."
  install -Dm644 "arch/riscv/boot/Image" "$modulesdir/vmlinux"
  install -Dm644 "arch/riscv/boot/Image" "$pkgdir/boot/vmlinux"

  echo "Installing modules..."
  CCACHE=$(which ccache 2>/dev/null)
  if [ "$CCACHE" != "" ]; then
    gcc="${CCACHE} gcc-13"
  else
    gcc="gcc-13"
  fi
  make HOSTCC="${gcc}" CC="${gcc}" -j $(nproc) INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

  echo "Installing dtbs..."
  make HOSTCC="${gcc}" CC="${gcc}" -j $(nproc) INSTALL_DTBS_PATH="$pkgdir/usr/share/dtbs/$kernver" dtbs_install
  make HOSTCC="${gcc}" CC="${gcc}" -j $(nproc) INSTALL_DTBS_PATH="$pkgdir/boot/dtbs/arch-cwt" dtbs_install

  # remove build links
  rm "$modulesdir"/build

  install -Dm644 ../linux.preset "${pkgdir}/etc/mkinitcpio.d/linux.preset"
  install -Dm644 ../90-linux.hook "${pkgdir}/usr/share/libalpm/hooks/90-linux.hook"
}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the $_desc kernel"
  depends=(pahole)
  provides=("linux-headers=${pkgver}")
  conflicts=('linux-headers')

  cd $_srcname
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map version
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/riscv" -m644 arch/riscv/Makefile
  cp -t "$builddir" -a scripts

  # required when DEBUG_INFO_BTF_MODULES is enabled
  cp --parents -r -t "$builddir/" tools/bpf/resolve_btfids

  echo "Installing VDSO files..."
  cp -a --parents -r -t "$builddir" arch/riscv/kernel/vdso/*
  cp -a --parents -r -t "$builddir" lib/vdso/*
  chmod -R g+w "$builddir/arch/riscv/kernel/vdso"

  echo "Installing certificate files..."
  install -Dt "$builddir/certs" -m640 certs/*.pem
  install -Dt "$builddir/certs" -m640 certs/*.x509

  echo "Installing headers..."
  cp -t "$builddir" -a include
  chmod -R g+w "$builddir/include/generated"
  cp -t "$builddir/arch/riscv" -a arch/riscv/include
  install -Dt "$builddir/arch/riscv/kernel" -m644 arch/riscv/kernel/asm-offsets.s

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

  # https://bugs.archlinux.org/task/13146
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # https://bugs.archlinux.org/task/20402
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # https://bugs.archlinux.org/task/71392
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  echo "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  echo "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */riscv/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  echo "Installing RAS from x86..."
  install -Dt "$builddir/arch/x86/ras"  -m644 arch/x86/ras/Kconfig

  echo "Removing documentation..."
  rm -r "$builddir/Documentation"

  echo "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  echo "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  echo "Stripping build tools..."
  local file
  while read -rd '' file; do
    case "$(file -bi "$file")" in
    application/x-sharedlib\;*) # Libraries (.so)
      llvm-strip -v $STRIP_SHARED "$file" ;;
    application/x-archive\;*) # Libraries (.a)
      llvm-strip -v $STRIP_STATIC "$file" ;;
    application/x-executable\;*) # Binaries
      llvm-strip -v $STRIP_BINARIES "$file" ;;
    application/x-pie-executable\;*) # Relocatable binaries
      llvm-strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  echo "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

pkgname=("$pkgbase" "$pkgbase-headers")
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

# vim:set ts=8 sts=2 sw=2 et:
