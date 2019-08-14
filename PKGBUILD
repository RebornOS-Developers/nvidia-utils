# Maintainer: Philip Müller <philm[at]manjaro[dog]org>
# Contributor: Sven-Hendrik Haase <sh@lutzhaase.com>
# Contributor: Thomas Baechler <thomas@archlinux.org>
# Contributor: James Rayner <iphitus@gmail.com>
# Contributor: Aaron Plattner <aplattner@nvidia.com>
pkgbase=nvidia-utils
pkgname=('nvidia-utils' 'mhwd-nvidia' 'opencl-nvidia')
pkgver=435.17
pkgrel=2
epoch=1
arch=('x86_64')
url="http://www.nvidia.com/"
license=('custom')
options=('!strip')
source=('mhwd-nvidia' 'nvidia-drm-outputclass.conf' 'nvidia-utils.sysusers')
source_x86_64=("http://download.nvidia.com/XFree86/Linux-x86_64/${pkgver}/NVIDIA-Linux-x86_64-${pkgver}-no-compat32.run")
sha256sums=('11176f1c070bbdbfaa01a3743ec065fe71ff867b9f72f1dce0de0339b5873bb5'
            'adb89c9ecc7027fceaf6e7af16c7715fe85aa900b3dc2bbb71f64b9d63caa1b8'
            'd8d1caa5d72c71c6430c2a0d9ce1a674787e9272ccce28b9d5898ca24e60a167')
sha256sums_x86_64=('1d5e23663c8730f6c8035debe728a18da112e3d0a12a859f76e0b16132c33162')

_pkg="NVIDIA-Linux-x86_64-${pkgver}-no-compat32"

prepare() {
    sh "${_pkg}.run" --extract-only
    cd "${_pkg}"
    bsdtar -xf nvidia-persistenced-init.tar.bz2
}

process_manifest () {
    declare -A type_handlers=(
        # opencl-nvidia
        ["CUDA_ICD"]="opencl-nvidia install_opencl_vendor"
        ["OPENCL_LIB"]="opencl-nvidia install_lib"
        ["OPENCL_LIB_SYMLINK"]="opencl-nvidia symlink_lib_with_path"

        # nvidia-utils
        ["APPLICATION_PROFILE"]="nvidia-utils install_app_profile"
        ["CUDA_LIB"]="nvidia-utils install_lib_with_path"
        ["CUDA_SYMLINK"]="nvidia-utils symlink_lib_with_path"
        ["DOCUMENTATION"]="nvidia-utils install_doc"
        ["DOT_DESKTOP"]="nvidia-utils install_dot_desktop"
        ["ENCODEAPI_LIB"]="nvidia-utils install_lib"
        ["ENCODEAPI_LIB_SYMLINK"]="nvidia-utils symlink_lib"
        ["GLVND_EGL_ICD_JSON"]="nvidia-utils install_egl_json"
        ["GLX_MODULE_SHARED_LIB"]="nvidia-utils install_glx_module"
        ["GLX_MODULE_SYMLINK"]="nvidia-utils symlink_glx_module"
        ["MANPAGE"]="nvidia-utils install_man"
        ["NVCUVID_LIB"]="nvidia-utils install_lib"
        ["NVCUVID_LIB_SYMLINK"]="nvidia-utils symlink_lib"
        ["NVIDIA_MODPROBE_MANPAGE"]="nvidia-utils install_man"
        ["NVIDIA_MODPROBE"]="nvidia-utils install_bin"
        ["NVIFR_LIB"]="nvidia-utils install_lib"
        ["NVIFR_LIB_SYMLINK"]="nvidia-utils symlink_lib"
        ["OPENGL_LIB"]="nvidia-utils install_lib"
        ["OPENGL_SYMLINK"]="nvidia-utils symlink_lib"
        ["TLS_LIB"]="nvidia-utils install_lib"
        ["UTILITY_BINARY"]="nvidia-utils install_bin"
        ["UTILITY_LIB"]="nvidia-utils install_lib"
        ["UTILITY_LIB_SYMLINK"]="nvidia-utils symlink_lib"
        ["VDPAU_LIB"]="nvidia-utils install_lib_with_path"
        ["VDPAU_SYMLINK"]="nvidia-utils symlink_lib_with_path"
        ["VULKAN_ICD_JSON"]="nvidia-utils install_vulkan_json"
        ["XMODULE_SHARED_LIB"]="nvidia-utils install_x_driver"
        ["XORG_OUTPUTCLASS_CONFIG"]="nvidia-utils install_x_config"

        # Ignored entries
        ["DKMS_CONF"]="ignored"                 # dkms isn't needed with Arch's version-locked packages
        #["DOT_DESKTOP"]="ignored"               # Use the separate Arch nvidia-settings package.
        ["EGL_CLIENT_LIB"]="ignored"            # provided by libglvnd
        ["EGL_CLIENT_SYMLINK"]="ignored"        # provided by libglvnd
        ["GLVND_LIB"]="ignored"                 # provided by libglvnd
        ["GLVND_SYMLINK"]="ignored"             # provided by libglvnd
        ["GLX_CLIENT_LIB"]="ignored"            # provided by libglvnd
        ["GLX_CLIENT_SYMLINK"]="ignored"        # provided by libglvnd
        ["INSTALLER_BINARY"]="ignored"          # provided by pacman :)
        ["KERNEL_MODULE_SRC"]="ignored"         # kernel modules are handled by the nvidia PKGBUILD
        ["LIBGL_LA"]="ignored"                  # .la files are not needed
        ["OPENCL_WRAPPER_LIB"]="ignored"        # provided by libcl
        ["OPENCL_WRAPPER_SYMLINK"]="ignored"    # provided by libcl
        ["OPENGL_HEADER"]="ignored"             # provided by mesa
        ["UTILITY_BIN_SYMLINK"]="ignored"       # provided by pacman
        ["UVM_MODULE_SRC"]="ignored"            # kernel modules are handled by the nvidia PKGBUILD
        ["VDPAU_WRAPPER_LIB"]="ignored"         # provided by libvdpau
        ["VDPAU_WRAPPER_SYMLINK"]="ignored"     # provided by libvdpau
        ["XMODULE_NEWSYM"]="ignored"            # not needed for modern X servers
        ["XMODULE_SYMLINK"]="ignored"           # not needed for modern X servers
        ["EGL_EXTERNAL_PLATFORM_JSON"]="ignored" # TODO: Install this somewhere
    )

    tail -n +9 .manifest | {
        while read name perm type extra1 extra2 extra3; do
            if [ ${type_handlers[${type}]+isset} ]; then
                local data=( ${type_handlers[${type}]} )
                local pkg=${data[0]}
                local handler=${data[1]}

                # Check if this file type is handled by the current package.
                # Otherwise, ignore it.
                if [ ${pkg} = ${pkgname} ]; then
                    eval ${handler} $name $perm $type $extra1 $extra2 $extra3
                fi
            else
                echo "Unrecognized manifest type ${type}"
                exit 1
            fi
        done
    }

    mkdir -p "${pkgdir}/usr/share/licenses"
    ln -s nvidia "${pkgdir}/usr/share/licenses/${pkgname}"
}

install_app_profile()   { install -D -m$2 "$1" "${pkgdir}/usr/share/nvidia/$1"; }
install_bin()           { install -D -m$2 "$1" "${pkgdir}/usr/bin/$1"; }
install_egl_json()      { install -D -m$2 "$1" "${pkgdir}/usr/share/glvnd/egl_vendor.d/$1"; }
install_glx_module()    { install -D -m$2 "$1" "${pkgdir}/usr/lib/xorg/modules/extensions/$1"; }
install_lib()           { install -D -m$2 "$1" "${pkgdir}/usr/lib/$1"; }
install_lib_with_path() { install -D -m$2 "$1" "${pkgdir}/usr/lib/$5$1"; }
install_man()           { install -D -m$2 "$1" "${pkgdir}/usr/share/man/man1/$1"; }
install_opencl_vendor() { install -D -m$2 "$1" "${pkgdir}/etc/OpenCL/vendors/$1"; }
install_vulkan_json()   { install -D -m$2 "$1" "${pkgdir}/etc/vulkan/$4$1"; }
install_x_config()      { install -D -m$2 "$1" "${pkgdir}/usr/share/X11/xorg.conf.d/$1"; }


#install_bin() {
#    case "$1" in
#        nvidia-settings)
#            # Use the separate Arch nvidia-settings package.
#            ;;
#        *)
#            install -D -m$2 "$1" "${pkgdir}/usr/bin/$1"
#            ;;
#    esac
#}

#install_lib() {
#    case "$1" in
#        libnvidia-gtk*)
#            # Use the separate Arch nvidia-settings package.
#            ;;
#        *)
#            install -D -m$2 "$1" "${pkgdir}/usr/lib/$1"
#            ;;
#    esac
#}

#install_man() {
#    case "$1" in
#        nvidia-settings.1.gz|nvidia-installer.1.gz)
#            # Skip manpages for utilities that are not packaged.
#            ;;
#        *)
#            install -D -m$2 "$1" "${pkgdir}/usr/share/man/man1/$1"
#            ;;
#    esac
#}

install_x_driver()      {
    case "$1" in
        libnvidia-wfb*)
            # not needed for modern X servers
            ;;
        *)
            install -D -m$2 "$1" "${pkgdir}/usr/lib/xorg/modules/$4$1";
            ;;
    esac
}

install_doc() {
    # Strip the historical NVIDIA_GLX-1.0 prefix off of the target path and
    # "html" off the source path.
    local src=$(basename $1)
    local target=${4#NVIDIA_GLX-1.0/}
    install -D -m$2 "$1" "${pkgdir}/usr/share/doc/nvidia/${target}/${src}"
}

install_dot_desktop()   {
    install -D -m$2 "$1" "${pkgdir}/usr/share/applications/$1"

    # Set the appropriate paths in the .desktop file
    sed -i -e s:__UTILS_PATH__:/usr/bin: \
           -e s:__PIXMAP_PATH__:/usr/share/doc/nvidia: \
           "${pkgdir}/usr/share/applications/$1"
}

symlink_glx_module()    { ln -s "$5" "${pkgdir}/usr/lib/xorg/modules/extensions/$1"; }
symlink_lib()           {
    if [ "$1" != libGLX_indirect.so.0 ]; then
        ln -s "$5" "${pkgdir}/usr/lib/$1"
    fi
}
symlink_lib_with_path() { ln -s "$6" "${pkgdir}/usr/lib/$5$1"; }

package_opencl-nvidia() {
    pkgdesc="OpenCL implemention for NVIDIA"
    depends=('zlib')
    optdepends=('opencl-headers: headers necessary for OpenCL development')
    provides=("opencl-nvidia=$pkgver" 'opencl-driver')
    cd "${_pkg}"

    process_manifest
}

package_mhwd-nvidia() {
    pkgdesc="MHWD module-ids for nvidia $pkgver"
    arch=('any')

    install -d -m755 "${pkgdir}/var/lib/mhwd/ids/pci/"

    # Generate mhwd database
    sh -e ${srcdir}/mhwd-nvidia \
    ${srcdir}/${_pkg}/README.txt \
    ${srcdir}/${_pkg}/kernel/nvidia/nv-kernel.o_binary \
    > ${pkgdir}/var/lib/mhwd/ids/pci/nvidia.ids
    # add PCIID: 1b82 Nvidia Gforce 1070 Ti
    sed -i 's/1b81 1b84/1b81 1b82 1b84/g' ${pkgdir}/var/lib/mhwd/ids/pci/nvidia.ids
}

package_nvidia-utils() {
    pkgdesc="NVIDIA drivers utilities"
    depends=('xorg-server')
    optdepends=('gtk3: nvidia-settings'
                'xorg-server-devel: nvidia-xconfig'
                'opencl-nvidia: OpenCL support')
    provides=('vulkan-driver' 'opengl-driver' 'nvidia-libgl' "nvidia-utils=$pkgver")
    conflicts=('nvidia-libgl')
    replaces=('nvidia-libgl')
    install="${pkgname}.install"
    cd "${_pkg}"

    process_manifest

    install -D -m644 LICENSE "${pkgdir}/usr/share/licenses/nvidia/LICENSE"
    ln -s nvidia "${pkgdir}/usr/share/doc/nvidia-utils"

    # new power management support
    install -D -m644 nvidia-suspend.service   -t "${pkgdir}/usr/lib/systemd/system"
    install -D -m644 nvidia-hibernate.service -t "${pkgdir}/usr/lib/systemd/system"
    install -D -m644 nvidia-resume.service    -t "${pkgdir}/usr/lib/systemd/system"
    install -D -m755 nvidia                   -t "${pkgdir}/usr/lib/systemd/system-sleep"
    install -D -m755 nvidia-sleep.sh          -t "${pkgdir}/usr/bin"

    # distro specific files must be installed in /usr/share/X11/xorg.conf.d
    install -m755 -d "${pkgdir}/usr/share/X11/xorg.conf.d"
    install -m644 "${srcdir}/nvidia-drm-outputclass.conf" "${pkgdir}/usr/share/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf"
    install -Dm644 "${srcdir}/nvidia-utils.sysusers" "${pkgdir}/usr/lib/sysusers.d/$pkgname.conf"
}
