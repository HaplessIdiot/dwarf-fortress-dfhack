# Maintainer: Sven-Hendrik Haase <svenstaro@archlinux.org>
# Added-by: Collin Beyer (patched for DFHack & safer bundled libs)
pkgname=dwarf-fortress-dfhack
pkgver=53.03
_pkgver=${pkgver/./_}
pkgrel=1
pkgdesc="A single-player fantasy game in which you build a dwarven outpost or play an adventurer in a randomly generated world (includes DFHack ${pkgver}-r1)"
arch=('x86_64')
url="https://www.bay12games.com/dwarves/"
license=('LicenseRef-dwarffortress')
depends=('libice' 'gtk3' 'glu' 'sdl2' 'sdl2_image' 'sdl2_mixer' 'sdl2_ttf'
         'libsndfile' 'libsm' 'openal' 'glew' 'gcc-libs' 'glib2')
makedepends=('git' 'cmake' 'curl')
optdepends=('nvidia-utils: If you have nvidia graphics'
            'alsa-lib: for alsa sound'
            'libpulse: for pulse sound')

options=('!strip' '!buildflags' '!debug')
install=$pkgname.install

# Sources
_dfhack_tag=53.03-r1
source=(dwarffortress.sh
        dwarffortress.desktop
        dwarffortress.png
        "https://www.bay12games.com/dwarves/df_${_pkgver}_linux.tar.bz2"
        "dfhack-${_dfhack_tag}-linux-x86_64.tar.xz::https://github.com/DFHack/dfhack/releases/download/${_dfhack_tag}/dfhack-${_dfhack_tag}-linux-x86_64.tar.xz")
sha256sums=('c23b984face14541e48af1b6845c318fae74800b9774c97a1d5b8aa4bfa19b6a'
            'e79e3d945c6cc0da58f4ca30a210c7bf1bc3149fd10406d1262a6214eb40445a'
            '83183abc70b11944720b0d86f4efd07468f786b03fa52fe429ca8e371f708e0f'
            'SKIP') # DFHack release artifacts vary; fill after verifying or updpkgsums

noextract=(df_${_pkgver}_linux.tar.bz2 dfhack-${_dfhack_tag}-linux-x86_64.tar.xz)

prepare() {
  # tar archives have no root directory
  mkdir -p df_${_pkgver}_linux
  tar -xf df_${_pkgver}_linux.tar.bz2 -C df_${_pkgver}_linux

  mkdir -p dfhack_${_dfhack_tag}
  tar -xf dfhack-${_dfhack_tag}-linux-x86_64.tar.xz -C dfhack_${_dfhack_tag}
}

package() {
  local _instroot="/opt/dwarffortress/${pkgver}"
  install -dm755 "$pkgdir/opt"
  cp -r "$srcdir/df_${_pkgver}_linux" "$pkgdir/opt/$pkgname"

  # copy DFHack into the install root alongside the DF binary/data
  cp -r "${srcdir}/dfhack_${_dfhack_tag}/"/* "$pkgdir${_instroot}/"

  # set correct permissions
  find "$pkgdir/opt/$pkgname" -type d -exec chmod 755 {} +
  find "$pkgdir/opt/$pkgname" -type f -exec chmod 644 {} +
  find "$pkgdir/opt/$pkgname" -type f -name "*.so*" -exec chmod 755 {} +
  chmod 755 "$pkgdir/opt/$pkgname/"*.so* || true
  chmod 755 "$pkgdir/opt/$pkgname/run_df" 2>/dev/null || true
  chmod 755 "$pkgdir/opt/$pkgname/dwarfort" 2>/dev/null || true

  # Create a lib dir for optional bundled runtimes (only safe compiler runtimes)
  install -dm755 "${pkgdir}${_instroot}/lib"
  syslibdir="/usr/lib"
  for so in libstdc++.so.6 libgcc_s.so.1; do
    if [[ -e "${syslibdir}/${so}" ]]; then
      install -m644 "${syslibdir}/${so}" "${pkgdir}${_instroot}/lib/${so}"
    fi
  done

  # usr/bin launcher
  install -Dm755 "$srcdir/dwarffortress.sh" "$pkgdir/usr/bin/dwarffortress"
  # If you don't supply dwarffortress.sh, the following inlined launcher will be installed instead:
  if [[ ! -f "$srcdir/dwarffortress.sh" ]]; then
    cat > "${pkgdir}/usr/bin/dwarffortress" <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

PKGVER="@PKGVER@"
ROOT="/opt/dwarffortress/${PKGVER}"
LIBDIR="$ROOT/lib"

# If bundled libdir exists, prepend it so bundled libstdc++/libgcc_s are used.
# Do not bundle or override glibc/libm.
if [[ -d "$LIBDIR" ]]; then
  export LD_LIBRARY_PATH="$LIBDIR:${LD_LIBRARY_PATH:-}"
fi

cd "$ROOT"
# Prefer the bundled DF binary; DFHack may provide wrapper binaries, prefer them first.
if [[ -x "./df" ]]; then
  exec ./df "$@"
elif [[ -x "./dwarf-fortress" ]]; then
  exec ./dwarf-fortress "$@"
elif [[ -x "./run_df" ]]; then
  exec ./run_df "$@"
else
  echo "dwarffortress: game binary not found in $ROOT" >&2
  exit 1
fi
EOF
    sed -i "s|@PKGVER@|${pkgver}|g" "${pkgdir}/usr/bin/dwarffortress"
    chmod 755 "${pkgdir}/usr/bin/dwarffortress"
  fi

  # Desktop launcher with icon
  install -Dm644 "$srcdir/dwarffortress.desktop" "$pkgdir/usr/share/applications/${pkgname}.desktop"
  install -Dm644 "$srcdir/dwarffortress.png" "$pkgdir/usr/share/pixmaps/${pkgname}.png"

  # ship readme/release notes
  install -Dm644 "$srcdir/df_${_pkgver}_linux/readme.txt" "$pkgdir/usr/share/$pkgname/readme.txt"
  install -Dm644 "$srcdir/df_${_pkgver}_linux/release\ notes.txt" "$pkgdir/usr/share/$pkgname/release-notes.txt"

  # license (handled by upstream)
  install -dm755 "$pkgdir/usr/share/licenses/${pkgname}"
  # Bay12's license file name may vary; try common names
  for f in "$srcdir"/df_${_pkgver}_linux/LICENSE "$srcdir"/df_${_pkgver}_linux/LICENSE.txt; do
    [[ -f "$f" ]] && install -m644 "$f" "$pkgdir/usr/share/licenses/${pkgname}/LICENSE"
  done
}

# vim:set ts=2 sw=2 et:
