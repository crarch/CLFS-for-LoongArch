# ~~手把手~~教你构建基于LoongArch32架构的Linux系统

## 0 前言

总之记录一下构建基于 LoonArch32 Reduced 指令集的方法。

本记录大多数 shell 来自《手把手教你构建基于 LoongArch64 架构的 Linux 系统》。

本记录基于 Ubuntu 系统，笔者是在 WSL Ubuntu 20.04 上进行的。

## 安装软件包

```shell
sudo apt install gdb git autoconf python3-dev python3-pip fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison textinfo make gcc g++ curl
```

## 设置环境变量

```shell
export SYSDIR=~/linux/la32
export BUILDDIR=$SYSDIR/build
export LC_ALL=POSIX
export CROSS_HOST=x86_64
export CROSS_TARGET=loongarch32-unknown-linux-gnu
export MABI="ilp32"
export ARCH=loongarch
export CROSS_COMPILE=$CROSS_TARGET-
export DOWNLOADDIR=~/linux/downloads
```

建议设置为`~/linux/envs.sh`，然后在`bashrc`等`*rc`文件中设置`source ~/linux/envs.sh`。

## 制作交叉编译工具链

这里使用我们 Computer Architecture Research, HITSZCRA 的 la32 port。

### Linux 内核头文件

```shell
pushd ${DOWNLOADDIR}
	git clone https://github.com/crarch/linux -b la32 --depth 1
	pushd ${DOWNLOADDIR}/linux
        make mrproper
        make ARCH=loongarch INSTALL_HDR_PATH=headers headers_install
        find headers/include -name '.*' -delete
        mkdir -pv ${SYSDIR}/sysroot/usr/include
        cp -rv headers/include/* ${SYSDIR}/sysroot/usr/include
    popd
popd
```

### 交叉编译器之 Binutils

```shell
pushd ${DOWNLOADDIR}
	git clone https://github.com/crarch/binutils-gdb -b upstream_v4 --depth 1
	pushd ${DOWNLOADDIR}/binutils-gdb
		./src-release.sh -x gdb
		rm -rf gdb* libdecnumber readline sim
        mkdir -p build
        cd build
        # todo: remove --enable-64-bit-bfd ?
        CC=gcc AR=ar AS=as \
        ../configure --prefix=${SYSDIR}/cross-tools --build=${CROSS_HOST} --host=${CROSS_HOST} \
                     --target=${CROSS_TARGET} --with-sysroot=${SYSDIR}/sysroot --disable-nls \
                     --disable-static --disable-werror --enable-64-bit-bfd
        make configure-host -j
        make -j
        make install-strip -j
        cp -v ../include/libiberty.h ${SYSDIR}/sysroot/usr/include
	popd
popd
```

### GMP

```shell
pushd ${DOWNLOADDIR}
	curl
popd
```

