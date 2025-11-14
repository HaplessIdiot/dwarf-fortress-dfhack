pkgname=dwarf-fortress-dfhack
pkgver=53.03
pkgrel=1
pkgdesc="Dwarf Fortress Classic ${pkgver} with DFHack; includes pinned runtime libs for Arch rolling"
arch=('x86_64')
url="https://www.bay12games.com/dwarves/"
license=('custom')
depends=('sdl2' 'openal' 'libpng' 'freetype2' 'gcc-libs' 'glibc')
makedepends=('curl' 'tar' 'xz')
# DFHack tag that matches DF ${pkgver}
_dfhack_tag=53.03-r1

# Upstream sources
source=(
  "df-53.03-linux.tar.gz::https://www.bay12games.com/dwarves/df_53_03_linux.tar.gz"
  "dfhack-${_dfhack_tag}-linux-x86_64.tar.xz::https://github.com/DFHack/dfhack/releases/download/${_dfhack_tag}/dfhack-${_dfhack_tag}-linux-x86_64.tar.xz"
  "dwarf-fortress.desktop"
  "LICENSE.custom"
)
sha256sums=(
  'SKIP'  # replace with actual DF tarball checksum
  'SKIP'  # replace with actual DFHack tarball checksum
  'SKIP'  # desktop file checksum
  'SKIP'  # license checksum
)

# Install root (versioned to allow side-by-side installs in future)
_instroot="/opt/dwarf-fortress/${pkgver}"

prepare() {
  # Extract DF
  mkdir -p "${srcdir}/df"
  tar -xf "${srcdir}/df-${pkgver}-linux.tar.bz2" -C "${srcdir}/df"

  # Extract DFHack
  mkdir -p "${srcdir}/dfhack"
  tar -xf "${srcdir}/dfhack-${_dfhack_tag}-linux-x86_64.tar.xz" -C "${srcdir}/dfhack"
}

package() {
  install -dm755 "${pkgdir}${_instroot}"

  # Copy DF files
  cp -r "${srcdir}/df/"* "${pkgdir}${_instroot}/"

  # Install DFHack files over DF
  cp -r "${srcdir}/dfhack/"* "${pkgdir}${_instroot}/"

  # Create lib dir for pinned runtime
  install -dm755 "${pkgdir}${_instroot}/lib"

  # Pin common libc++ stack; try system first (these filenames are stable SONAMEs)
  syslibdir="/usr/lib"
  for so in libc.so.6 libm.so.6 libstdc++.so.6 libgcc_s.so.1; do
    if [[ -e "${syslibdir}/${so}" ]]; then
      install -m644 "${syslibdir}/${so}" "${pkgdir}${_instroot}/lib/${so}"
    fi
  done

  # Allow builder to provide exact-matching pinned libs (glibc 2.35, libstdc++ ABI 3.4.29) at build time
  # Put files in ${srcdir}/pinned_libs/ with names: libc.so.6, libm.so.6, libstdc++.so.6, libgcc_s.so.1
  if [[ -d "${srcdir}/pinned_libs" ]]; then
    for so in libc.so.6 libm.so.6 libstdc++.so.6 libgcc_s.so.1; do
      if [[ -e "${srcdir}/pinned_libs/${so}" ]]; then
        install -m644 "${srcdir}/pinned_libs/${so}" "${pkgdir}${_instroot}/lib/${so}"
      fi
    done
  fi

  # Wrapper that prefers bundled pinned libs and launches DF (DFHack-enabled)
  install -dm755 "${pkgdir}/usr/bin"
  cat > "${pkgdir}/usr/bin/dwarf-fortress" <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

ROOT="/opt/dwarf-fortress/53.03"
LIBDIR="$ROOT/lib"

# Prefer pinned libs if present
if [[ -d "$LIBDIR" ]]; then
  export LD_LIBRARY_PATH="$LIBDIR:${LD_LIBRARY_PATH:-}"
fi

cd "$ROOT"
exec ./df "$@"
EOF
  chmod 755 "${pkgdir}/usr/bin/dwarf-fortress"

  # Desktop entry
  install -dm755 "${pkgdir}/usr/share/applications"
  install -m644 "${srcdir}/dwarf-fortress.desktop" "${pkgdir}/usr/share/applications/dwarf-fortress.desktop"

  # Icon (if present in DF data)
  if [[ -f "${pkgdir}${_instroot}/data/art/icon.png" ]]; then
    install -dm755 "${pkgdir}/usr/share/icons/hicolor/256x256/apps"
    install -m644 "${pkgdir}${_instroot}/data/art/icon.png" "${pkgdir}/usr/share/icons/hicolor/256x256/apps/dwarf-fortress.png"
  fi

  # License
  install -dm755 "${pkgdir}/usr/share/licenses/${pkgname}"
  install -m644 "${srcdir}/LICENSE.custom" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
