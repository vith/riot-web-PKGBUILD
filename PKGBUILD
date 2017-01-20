# Maintainer: Jason Papakostas <vithos@gmail.com>
pkgbase='riot-web'
pkgname=('riot-web' 'riot-desktop')
_ver='0.9.6'
pkgver="${_ver//-/_}" # sometimes upstream uses hyphens; we can't
pkgrel=1
arch=('any')
url='https://riot.im'
license=('Apache')
makedepends=('git' 'yarn' 'gendesk')
checkdepends=('phantomjs' 'fontconfig')
replaces=('vector-web')
changelog='CHANGELOG.md'
source=("$pkgbase-v$_ver.tar.gz::https://github.com/vector-im/$pkgbase/archive/v$_ver.tar.gz"
        'riot-desktop.sh'
        'Caddyfile.example')
sha512sums=('074156096fb7382b2914c48dd15eb7af660ec0268729e087f002c2da6f1561137cfd98afe752bd90627bc9998d6ef3dd5202d33f6a8aa4ba460a41c0356606d2'
            '1dcf873b6cdd33c628160ddad3704e1c54909d245cac8f65c6dae4e98c1d9f0e6a0b4879e7826211748ff8e4441fc3617ba4c22a1971a40caf4e1be4ef2a0d21'
            'e79eeada2a945dbc13f9b0d56e89979c9462ed6fc606da0bab516f574adea3ea1a19dd4bb908c800eb865d0dd79f4e616da1425a119f2ab9807af764e1a4483d')

_unpacked_dirname="$pkgbase-$_ver"

prepare() {
  msg2 "Generating desktop entry"
  gendesk -f -n --pkgname "riot-desktop" \
                --name 'Riot' \
                --genericname 'Matrix client' \
                --exec '/usr/bin/riot-desktop' \
                --categories 'Network;Chat;Telephony;VideoConference;FileTransfer' \
                --custom 'StartupWMClass=Riot'

  msg2 "Ensure changelog is up to date"
  cmp "$startdir/CHANGELOG.md" "$srcdir/$_unpacked_dirname/CHANGELOG.md"
}

build() {
  cd "$srcdir/$_unpacked_dirname"

  msg2 "Installing dependencies with yarn"
  yarn install

  msg2 "Cleaning build directory"
  yarn run clean

  msg2 "Building $pkgname"
  yarn run build

  msg2 "Populating version file"
  echo "v$_ver" > "$srcdir/$_unpacked_dirname/webapp/version"
}

check() {
  cd "$srcdir/$_unpacked_dirname"

  msg2 "Running test suite"
  # https://github.com/ariya/phantomjs/issues/14061
  QT_QPA_PLATFORM='' yarn run test
}

package_riot-web() {
  pkgdesc='A glossy Matrix collaboration client for the web'
  replaces=('vector-web')
  install='riot-web.install'

  msg2 "Creating directory structure"
  mkdir -p "$pkgdir"/{usr/share,etc}/webapps/"$pkgname"

  msg2 "Copying public assets"
  cp -RL "$srcdir/$_unpacked_dirname"/webapp/* "$pkgdir/usr/share/webapps/$pkgname"

  msg2 "Copying sample config"
  cp "$srcdir/$_unpacked_dirname/config.sample.json" "$pkgdir/etc/webapps/$pkgname/config.sample.json"

  msg2 "Creating symlink from public dir to /etc/webapps/$pkgname/config.json"
  ln -s "/etc/webapps/$pkgname/config.json" "$pkgdir/usr/share/webapps/$pkgname/config.json"

  msg2 "Copying sample Caddyfile"
  cp "$srcdir/Caddyfile.example" "$pkgdir/etc/webapps/$pkgname"
}

package_riot-desktop() {
  pkgdesc='A glossy Matrix collaboration client'
  depends=('electron')

  msg2 "Copying assets"
  mkdir -p "$pkgdir/usr/lib/$pkgname"/{electron,webapp}
  cp -R "$srcdir/$_unpacked_dirname/electron"/{img,src} "$pkgdir/usr/lib/$pkgname/electron"
  cp -R "$srcdir/$_unpacked_dirname"/{webapp,package.json} "$pkgdir/usr/lib/$pkgname"

  msg2 "Launcher script"
  install -Dm755 "$srcdir/$pkgname.sh" "$pkgdir/usr/bin/$pkgname"

  msg2 "Icons"
  local _i
  for _i in 24 96 16 48 64 128 256 512; do
      install -Dm644 "$srcdir/$_unpacked_dirname/electron/build/icons/${_i}x${_i}.png" \
          "$pkgdir/usr/share/icons/hicolor/${_i}x${_i}/apps/$pkgname.png"
  done

  msg2 "Desktop entry"
  install -Dm644 "$srcdir/$pkgname.desktop" -t "$pkgdir/usr/share/applications"
}
