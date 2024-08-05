# Maintainer: Bin Jin <bjin@ctrl-d.org>
# Contributor: Kevin Stolp <kevinstolp@gmail.com>
# Contributor: Eli Schwartz <eschwartz@archlinux.org>
# Contributor: Iacopo Isimbaldi <isiachi@rhye.it>

_pkgname=zfs
_git_repo=https://github.com/openzfs/zfs.git
_git_branch="$(/usr/bin/git ls-remote -h --sort=-v:refname "${_git_repo}" 'zfs-*-staging' | head -n 1)"
_git_branch=${_git_branch##*/}
_staging_ver=${_git_branch#zfs-}
_staging_ver=${_staging_ver%-staging}

if /usr/bin/git ls-remote -t --exit-code "${_git_repo}" "zfs-${_staging_ver}" >/dev/null; then
    _git_branch="tag=zfs-${_staging_ver}"
else
    _git_branch="branch=${_git_branch}"
fi

pkgname=${_pkgname}-dkms-staging-git
pkgver=2.2.4.r12.g722af42420
pkgrel=1
pkgdesc="Kernel modules for the Zettabyte File System (release staging branch)."
arch=('any')
url="https://zfsonlinux.org/"
license=('CDDL-1.0')
groups=('zfs-staging-git')
provides=("ZFS-MODULE" "SPL-MODULE" "zfs-dkms" "zfs")
conflicts=("zfs-dkms")
makedepends=("git")
source=("${_pkgname}::git+${_git_repo}#${_git_branch}"
        "linux69-call-adddisk.patch::https://github.com/openzfs/zfs/commit/49f3ce338587410cabc078646b76152685ae102d.patch?full_index=1"
        "disable-dependency-tracking.patch::https://github.com/openzfs/zfs/commit/c98295eed2687cee704ef5f8f3218d3d44a6a1d8.patch?full_index=1"
        "linux610-rework-queue-limits-setup.patch::https://github.com/openzfs/zfs/commit/b409892ae5028965a6fe98dde1346594807e6e45.patch?full_index=1"
        "linux610-avoid-kmem_cache_alloc.patch::https://github.com/openzfs/zfs/commit/e951dba48a6330aca9c161c50189f6974e6877f0.patch?full_index=1"
        "linux516-use-bdev_nr_bytes.patch::https://github.com/openzfs/zfs/commit/7ca7bb7fd723a91366ce767aea53c4f5c2d65afb.patch?full_index=1"
        "0001-only-build-the-module-in-dkms.conf.patch")
sha256sums=('SKIP'
            '8cebe7524402bc0b4093d865eabec2062a38a93ca953f8d14b62ffc541932a98'
            '9dc7963c3ac59b0d7ea33ca2aed9dd2e80de0cfb1517df9d028e8e0f2944d3dd'
            'efad66fe48b9cae4809410182ccf4843b052a5938d2f15fae064f97f8ab6e609'
            '86a732705a19b5578aa9df1b4d7a72b94ed26d24238149ba177a1989a8e6e3d4'
            'd9d70f23b57629ede9a360249de4b01592d09b68cd5735865b8f83a09bad3ea6'
            '8d5c31f883a906ab42776dcda79b6c89f904d8f356ade0dab5491578a6af55a5')

prepare() {
    cd "${srcdir}/${_pkgname}"

    msg2 "Staging branch set to ${_git_branch}"

    local -a patches
    patches=($(printf '%s\n' "${source[@]}" | grep -F '.patch'))
    patches=("${patches[@]%%::*}")
    patches=("${patches[@]##*/}")

    for patch in "${patches[@]}"; do
        if patch -p1 -R -i "../$patch" --dry-run -sf >/dev/null; then
            msg2 "Ignoring patch $patch..."
        else
            msg2 "Applying patch $patch..."
            patch -p1 -N -i "../$patch"
        fi
    done

    # remove unneeded sections from module build
    sed -ri "/AC_CONFIG_FILES/,/]\)/{
/AC_CONFIG_FILES/n
/]\)/n
/^\s*(module\/.*|${_pkgname}.release|Makefile)/!d
}" configure.ac
}

pkgver() {
    cd "${srcdir}/${_pkgname}"

    METAVER=$(grep -F Version "${srcdir}/${_pkgname}/META" | tr -d '[:space:]')
    METAVER=${METAVER##*:}
    printf "%s.r%s.g%s" "${METAVER}" "$(git rev-list zfs-${METAVER}..HEAD --count)" "$(git rev-parse --short HEAD)"
}

build() {
    cd "${srcdir}/${_pkgname}"

    # modify META after pkgver() called
    sed -i -e "s/Version:[[:print:]]*/Version:       ${pkgver}/" META
    sed -i -e "s/Release:[[:print:]]*/Release:       ${pkgrel}/" META
    autoreconf -fi

    ./scripts/dkms.mkconf -n ${_pkgname} -v "${pkgver}" -f dkms.conf
    printf '#define\tZFS_META_GITREV "zfs-%s"\n' "${pkgver}" >include/zfs_gitrev.h

}

package() {
    depends=("zfs-utils>=${pkgver%%.r*}" "zfs-utils<=${_staging_ver}" 'dkms')

    cd "${srcdir}/${_pkgname}"

    dkmsdir="${pkgdir}/usr/src/${_pkgname}-${pkgver}"
    install -d "${dkmsdir}"/{config,scripts}
    cp -a configure dkms.conf Makefile.in META ${_pkgname}_config.h.in ${_pkgname}.release.in include/ module/ "${dkmsdir}"/
    cp config/compile config/config.* config/missing config/*sh "${dkmsdir}"/config/
    cp scripts/enum-extract.pl scripts/dkms.postbuild "${dkmsdir}"/scripts/
}
