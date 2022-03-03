# Maintainer: Philip Müller <philm[at]manjaro[dot]org>
# Maintainer: Bernhard Landauer <bernhard[at]manjaro[dot]org>

# Arch credits:
# Maintainer: Sven-Hendrik Haase <svenstaro@archlinux.org>
# Maintainer: Thomas Baechler <thomas@archlinux.org>
# Contributor: James Rayner <iphitus@gmail.com>

pkgbase=nvidia-utils
pkgname=("nvidia-dkms" "nvidia-utils" "mhwd-nvidia" "opencl-nvidia")
pkgver=510.54
pkgrel=2
arch=('x86_64')
url="http://www.nvidia.com/"
license=('custom')
options=('!strip')
_pkg="NVIDIA-Linux-x86_64-${pkgver}"
source=('10-amdgpu-nvidia-drm-outputclass.conf'
        '10-intel-nvidia-drm-outputclass.conf'
        '90-nvidia-utils.hook'
        'mhwd-nvidia'
        'nvidia-utils.sysusers'
        'nvidia.rules'
        'nvidia.shutdown'
        "https://us.download.nvidia.com/XFree86/Linux-x86_64/${pkgver}/${_pkg}.run")
sha256sums=('3b017d461420874dc9cce8e31ed3a03132a80e057d0275b5b4e1af8006f13618'
            'f57d8e876dd88e6bb7796899f5d45674eb7f99cee16595f34c1bab7096abdeb3'
            'c2396f48835caf7ae60bc17e07eeaf142c8b7074d15d428d6c61d9e38373b8d8'
            'ddffe7033abf38253b50d4c02d780a270f79089bbe163994e00a4d7c91d64f0e'
            'd8d1caa5d72c71c6430c2a0d9ce1a674787e9272ccce28b9d5898ca24e60a167'
            '4fbfd461f939f18786e79f8dba5fdb48be9f00f2ff4b1bb2f184dbce42dd6fc3'
            '402b5549cbc7cc7e3d5d64fdff2c6e3c9d246cbf0066410a698e1457ed749172'
            '4c20deccae3fe347adfd0e6989a306f9024fdadf831adc1e8e60855675335161')

create_links() {
    # create soname links
    find "$pkgdir" -type f -name '*.so*' ! -path '*xorg/*' -print0 | while read -d $'\0' _lib; do
        _soname=$(dirname "${_lib}")/$(readelf -d "${_lib}" | grep -Po 'SONAME.*: \[\K[^]]*' || true)
        _base=$(echo ${_soname} | sed -r 's/(.*)\.so.*/\1.so/')
        [[ -e "${_soname}" ]] || ln -s $(basename "${_lib}") "${_soname}"
        [[ -e "${_base}" ]] || ln -s $(basename "${_soname}") "${_base}"
    done
}

prepare() {
    sh "${_pkg}.run" --extract-only
    cd "${_pkg}"
    bsdtar -xf nvidia-persistenced-init.tar.bz2

    cd kernel

    sed -i "s/__VERSION_STRING/${pkgver}/" dkms.conf
    sed -i 's/__JOBS/`nproc`/' dkms.conf
    sed -i 's/__DKMS_MODULES//' dkms.conf
    sed -i '$iBUILT_MODULE_NAME[0]="nvidia"\
DEST_MODULE_LOCATION[0]="/kernel/drivers/video"\
BUILT_MODULE_NAME[1]="nvidia-uvm"\
DEST_MODULE_LOCATION[1]="/kernel/drivers/video"\
BUILT_MODULE_NAME[2]="nvidia-modeset"\
DEST_MODULE_LOCATION[2]="/kernel/drivers/video"\
BUILT_MODULE_NAME[3]="nvidia-drm"\
DEST_MODULE_LOCATION[3]="/kernel/drivers/video"\
BUILT_MODULE_NAME[4]="nvidia-peermem"\
DEST_MODULE_LOCATION[4]="/kernel/drivers/video"' dkms.conf

    # Gift for linux-rt guys
    sed -i 's/NV_EXCLUDE_BUILD_MODULES/IGNORE_PREEMPT_RT_PRESENCE=1 NV_EXCLUDE_BUILD_MODULES/' dkms.conf
}

package_opencl-nvidia() {
    pkgdesc="OpenCL implemention for NVIDIA"
    depends=('zlib')
    optdepends=('opencl-headers: headers necessary for OpenCL development')
    provides=("opencl-nvidia=$pkgver" 'opencl-driver')
    replaces=('opencl-nvidia-418xx' 'opencl-nvidia-430xx'
              'opencl-nvidia-435xx' 'opencl-nvidia-440xx'
              'opencl-nvidia-450xx' 'opencl-nvidia-455xx'
              'opencl-nvidia-460xx' 'opencl-nvidia-465xx')
    conflicts=('opencl-nvidia-340xx' 'opencl-nvidia-390xx'
               'opencl-nvidia-418xx' 'opencl-nvidia-430xx'
               'opencl-nvidia-435xx' 'opencl-nvidia-440xx'
               'opencl-nvidia-450xx' 'opencl-nvidia-455xx'
               'opencl-nvidia-460xx' 'opencl-nvidia-465xx')
    cd $_pkg

    # OpenCL
    install -Dm644 nvidia.icd "${pkgdir}/etc/OpenCL/vendors/nvidia.icd"
    install -Dm755 "libnvidia-compiler.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-compiler.so.${pkgver}"
    install -Dm755 "libnvidia-opencl.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-opencl.so.${pkgver}"

    create_links

    mkdir -p "${pkgdir}/usr/share/licenses"
    ln -s nvidia-utils "${pkgdir}/usr/share/licenses/opencl-nvidia"
}

package_nvidia-dkms() {
    pkgdesc="NVIDIA drivers - module sources"
    depends=('dkms' "nvidia-utils=$pkgver" 'libglvnd')
    provides=('NVIDIA-MODULE' "nvidia=$pkgver")

    cd ${_pkg}

    install -dm 755 "${pkgdir}"/usr/src
    cp -dr --no-preserve='ownership' kernel "${pkgdir}/usr/src/nvidia-${pkgver}"

    install -Dt "${pkgdir}/usr/share/licenses/${pkgname}" -m644 "${srcdir}/${_pkg}/LICENSE"
}

package_nvidia-utils() {
    pkgdesc="NVIDIA drivers utilities"
    depends=('xorg-server' 'libglvnd' 'egl-wayland' 'jansson' 'gtk3' 'libxv' 'libvdpau' 'libxnvctrl')
    optdepends=('xorg-server-devel: nvidia-xconfig'
                'opencl-nvidia: OpenCL support')
    provides=('vulkan-driver' 'opengl-driver' 'nvidia-libgl' "nvidia-utils=$pkgver")
    replaces=('nvidia-418xx-utils' 'nvidia-430xx-utils' 'nvidia-435xx-utils'
              'nvidia-440xx-utils' 'nvidia-450xx-utils' 'nvidia-455xx-utils')
    conflicts=('nvidia-libgl' 'nvidia-340xx-utils' 'nvidia-390xx-utils'
               'nvidia-418xx-utils' 'nvidia-430xx-utils' 'nvidia-435xx-utils'
               'nvidia-440xx-utils' 'nvidia-450xx-utils' 'nvidia-455xx-utils')
    replaces=('nvidia-libgl')
    install="${pkgname}.install"

    cd "${_pkg}"

    # Check https://us.download.nvidia.com/XFree86/Linux-x86_64/${pkgver}/README/installedcomponents.html
    # for hints on what needs to be installed where.

    # X driver
    install -Dm755 nvidia_drv.so "${pkgdir}/usr/lib/xorg/modules/drivers/nvidia_drv.so"

    # Wayland/GBM
    install -Dm755 libnvidia-egl-gbm.so.1* -t "${pkgdir}/usr/lib/"
    install -Dm644 15_nvidia_gbm.json "${pkgdir}/usr/share/egl/egl_external_platform.d/15_nvidia_gbm.json"
    mkdir -p "${pkgdir}/usr/lib/gbm"
    ln -sr "${pkgdir}/usr/lib/libnvidia-allocator.so.${pkgver}" "${pkgdir}/usr/lib/gbm/nvidia-drm_gbm.so"

    # firmware
    install -Dm644 firmware/gsp.bin "${pkgdir}/usr/lib/firmware/nvidia/${pkgver}/gsp.bin"

    # GLX extension module for X
    install -Dm755 "libglxserver_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so.${pkgver}"
    # Ensure that X finds glx
    ln -s "libglxserver_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so.1"
    ln -s "libglxserver_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so"

    install -Dm755 "libGLX_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libGLX_nvidia.so.${pkgver}"

    # OpenGL libraries
    install -Dm755 "libEGL_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libEGL_nvidia.so.${pkgver}"
    install -Dm755 "libGLESv1_CM_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libGLESv1_CM_nvidia.so.${pkgver}"
    install -Dm755 "libGLESv2_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libGLESv2_nvidia.so.${pkgver}"
    install -Dm644 "10_nvidia.json" "${pkgdir}/usr/share/glvnd/egl_vendor.d/10_nvidia.json"

    # OpenGL core library
    install -Dm755 "libnvidia-glcore.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-glcore.so.${pkgver}"
    install -Dm755 "libnvidia-eglcore.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-eglcore.so.${pkgver}"
    install -Dm755 "libnvidia-glsi.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-glsi.so.${pkgver}"

    # misc
    install -Dm755 "libnvidia-fbc.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-fbc.so.${pkgver}"
    install -Dm755 "libnvidia-encode.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-encode.so.${pkgver}"
    install -Dm755 "libnvidia-cfg.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-cfg.so.${pkgver}"
    install -Dm755 "libnvidia-ml.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-ml.so.${pkgver}"
    install -Dm755 "libnvidia-glvkspirv.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-glvkspirv.so.${pkgver}"
    install -Dm755 "libnvidia-allocator.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-allocator.so.${pkgver}"
    install -Dm755 "libnvidia-vulkan-producer.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-vulkan-producer.so.${pkgver}"
    # Sigh libnvidia-vulkan-producer.so has no SONAME set so create_links doesn't catch it. NVIDIA please fix!
    ln -s "libnvidia-vulkan-producer.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-vulkan-producer.so.1"
    ln -s "libnvidia-vulkan-producer.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-vulkan-producer.so"

    # Vulkan ICD
    install -Dm644 "nvidia_icd.json" "${pkgdir}/usr/share/vulkan/icd.d/nvidia_icd.json"
    install -Dm644 "nvidia_layers.json" "${pkgdir}/usr/share/vulkan/implicit_layer.d/nvidia_layers.json"

    # VDPAU
    install -Dm755 "libvdpau_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/vdpau/libvdpau_nvidia.so.${pkgver}"

    # nvidia-tls library
    install -Dm755 "libnvidia-tls.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-tls.so.${pkgver}"

    # CUDA
    install -Dm755 "libcuda.so.${pkgver}" "${pkgdir}/usr/lib/libcuda.so.${pkgver}"
    install -Dm755 "libnvcuvid.so.${pkgver}" "${pkgdir}/usr/lib/libnvcuvid.so.${pkgver}"

    # PTX JIT Compiler (Parallel Thread Execution (PTX) is a pseudo-assembly language for CUDA)
    install -Dm755 "libnvidia-ptxjitcompiler.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-ptxjitcompiler.so.${pkgver}"

    # raytracing
    install -Dm755 "libnvoptix.so.${pkgver}" "${pkgdir}/usr/lib/libnvoptix.so.${pkgver}"
    install -Dm755 "libnvidia-rtcore.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-rtcore.so.${pkgver}"

    # NGX
    install -Dm755 nvidia-ngx-updater "${pkgdir}/usr/bin/nvidia-ngx-updater"
    install -Dm755 "libnvidia-ngx.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-ngx.so.${pkgver}"
    install -Dm755 _nvngx.dll "${pkgdir}/usr/lib/nvidia/wine/_nvngx.dll"
    install -Dm755 nvngx.dll "${pkgdir}/usr/lib/nvidia/wine/nvngx.dll"

    # Optical flow
    install -Dm755 "libnvidia-opticalflow.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-opticalflow.so.${pkgver}"

    # DEBUG
    install -Dm755 nvidia-debugdump "${pkgdir}/usr/bin/nvidia-debugdump"

    # nvidia-xconfig
    install -Dm755 nvidia-xconfig "${pkgdir}/usr/bin/nvidia-xconfig"
    install -Dm644 nvidia-xconfig.1.gz "${pkgdir}/usr/share/man/man1/nvidia-xconfig.1.gz"

    # nvidia-bug-report
    install -Dm755 nvidia-bug-report.sh "${pkgdir}/usr/bin/nvidia-bug-report.sh"

    # nvidia-smi
    install -Dm755 nvidia-smi "${pkgdir}/usr/bin/nvidia-smi"
    install -Dm644 nvidia-smi.1.gz "${pkgdir}/usr/share/man/man1/nvidia-smi.1.gz"

    # nvidia-cuda-mps
    install -Dm755 nvidia-cuda-mps-server "${pkgdir}/usr/bin/nvidia-cuda-mps-server"
    install -Dm755 nvidia-cuda-mps-control "${pkgdir}/usr/bin/nvidia-cuda-mps-control"
    install -Dm644 nvidia-cuda-mps-control.1.gz "${pkgdir}/usr/share/man/man1/nvidia-cuda-mps-control.1.gz"

    # nvidia-modprobe
    # This should be removed if nvidia fixed their uvm module!
    install -Dm4755 nvidia-modprobe "${pkgdir}/usr/bin/nvidia-modprobe"
    install -Dm644 nvidia-modprobe.1.gz "${pkgdir}/usr/share/man/man1/nvidia-modprobe.1.gz"

    # nvidia-persistenced
    install -Dm755 nvidia-persistenced "${pkgdir}/usr/bin/nvidia-persistenced"
    install -Dm644 nvidia-persistenced.1.gz "${pkgdir}/usr/share/man/man1/nvidia-persistenced.1.gz"
    install -Dm644 nvidia-persistenced-init/systemd/nvidia-persistenced.service.template "${pkgdir}/usr/lib/systemd/system/nvidia-persistenced.service"
    sed -i 's/__USER__/nvidia-persistenced/' "${pkgdir}/usr/lib/systemd/system/nvidia-persistenced.service"

    # application profiles
    install -Dm644 nvidia-application-profiles-${pkgver}-rc "${pkgdir}/usr/share/nvidia/nvidia-application-profiles-${pkgver}-rc"
    install -Dm644 nvidia-application-profiles-${pkgver}-key-documentation "${pkgdir}/usr/share/nvidia/nvidia-application-profiles-${pkgver}-key-documentation"

    install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/nvidia-utils/LICENSE"
    install -Dm644 README.txt "${pkgdir}/usr/share/doc/nvidia/README"
    install -Dm644 NVIDIA_Changelog "${pkgdir}/usr/share/doc/nvidia/NVIDIA_Changelog"
    cp -r html "${pkgdir}/usr/share/doc/nvidia/"
    ln -s nvidia "${pkgdir}/usr/share/doc/nvidia-utils"

    # new power management support
    install -Dm644 systemd/system/*.service -t "${pkgdir}/usr/lib/systemd/system"
    install -Dm755 systemd/system-sleep/nvidia "${pkgdir}/usr/lib/systemd/system-sleep/nvidia"
    install -Dm755 systemd/nvidia-sleep.sh "${pkgdir}/usr/bin/nvidia-sleep.sh"
    install -Dm755 nvidia-powerd "${pkgdir}/usr/bin/nvidia-powerd"
    install -Dm644 nvidia-dbus.conf "${pkgdir}"/usr/share/dbus-1/system.d/nvidia-dbus.conf

    # distro specific files must be installed in /usr/share/X11/xorg.conf.d
    install -Dm644 "$srcdir/10-amdgpu-nvidia-drm-outputclass.conf" "$pkgdir/usr/share/X11/xorg.conf.d/10-amdgpu-nvidia-drm-outputclass.conf"
    install -Dm644 "$srcdir/10-intel-nvidia-drm-outputclass.conf" "$pkgdir/usr/share/X11/xorg.conf.d/10-intel-nvidia-drm-outputclass.conf"

    install -Dm644 "${srcdir}/nvidia-utils.sysusers" "${pkgdir}/usr/lib/sysusers.d/$pkgname.conf"

    install -Dm644 "${srcdir}/nvidia.rules" "$pkgdir"/usr/lib/udev/rules.d/60-nvidia.rules

    echo "blacklist nouveau" | install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modprobe.d/${pkgname}.conf"
    echo "nvidia-uvm" | install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modules-load.d/${pkgname}.conf"

    # nvidia-settings
    install -Dm755 nvidia-settings "${pkgdir}/usr/bin/nvidia-settings"
    install -Dm644 nvidia-settings.1.gz "${pkgdir}/usr/share/man/man1/nvidia-settings.1.gz"
    install -Dm644 nvidia-settings.desktop "${pkgdir}/usr/share/applications/nvidia-settings.desktop"
    install -Dm644 nvidia-settings.png "${pkgdir}/usr/share/pixmaps/nvidia-settings.png"
    install -Dm755 "libnvidia-gtk3.so.$pkgver" "$pkgdir/usr/lib/libnvidia-gtk3.so.$pkgver"
    sed \
        -e 's:__UTILS_PATH__:/usr/bin:' \
        -e 's:__PIXMAP_PATH__:/usr/share/pixmaps:' \
        -e 's/__NVIDIA_SETTINGS_DESKTOP_CATEGORIES__/Settings;HardwareSettings;/' \
        -e 's/Icon=.*/Icon=nvidia-settings/' \
        -i "${pkgdir}/usr/share/applications/nvidia-settings.desktop"

    # install fix for oldroot unmount
    install -Dm755 "${srcdir}/nvidia.shutdown" "${pkgdir}/usr/lib/systemd/system-shutdown/nvidia.shutdown"

    # install alpm hook
    install -Dm644 "$srcdir/90-nvidia-utils.hook" "$pkgdir/usr/share/libalpm/hooks/90-nvidia-utils.hook"

    create_links
}

package_mhwd-nvidia() {
    pkgdesc="MHWD module-ids for nvidia $pkgver"
    arch=('any')
    replaces=('mhwd-nvidia-418xx' 'mhwd-nvidia-430xx'
              'mhwd-nvidia-435xx' 'mhwd-nvidia-440xx'
              'mhwd-nvidia-450xx' 'mhwd-nvidia-455xx'
              'mhwd-nvidia-460xx' 'mhwd-nvidia-465xx')

    install -d "$pkgdir/var/lib/mhwd/ids/pci/"

    # Generate mhwd database
    sh -e $srcdir/mhwd-nvidia \
    $srcdir/$_pkg/README.txt \
    $srcdir/$_pkg/kernel/nvidia/nv-kernel.o_binary \
    > $pkgdir/var/lib/mhwd/ids/pci/nvidia.ids
    # add PCIID: 1b82 Nvidia Gforce 1070 Ti
    sed -i 's/1b81 1b84/1b81 1b82 1b84/g' $pkgdir/var/lib/mhwd/ids/pci/nvidia.ids
}
