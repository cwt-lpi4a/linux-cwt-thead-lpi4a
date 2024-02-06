# Maintainer: Chaiwat Suttipongsakul <cwt@bashell.com>

pkgbase=linux-cwt-510-thead-lpi4a
_variant=cwt
_commit=2bd2eac4a77b84410e0eb54bd19984186f6300e7
pkgver=r192.2bd2eac4a
pkgrel=1
_desc='Linux 5.10.x (-cwt) for Sipeed Lichee Pi 4A'
_srcname=thead-kernel
url="https://github.com/revyos/thead-kernel"
arch=(riscv64)
license=('GPL2')
makedepends=(bc libelf pahole cpio perl tar xz gcc)
options=('!strip')
source=("git+https://github.com/revyos/thead-kernel.git#commit=$_commit"
  '1-double_size_of_KSYM_NAME_LEN_to_support_very_long_T-HEAD_ISA_extensions.patch'
  '2-add_TH1520_C910_CFLAGS_AFLAGS.patch'
  '3-disable-writeable-clk-debugfs.patch'
  'config'
  'linux.preset'
  '90-linux.hook')

sha256sums=('SKIP'
            'ea7f882f3acb349cf357bf580a051f6a1cd08332e86511dc7070ff835aec9104'
            '3809f65aee776f612aebf5cbb7b512d46dfa99f603a51dcc5f60f1d933bdf345'
            '5528eb99523e64db2d65a0bc6b988c2bfabbadab680ab71990bd0185c57de4c3'
            '707d3a1c2a9c41613fb086208808ba5b61c776c8903de1e27d92d6fb5bc3fe70'
            'e4ff5cbb56c569e7879152b0b41789302f6ede3ddf0a48f2290caa3373a52dff'
            'b12895f9020d124af74d996a4f2be4dc246f49bda868f5a48003a84a4516b4b6')

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
  make -j $(nproc) olddefconfig
  cp .config ../../config.new

  make -j $(nproc) -s kernelrelease >version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  unset CFLAGS
  cd $_srcname
  make -j $(nproc) all
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
  make -j $(nproc) INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

  echo "Installing dtbs..."
  make -j $(nproc) INSTALL_DTBS_PATH="$pkgdir/usr/share/dtbs/$kernver" dtbs_install
  make -j $(nproc) INSTALL_DTBS_PATH="$pkgdir/boot/dtbs/" dtbs_install

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