+++
title = "NixOS 的安全启动配置"
date = 2023-01-06T09:02:22+08:00
cover = "https://s2.loli.net/2023/01/06/L5RrymTi24AEnf8.png"
tags = ["nixos", "secureboot"]
keywords = ["nixos", "secureboot"]
description = "搭配 TPM 的完整安全启动体验！"
+++

在三周之前，`bootspec` 作为一个实验性 RFC [合并到了 nixpkgs 中](https://github.com/NixOS/nixpkgs/pull/172237/)。这宣告 NixOS 的启动器进入了一个新的时代——无需黑魔法，即可配置安全启动了！

这个 RFC 还处于早期测试阶段。尽管已经并入官方主线，你依然需要一个[额外设置一个 flag](https://github.com/NixOS/nixpkgs/pull/172237/files#diff-a44165d56783b4b5ce64115fcab496a6513d177ec9c2c6172cb9abc2a9fdced1) 来声明使用它：

```shell
boot.bootspec.enable = true
```

如果开启成功，在你下次 `nixos-rebuild` 时，控制台会打出这么一串警告：

```shell
trace: warning: RFC-0125 is not merged yet, this is a feature preview of bootspec.
        The schema is not definitive and features are not guaranteed to be stable until RFC-0125 is merged.
        See:
        - https://github.com/NixOS/nixpkgs/pull/172237 to track merge status in nixpkgs.
        - https://github.com/NixOS/rfcs/pull/125 to track RFC status.

```

不必担心，这是正常的例行警告。

## 配置安全启动

事实上，`bootspec` 附带一个 `secureboot` 的实现。不过，我在实际测试时，始终无法正常签发启动文件，遂放弃并寻找替代品。

之后，我发现了一个近乎完美的替代品：[`lanzaboote`](https://github.com/nix-community/lanzaboote)。这是一个[诞生不久](https://github.com/nix-community/lanzaboote/commit/cd39fd3a6b016a6e28dca8df4fdfa319cdda2703)的新生项目，旨在为 NixOS 提供安全启动支持。

以下配置步骤假定你使用 flake。

### 开启 Setup Mode
进入你的 BIOS 设置页面，将安全启动模式从 ```User Mode``` 改为 ```Setup Mode```。保存配置并重启到 NixOS。

### 引入 lanzaboote
首先，你需要在 inputs 中引入 `lanzaboote`：

```shell
...
lanzaboote.url = "github:nix-community/lanzaboote";
lanzaboote.inputs.nixpkgs.follows = "nixos";
...
```

并在 outputs 中的 modules 里引用它：

```shell
modules = [
...
lanzaboote.nixosModules.lanzaboote
...
]
```

### 生成安全密钥
引入完成后，你还需要创建安全密钥。输入以下指令即可生成一套：
```shell
sbctl create-keys
```

安全密钥默认生成在 ```/etc/secureboot``` 里。

### 注册安全密钥
输入以下指令，将刚生成的安全密钥注册到固件内：
```shell
sbctl enroll-keys --microsoft
```

上述指令假定需要引入 Windows 的安全启动证书。如需 NixOS + Windows 双启动，这是必要的。反之：

```shell
sbctl enroll-keys --yes-this-might-brick-my-machine
```

即可只注册自己生成的安全密钥。

### 引入配置文件
> 你不需要手动添加 bootspec 的实验标志。lanzaboote 会自动开启它。
```shell
  boot.lanzaboote = {
    enable = true;
    publicKeyFile = "/etc/secureboot/keys/db/db.pem"; # DB public key
    privateKeyFile = "/etc/secureboot/keys/db/db.key"; # DB private key
  };
```

### 生成统一启动镜像
重新构建 NixOS 系统。如果一切顺利，你将在 EFI/Linux 下找到签名好的系统镜像。下次启动时，选择它即可安全启动 NixOS。

> 警告：请务必使用 ```sbctl verify``` 验证你的启动文件签名情况。以下几个文件必须拥有正确的签名，才能让 NixOS 安全启动：
> - EFI/Boot/BOOTX64.EFI
> - EFI/Linux/nixos-generation-xx.efi
> - EFI/systemd/systemd-bootx64.efi
>
> 在特定情况下，```lanzaboote``` 不会对上面的某些文件进行签名！**这会导致你的 NixOS 无法安全启动！** 这是一个[已知问题](https://github.com/nix-community/lanzaboote/issues/39)。你可以在重建 NixOS 系统前删除以上文件，来规避这一问题。

## 使用 TPM2
如果你在使用 ```LUKS``` 加密分区，或许会对如何使用 ```TPM``` 自动解锁某些分区感兴趣。

### 使用 systemd 作为引导加载程序
> 事实上，你也可以将 ```clevis``` 插入到 [preLVM](https://search.nixos.org/options?channel=unstable&show=boot.initrd.preLVMCommands&from=0&size=50&sort=relevance&type=packages&query=preLVM) 配置中，以实现 shell script 下自动解锁特定分区的效果。不过，出于性能考虑，本文采用基于 ```systemd``` 的方案

你需要将以下配置添加到配置文件中：
```shell
boot.initrd.systemd.enable = true;
```

以启动基于 systemd 的引导程序。

### 引入内核模块
添加类似于下面的配置：
```shell
boot.initrd.kernelModules = [ "tpm" "tpm_tis" "tpm_crb"]
```

请依据自己的实际情况，按需引入 TPM 内核模块。

### 向 TPM 注册密钥
> 以下内容主要来自于 [ArchLinux Wiki](https://wiki.archlinux.org/title/TPM)

在终端里执行以下命令：
```
systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+7 --tpm2-with-pin=true /dev/sdX
```

其中，```--tpm2-with-pin``` 要求在 TPM 自动解锁前输入一串密码，而不是直接解锁。这可以有限的提升安全性。如果你不需要，请移除它。

> 注意：该密码虽然叫做 PIN，但其实可以包含字母等不属于数字的内容

大功告成！构建并重启你的 NixOS，即可享受安全启动的安全性，以及自动解锁的便利性。