Rust 环境搭建
============================

介绍
----------------------------
A language empowering everyone
to build reliable and efficient software.


Rust 官网： [`https://rust-lang.org/`](https://rust-lang.org/)

安装
-----------------------------

### Windows
下载安装[`rustup-init.exe`](https://static.rust-lang.org/rustup/dist/x86_64-pc-windows-msvc/rustup-init.exe)

运行后，转到命令行界面
```shell
Rust Visual C++ prerequisites

Rust requires a linker and Windows API libraries but they don't seem to be
available.

These components can be acquired through a Visual Studio installer.

1) Quick install via the Visual Studio Community installer
   (free for individuals, academic uses, and open source).

2) Manually install the prerequisites
   (for enterprise and advanced users).

3) Don't install the prerequisites
   (if you're targeting the GNU ABI).

>
```
这里选择3， 我们不需要安装Visual Studio来提供构建工具。然后：
```shell
The Cargo home directory is located at:

  C:\Users\iotah\.cargo

This can be modified with the CARGO_HOME environment variable.

The cargo, rustc, rustup and other commands will be added to
Cargo's bin directory, located at:

  C:\Users\iotah\.cargo\bin

This path will then be added to your PATH environment variable by
modifying the HKEY_CURRENT_USER/Environment/PATH registry key.

You can uninstall at any time with rustup self uninstall and
these changes will be reverted.

Current installation options:


   default host triple: x86_64-pc-windows-msvc
     default toolchain: stable (default)
               profile: default
  modify PATH variable: yes

1) Proceed with installation (default)
2) Customize installation
3) Cancel installation
```
这里选择1 保持默认就好

安装完成后，打开`powershell`，输入如下命令安装gnu编译工具链
```shell
rustup target add x86_64-pc-windows-gnu
rustup default stable-x86_64-pc-windows-gnu
```

### Linux

### MacOS

学习
-----------------------------

这里我推荐b站杨旭老师的教程
[`https://www.bilibili.com/video/BV1hp4y1k7SV`](https://www.bilibili.com/video/BV1hp4y1k7SV)

笔者建议边看边练，不要眼高手低，Rust是一门需要大量实例练习才能掌握的语言。