#### 如何添加一个新的Layer

use bitbake-layers script
```shell
# at poky/
bitbake-layers create-layer meta-hello
```

A example layer will be created like this:
```shell
TODO
```

#### 添加 layer 到项目
```shell
# at poky/build
bitbake-layers add-layer ../meta-hello

cat conf/bblayers.conf
```

#### 添加一个 helloworld 配方到 layer
```shell
# at poky/build
cd ../meta-hellp

# at poky/meta-hello
mkdir -p recipes-hello/hello && cd recipes-hello/hello
mkdir files && cd files
vim hello.c
```

a `hello.c` like this:
```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    printf("Hello, world!\n");
    return 0;
}
```

add a recipe file
```shell
cd ~/poky/meta-hello/recipes-hello/hello
vim hello_1.0.bb
```

a `hello_1.0.bb` like this:
> The md5sum was calculated from common_licenses/MIT
> `poky/meta/files/common_licenses/MIT`
```text
SUMMARY = "Simple helloworld application"
SECTION = "hello"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

TARGET_CC_ARCH += "${LDFLAGS}"

SRC_URI = "file://hello.c"

S = "${WORKDIR}"

do_compile() {
    ${CC} hello.c -o hello
}

do_install() {
    install -d ${D}${bindir}
    install -m 0755 hello ${D}${bindir}
}
```

#### finnally, specify the addition

at `poky/build/conf/local.conf`
add a new line
```
IMAGE_INSTALL:append = " hello" # a whitespace is required before image name
```