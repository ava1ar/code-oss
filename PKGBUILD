# Maintainer: Michael Hansen <zrax0111 gmail com>
# Contributor: Francisco Magalhães <franmagneto gmail com>
# Contributor: ava1ar <mail(at)ava1ar(dot)me>

_pkgname=code-oss
pkgname=${_pkgname}-git
pkgdesc='Microsoft Code for Linux, Open Source version from git (VSCode)'
pkgver=1.24.1_777c8e57e0
pkgrel=1
arch=('i686' 'x86_64' 'armv7h' 'aarch64')
url="https://github.com/Microsoft/vscode"
license=('MIT')
makedepends=('npm' 'nodejs-lts-carbon' 'gulp' 'python2' 'git' 'yarn')
depends=('gtk3' 'gconf' 'libnotify' 'libxss' 'libxtst' 'libxkbfile' 'nss' 'alsa-lib')
provides=('code')
conflicts=('code')
options=(!strip)
source=("git+https://github.com/Microsoft/vscode"
        "product_json.patch"
        "code-liveshare.patch"
		"aarch64.patch")
sha256sums=('SKIP'
            '2a26fd93719970069da0a326b5ed77592234ffc6d05587b4d5bb8242c7f4c9b1'
            '90b8915d8195546088e845f3205fb965e941561d309c8b462bb0b22a159e041c'
            'aa35f64af75a1abeb11499b365ccdf01c0555d6613f55101a5faffc50663ea6b')

case "$CARCH" in
    i686)
        _vscode_arch=ia32
        ;;
    x86_64)
        _vscode_arch=x64
        ;;
    armv7h)
        _vscode_arch=arm
        ;;
    aarch64)
        _vscode_arch=arm64
        ;;
    *)
        # Needed for mksrcinfo
        _vscode_arch=DUMMY
        ;;
esac

pkgver() {
    cd "${srcdir}/vscode"
    printf "%s_%s" \
		"$(git for-each-ref --sort=-committerdate refs/tags | head -1 | cut -d'/' -f3)" \
        "$(git rev-parse --short HEAD)"
}

prepare() {
    cd "${srcdir}/vscode"

    # This patch no longer contains proprietary modifications.
    # See https://github.com/Microsoft/vscode/issues/31168 for details.
    patch -p1 -i "${srcdir}/product_json.patch"

	# fix aarch64 build
    patch -p1 -i "${srcdir}/aarch64.patch"

    local _commit=$(cd "${srcdir}/vscode" && git rev-parse HEAD)
    local _datestamp=$(date -u -Is | sed 's/\+00:00/Z/')
    sed -e "s/@COMMIT@/${_commit}/" -e "s/@DATE@/${_datestamp}/" -i product.json

    # See https://github.com/MicrosoftDocs/live-share/issues/262 for details
    patch -p1 -i "${srcdir}/code-liveshare.patch"
}

build() {
    cd "${srcdir}/vscode"

    yarn install --arch=${_vscode_arch}

    # The default memory limit may be too low for current versions of node
    # to successfully build vscode.  Uncomment this to set it to 2GB, or
    # change it if this number still doesn't work for your system.
    mem_limit="--max_old_space_size=2048"

    if ! /usr/bin/node $mem_limit /usr/bin/gulp vscode-linux-${_vscode_arch}-min
    then
        echo
        echo "*** NOTE: If the build failed due to running out of file handles (EMFILE),"
        echo "*** you will need to raise your max open file limit."
        echo "*** This can be done by:"
        echo "*** 1) Set a higher 'nofile' limit (at least 10000) in either"
        echo "***    /etc/systemd/system.conf.d/limits.conf (for systemd systems)"
        echo "***    /etc/security/limits.conf (for non-systemd systems)"
        echo "*** 2) Reboot (or log out and back in)"
        echo "*** 3) Run 'ulimit -n' and ensure the value set above is shown before"
        echo "***    re-attempting to build this package."
        echo
        exit 1
    fi
}

package() {
	# Install main software bundle
    install -m 0755 -d "${pkgdir}"/usr/share/${_pkgname}
    cp -r "${srcdir}"/VSCode-linux-${_vscode_arch}/* "${pkgdir}"/usr/share/${_pkgname}

    # Create symlink to the startup script in /usr/bin
    install -m 0755 -d "${pkgdir}"/usr/bin
	ln -s /usr/share/${_pkgname}/bin/${_pkgname} "${pkgdir}"/usr/bin/${_pkgname}

    # Create .desktop file
	install -m 0755 -d "${pkgdir}"/usr/share/applications
	sed -e "s|@@NAME_LONG@@|Microsoft Code OSS|g" \
		-e "s|@@NAME@@|${_pkgname}|g" \
		-e "s|@@NAME_SHORT@@|Code - OSS|g" \
		-e "s|@@ICON@@|code|g" "${srcdir}"/vscode/resources/linux/code.desktop \
		> "${pkgdir}"/usr/share/applications/${_pkgname}.desktop

	# Install icon
    install -D -m644 "${pkgdir}"/usr/share/${_pkgname}/resources/app/resources/linux/code.png \
           "${pkgdir}"/usr/share/pixmaps/code.png

    # Install license file
    install -D -m644 "${srcdir}"/VSCode-linux-${_vscode_arch}/resources/app/LICENSE.txt \
            "${pkgdir}"/usr/share/licenses/${_pkgname}/LICENSE
}
