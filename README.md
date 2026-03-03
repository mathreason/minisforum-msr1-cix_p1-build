#  ms-r1编译流程

##  编译要求

* ubuntu server 24.04. 理论上debian应该也可以,但没测试过.
* 磁盘可用可用空间在120GB以上. 编译完成后整个文件是97GB,编译过程中还会有很多中间产物.

##  安装依赖

```bash
sudo apt install 7zip android-sdk-libsparse-utils apt-utils base-files bash bash-completion bc bison bsdutils build-essential ca-certificates cpio crossbuild-essential-arm64 curl dash debhelper debianutils debootstrap device-tree-compiler diffutils dosfstools fakeroot findutils flex git grep grub-pc gzip iputils-arping iputils-clockdiff iputils-ping iputils-tracepath kmod libarchive-tools libc-bin libdrm-dev libegl1-mesa-dev libelf-dev libepoxy-dev libgbm-dev libgl1-mesa-dev libgles2-mesa-dev libglib2.0-dev libncurses-dev libssl-dev libwayland-dev libx11-dev libx11-xcb-dev libxcb1-dev libxdamage-dev libxext-dev libxfixes-dev libxrandr-dev linux-generic login lxd-installer lzop mawk mesa-common-dev mtools ncurses-base ncurses-bin net-tools openssh-server parted pkg-config python3 python3-pip python3-ply python3-setuptools python3-wheel qemu-user-static rsync scons swig trace-cmd tree u-boot-tools util-linux uuid-dev uuid-runtime wget xorriso meson
```




## 设置环境变量
* 用于创建编译目录，最好不要使用已存在的目录。可根据实际需要修改目录名，执行构建时不要退出、切换终端

  ```bash
  export builddirname='build' #默认创建在"~/build"，根据实际情况修改值，不影响后续执行
  export builddir=$(realpath ~/$builddirname)
  mkdir -p ${builddir}
  ```



## 下载编译资源

* 如果一次性执行下列所有命令，出错后任会继续执行导致后续构建失败(git clone比较容易出现错误)，建议分段执行或创建成脚本执行。

  ```bash
  mkdir -p ${builddir}
  cd ${builddir}
  mkdir -p ext_debs
  cd ext_debs
  git clone https://github.com/minisforum-cix-p1-repo/ext_debs .
  git checkout a0fb5/5cf6e/cix_p1_mg_dev
  cd ${builddir}
  mkdir -p bsp/uefi/acpica
  cd bsp/uefi/acpica
  git clone https://github.com/minisforum-cix-p1-repo/cix_private__acpica .
  git checkout a0fb5/5cf6e/tag_R2024_12_12
  mkdir -p ${builddir}/bsp/uefi_release/tools/acpica
  rmdir ${builddir}/bsp/uefi_release/tools/acpica
  ln -sf ${builddir}/bsp/uefi/acpica ${builddir}/bsp/uefi_release/tools/acpica
  cd ${builddir}
  mkdir -p bsp/uefi_release/edk2
  cd bsp/uefi_release/edk2
  git clone https://github.com/minisforum-cix-p1-repo/cix_opensource__release__edk2 .
  git checkout a0fb5/5cf6e/cix_p1_mg_mp_dev
  cd ${builddir}
  mkdir -p bsp/uefi_release/edk2-non-osi
  cd bsp/uefi_release/edk2-non-osi
  git clone https://github.com/minisforum-cix-p1-repo/cix_opensource__release__edk2-non-osi .
  git checkout a0fb5/5cf6e/cix_p1_mg_mp_dev
  cd ${builddir}
  mkdir -p bsp/uefi_release/edk2-platforms
  cd bsp/uefi_release/edk2-platforms
  git clone https://github.com/minisforum-cix-p1-repo/cix_opensource__release__edk2-platforms .
  git checkout a0fb5/5cf6e/cix_p1_mg_mp_dev
  cd ${builddir}
  mkdir -p build-scripts
  cd build-scripts
  git clone https://github.com/minisforum-cix-p1-repo/linux_repo__cix_build_scripts .
  git checkout a0fb5/5cf6e/cix_p1_mg_dev
  cd ${builddir}
  mkdir -p component/cix_opensource/chromium/chromium-cix
  cd component/cix_opensource/chromium/chromium-cix
  git clone https://github.com/minisforum-cix-p1-repo/cix_opensource__chromium-cix .
  git checkout a0fb5/5cf6e/cix_p1_mg_dev
  cd ${builddir}
  mkdir -p component/cix_opensource/cix_unit_test
  cd component/cix_opensource/cix_unit_test
  git clone https://github.com/minisforum-cix-p1-repo/cix_opensource__cix_unit_test .
  git checkout a0fb5/5cf6e/cix_p1_mg_dev
  cd ${builddir}
  mkdir -p component/cix_opensource/gpu/gpu_kernel
  cd component/cix_opensource/gpu/gpu_kernel
  git clone https://github.com/minisforum-cix-p1-repo/cix_opensource__gpu_kernel .
  git checkout a0fb5/5cf6e/cix_p1_mg_dev
  cd ${builddir}
  mkdir -p component/cix_opensource/gstreamer
  cd component/cix_opensource/gstreamer
  git clone https://github.com/minisforum-cix-p1-repo/freedesktop_repo__gstreamer__gstreamer .
  git checkout a0fb5/5cf6e/cix_p1_mg_dev
  cd ${builddir}
  mkdir -p component/cix_opensource/npu/npu_driver
  cd component/cix_opensource/npu/npu_driver
  git clone https://github.com/minisforum-cix-p1-repo/cix_opensource__npu_driver .
  git checkout a0fb5/5cf6e/cix_p1_mg_dev
  cd ${builddir}
  mkdir -p component/cix_opensource/os
  cd component/cix_opensource/os
  git clone https://github.com/minisforum-cix-p1-repo/linux_repo__cix_os_src .
  git checkout a0fb5/5cf6e/cix_p1_mg_dev
  cd ${builddir}
  mkdir -p component/cix_opensource/vpu/vpu_driver
  cd component/cix_opensource/vpu/vpu_driver
  git clone https://github.com/minisforum-cix-p1-repo/cix_opensource__vpu_driver .
  git checkout a0fb5/5cf6e/cix_p1_mg_dev
  cd ${builddir}
  mkdir -p component/cix_proprietary
  cd component/cix_proprietary
  git clone https://github.com/minisforum-cix-p1-repo/cix_proprietary__cix_proprietary .
  git checkout a0fb5/5cf6e/cix_p1_mg_dev
  cd ${builddir}
  mkdir -p debian12
  cd debian12
  git clone https://github.com/minisforum-cix-p1-repo/linux_repo__cix_debian12 .
  git checkout a0fb5/5cf6e/cix_p1_mg_dev
  cd ${builddir}
  mkdir -p linux
  cd linux
  git clone https://github.com/minisforum-cix-p1-repo/cix_opensource__linux .
  git checkout a0fb5/5cf6e/cix_p1_mg_dev
  cd ${builddir}
  mkdir -p tools/cix_binary
  cd tools/cix_binary
  git clone https://github.com/minisforum-cix-p1-repo/tools__cix_binary .
  git checkout a0fb5/5cf6e/cix_p1_mg_mp_dev
  cd ${builddir}
  mkdir -p tools/gcc/arm-gnu-toolchain-12.3.rel1-x86_64-aarch64-none-linux-gnu
  cd tools/gcc/arm-gnu-toolchain-12.3.rel1-x86_64-aarch64-none-linux-gnu
  git clone https://github.com/minisforum-cix-p1-repo/cix_opensource__toolchain__gcc .
  git checkout a0fb5/5cf6e/arm-gnu-toolchain-12.3.rel1-x86_64-aarch64-none-linux-gnu
  cd ${builddir}
  mkdir -p tools/gcc/gcc
  cd tools/gcc/gcc
  git clone https://github.com/minisforum-cix-p1-repo/cix_opensource__toolchain__gcc .
  git checkout a0fb5/5cf6e/ubuntu-9.4.0
  cd ${builddir}
  mkdir -p tools/gcc/gcc-arm-10.2-2020.11-x86_64-aarch64-none-elf
  cd tools/gcc/gcc-arm-10.2-2020.11-x86_64-aarch64-none-elf
  git clone https://github.com/minisforum-cix-p1-repo/cix_opensource__toolchain__gcc .
  git checkout a0fb5/5cf6e/arm-10.2-2020.11-x86_64-aarch64-none-elf
  mkdir -p ${builddir}/bsp/uefi_release/tools/gcc/gcc-arm-10.2-2020.11-x86_64-aarch64-none-elf
  rmdir ${builddir}/bsp/uefi_release/tools/gcc/gcc-arm-10.2-2020.11-x86_64-aarch64-none-elf
  ln -sf ${builddir}/tools/gcc/gcc-arm-10.2-2020.11-x86_64-aarch64-none-elf ${builddir}/bsp/uefi_release/tools/gcc/gcc-arm-10.2-2020.11-x86_64-aarch64-none-elf
  cd ${builddir}
  mkdir -p tools/gcc/gcc-arm-none-eabi-10.3-2021.10-x86_64-linux
  cd tools/gcc/gcc-arm-none-eabi-10.3-2021.10-x86_64-linux
  git clone https://github.com/minisforum-cix-p1-repo/cix_opensource__toolchain__gcc .
  git checkout a0fb5/5cf6e/arm-none-eabi-10.3-2021.10-x86_64-linux
  ```

* 下载构建资源包,并解压

  ```bash
  cd ${builddir}
  wget https://github.com/minisforum-cix-p1-repo/ext/releases/download/202510.1/cix_ext.7z.00{1..7}
  7z x ./cix_ext.7z.001
  ```



##  修改内核配置文件(可选)

### 自定义修改
* 根据需要修改内核配置文件 `${builddir}/linux/arch/arm64/configs/cix.config`

  例如添加官方论坛中[AMDGPU](https://github.com/minisforum-docs/MS-R1-Docs/issues/15)、[CIFS](https://github.com/minisforum-docs/MS-R1-Docs/issues/2)、[SQUASHFS_XZ](https://github.com/minisforum-docs/MS-R1-Docs/issues/4) 功能支持

  在 `${builddir}/linux/arch/arm64/configs/cix.config`任意位置添加以下文本

  ```ini
  ############AMDGPU############
  CONFIG_DRM=y
  CONFIG_DRM_KMS_HELPER=y
  CONFIG_DRM_AMD=y
  CONFIG_DRM_AMDGPU=m
  # CONFIG_DRM_RADEON=n #旧radeon驱动
  
  ############SQUASHFS_XZ############
  CONFIG_SQUASHFS_XZ=y
  
  ############CIFS############
  CONFIG_CIFS=m
  CONFIG_CIFS_SMB2=y
  CONFIG_CIFS_ACL=y
  CONFIG_CIFS_XATTR=y
  CONFIG_CIFS_POSIX=y
  CONFIG_CIFS_UPCALL=y
  CONFIG_CIFS_DFS_UPCALL=y
  CONFIG_CIFS_FSCACHE=y
  ```

### 使用预定义内核配置文件

* 使用`kconfig/merge.conf` 内核配置文件. 该文件是官方内核配置文件与debian 12.13内核配置合并后，去除x86、intel、x64、ia32、amd相关配置得出的. 

  `kconfig/merge.conf`配置文件包含了`自定义修改` 中所有配置. 这部分配置在`# --- merged from B: keys absent in A ---` 行之前.

  `kconfig/merge.conf` 配置文件`# --- merged from B: keys absent in A ---`之后的所有行，都为新加的内核配置.

  ```bash
  cp kconfig/merge.conf $builddir/linux/arch/arm64/configs/cix.config
  ```

  

##  构建镜像

* 修改`${builddir}/build-scripts/build-kernel.sh` 的71、72行：

  ```bash
      rm -f "${PATH_ROOT}"/linux-*
      rm -f "${PATH_OUT}"/linux-*
  ```

  修改为

  ```bash
      rm -rf "${PATH_ROOT}"/linux-*
      rm -rf "${PATH_OUT}"/linux-*
  ```

  

* 开始正式编译

  ```bash
  sudo ln -sf /usr/bin/python3 /usr/bin/python
  cd ${builddir}
  source ./build-scripts/envtool.sh #执行这条命令有报错，不用处理，我们主要使用这个脚本初始化环境变量。需要的依赖已经手动安装。
  updateres
  ./build-scripts/build-all.sh -d release -p cix -f debian -h sky1_a0 -b evb -k rsa3072_product -m lkms -t optee -r axi-4G -s 1 -a 1 -x customer -o 1 -l disable -e disable -w open -K none build
  ```



##  获取构建产物

* img镜像位置

  ```bash
  ${builddir}/output/cix_evb/images/linux-fs.sdcard
  ```

  
