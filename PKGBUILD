pkgname=dwarf-fortress-dfhack
pkgver=53.03
pkgrel=1
pkgdesc="Dwarf Fortress Classic ${pkgver} with DFHack; includes pinned runtime libs for Arch rolling"
arch=('x86_64')
url="https://www.bay12games.com/dwarves/"
license=('custom')
depends=('sdl2' 'openal' 'libpng' 'freetype2' 'gcc-libs' 'glibc')
makedepends=('curl' 'tar' 'xz')

_dfhack_tag=53.03-r1

# Harden downloads (curl UA + redirects)
DLAGENTS=('https::/usr/bin/curl -fLC - --retry 3 --retry-delay 2 -A "Mozilla/5.0 (X11; ArchLinux) makepkg/$(pacman -V | head -n1 | awk '\''{print $3}'\'')" -o %o %u')

source=(
  "df-53.03-linux.tar.bz2::https://www.bay12games.com/dwarves/df_53_03_linux.tar.bz2"
  "dfhack-${_dfhack_tag}-linux-x86_64.tar.xz::https://github.com/DFHack/dfhack/releases/download/${_dfhack_tag}/dfhack-${_dfhack_tag}-linux-x86_64.tar.xz"
  "LICENSE"
sha256sums=(
  'SKIP'  # set after first successful build via updpkgsums
  'SKIP'
  'SKIP'
)

_instroot="/opt/dwarf-fortress/${pkgver}"

prepare() {
  mkdir -p "${srcdir}/df" "${srcdir}/dfhack"
  # Extract DF (bz2)
  tar -xf "${srcdir}/df-53.03-linux.tar.bz2" -C "${srcdir}/df"
  # Extract DFHack (xz)
  tar -xf "${srcdir}/dfhack-${_dfhack_tag}-linux-x86_64.tar.xz" -C "${srcdir}/dfhack"
}

package() {
  install -dm755 "${pkgdir}${_instroot}"
  cp -r "${srcdir}/df/"* "${pkgdir}${_instroot}/"
  cp -r "${srcdir}/dfhack/"* "${pkgdir}${_instroot}/"

  install -dm755 "${pkgdir}${_instroot}/lib"
  syslibdir="/usr/lib"
  for so in libc.so.6 libm.so.6 libstdc++.so.6 libgcc_s.so.1; do
    [[ -e "${syslibdir}/${so}" ]] && install -m644 "${syslibdir}/${so}" "${pkgdir}${_instroot}/lib/${so}"
  done
  if [[ -d "${srcdir}/pinned_libs" ]]; then
    for so in libc.so.6 libm.so.6 libstdc++.so.6 libgcc_s.so.1; do
      [[ -e "${srcdir}/pinned_libs/${so}" ]] && install -m644 "${srcdir}/pinned_libs/${so}" "${pkgdir}${_instroot}/lib/${so}"
    done
  fi

  install -dm755 "${pkgdir}/usr/bin"
  cat > "${pkgdir}/usr/bin/dwarf-fortress" <<'EOF'
#!/usr/bin/env bash
set -euo pipefail
ROOT="/opt/dwarf-fortress/53.03"
LIBDIR="$ROOT/lib"
[[ -d "$LIBDIR" ]] && export LD_LIBRARY_PATH="$LIBDIR:${LD_LIBRARY_PATH:-}"
cd "$ROOT"
exec ./df "$@"
EOF
  chmod 755 "${pkgdir}/usr/bin/dwarf-fortress"

  install -dm755 "${pkgdir}/usr/share/applications"
  install -m644 "${srcdir}/dwarf-fortress.desktop" "${pkgdir}/usr/share/applications/dwarf-fortress.desktop"

  if [[ -f "${pkgdir}${_instroot}/data/art/icon.png" ]]; then
    install -dm755 "${pkgdir}/usr/share/icons/hicolor/256x256/apps"
    install -m644 "${pkgdir}${_instroot}/data/art/icon.png" "${pkgdir}/usr/share/icons/hicolor/256x256/apps/dwarf-fortress.png"
  fi

  install -dm755 "${pkgdir}/usr/share/licenses/${pkgname}"
  install -m644 "${srcdir}/LICENSE.custom" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
