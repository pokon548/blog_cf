+++
title = "真正轻量的虚拟机 —— 基于 MicroVM，利用 virtiofs 复用基础系统设施"
date = "2023-09-23T17:46:25+08:00"
author = ""
authorTwitter = "" #do not include @
cover = "https://s2.loli.net/2023/09/23/RK4icfPu7j5BMog.png"
tags = ["virtual machine", "nixos", "lightweight"]
keywords = ["virtual machine", "nixos", "lightweight"]
description = "让你的虚拟机和 Windows™ Sandbox 一样轻量！"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

长话短说：我自己有分离个人与工作环境的习惯，在手机上，我利用 Android 自带的工作空间功能，将两环境进行分离，以更好的服务于我自己的需求。但是，Linux 本身并没有自带类似的机制，可以让用户在同一个界面下，展示来自不同用户的程序[^1]。这让我开始尝试实现与之类似的一种机制。

## 一些考虑过，但最终不采纳的备选方案

- 传统意义上的虚拟机。利用 KVM 开一个传统意义上的虚拟机，装一个系统，然后开始用。这样做的好处自不必说，便是完全的与主机隔离，而且操作较为简单。但这种方案在后续维护上略显繁琐，因为这意味着我要单独维护一个系统，会增加额外的精力负担[^2]；
- 创建多个用户。传统意义上的多用户，但无法在 Wayland 下满足同屏展示不同用户程序的需求[^3]，只能在使用时手动切换用户；

## 轻量级虚拟机 —— MicroVM

之后，我在某群群友的提及下，接触到了一个 `NixOS` 下的声明式虚拟机配置项目 —— [NixOS MicroVM](https://github.com/astro/microvm.nix)。

参考文档，发现它可以利用 `virtiofs`，将主机上的 `NixOS Store`映射到虚拟机内[^4]，这样做，就可以复用主机的基本 `NixOS` 设施，做到：

- 避免每次 `rebuild` 系统时，构建巨大的系统映像到系统里，减少构建时长；
- 系统软件跟随主机一同更新，大幅度降低传统虚拟机的维护成本；

其还支持各种虚拟化引擎，包括：

- qemu
- cloud-hypervisor
- firecracker
- crosvm
- kvmtool
- stratovirt

> 需要留意的是，如果你和我一样，需要在虚拟机里用到图形化界面，或许底层虚拟化引擎的最佳选择是 qemu，因为：
>
> - cloud-hypervisor 似乎无法正常编译出所需的固件，但或许是我的配置问题，你可以大胆尝试一下，然后在评论区表达你的观点 :)
> - crosvm 的图形化支持是损坏的，你可能无法运行 `electron` 与 `xorg` 类应用[^5]。
>
> 其它的虚拟化方案不支持 `virtiofs shares`，故跳过测试。

## 一个可以开机的最小化配置
> 这里假定你已经将 `MicroVM` 集成到 `NixOS` 内，并精通 `NixOS` 的语法

如果你只是想简单的尝试一下效果，可以这么写配置：
```nix
  microvm.vms = {
    examplevm = {
      config = {
        microvm = {
          hypervisor = "qemu";
          graphics.enable = true;
          shares = [
            {
              source = "/nix/store";
              mountPoint = "/nix/.ro-store";
              tag = "ro-store";
            }
          ];
        };

        # TODO: Add custom config below, just treated as new machine in NixOS :)
      };
    };
  };
```

这不是一份完整的配置，你需要在 `TODO` 下面继续写上你期望的 `NixOS` 配置。如有疑问，可参考 [官方说明](https://github.com/astro/microvm.nix)。

> 运行配置好的虚拟机的方法也写在官方说明内，故这里不进行重复说明。请参考上面的 [官方说明](https://github.com/astro/microvm.nix)，以测试你的虚拟机配置

## 添加持久层
`MicroVM` 在上述配置下，并不会保留你在虚拟机里的任何变更。换言之，你的所有更改都会在重启之后丢失。

如果你只是想把虚拟机当成 `Windows™ Sandbox` 来用，那么你可以跳过这一节。不过，如果你和我一样，需要把它当成一个经典的虚拟机来用，就需要想办法留存虚拟机的变更。这在 `MicroVM` 中很容易实现，只需要挂载一个持久层即可，比如下面的配置，就让 `/` 下的所有变更都保留下来。持久层大小设置为 `8GB`：
```nix
  microvm.vms = {
    examplevm = {
      config = {
        microvm = {
          hypervisor = "qemu";
          graphics.enable = true;
          shares = [
            {
              source = "/nix/store";
              mountPoint = "/nix/.ro-store";
              tag = "ro-store";
            }
          ];

          # NEW! Persist all changes in /
          volumes = [
            {
              image = "root-overlay.img";
              mountPoint = "/";
              size = 8192;
            }
          ];
        };

        # TODO: Add custom config below, just treated as new machine in NixOS :)
      };
    };
  };
```

应用完成之后，别忘记重启虚拟机。之后，你就惊奇的发现，系统不会再因为重启而丢失所有的变更了[^6]！

## 在虚拟机里 rebuild
想要在虚拟机里运行 `nixos-rebuild`？基于上面持久层挂载的基础上，这同样很容易实现，只需要额外指定 `writable store` 即可：

```nix
  microvm.vms = {
    examplevm = {
      config = {
        microvm = {
          hypervisor = "qemu";
          graphics.enable = true;
          shares = [
            {
              source = "/nix/store";
              mountPoint = "/nix/.ro-store";
              tag = "ro-store";
            }
          ];
          # NEW! Delcare writable store overlay
          writableStoreOverlay = "/nix/.rw-store";
          volumes = [
            {
              image = "root-overlay.img";
              mountPoint = "/";
              size = 8192;
            }
          ];
        };

        # TODO: Add custom config below, just treated as new machine in NixOS :)
      };
    };
  };
```

## 声音呢？我需要声音！
在一番测试之后，我发现似乎只有 `pipewire` 后端是可以让 `QEMU` 正常播放出声音的。这里假定你系统使用的声音系统为 `pipewire`，而且是在系统层面上安装了它[^7]。

> 不会配置 `pipewire` 吗？你可以参考我的 [配置](https://github.com/pokon548/dotfiles/blob/main/nixos/modules/pipewire.nix)！

之后，在配置里加上这些自定义 `QEMU` 参数：
```nix
  microvm.vms = {
    examplevm = {
      config = {
        microvm = {
          hypervisor = "qemu";
          graphics.enable = true;
          # NEW! Add soundcard devices to QEMU
          qemu.extraArgs = [
            "-audiodev"
            "pipewire,id=auddev0"
            "-device"
            "intel-hda"
            "-device"
            "hda-output,audiodev=auddev0"
          ];
          shares = [
            {
              source = "/nix/store";
              mountPoint = "/nix/.ro-store";
              tag = "ro-store";
            }
          ];
          writableStoreOverlay = "/nix/.rw-store";
          volumes = [
            {
              image = "root-overlay.img";
              mountPoint = "/";
              size = 8192;
            }
          ];
        };

        # TODO: Add custom config below, just treated as new machine in NixOS :)
      };
    };
  };
```

重启虚拟机，系统就可以正常放出声音了。[^8]

什么，你说还是没有声音？记得看看你的虚拟机是不是设置为*静音模式*了！

## 用户层网络
如果你和我一样，只是单纯的想要让虚拟机连接到因特网，而不需要特别的配置，使用 `QEMU` 的用户层联网即可。配置如下：
```nix
  microvm.vms = {
    examplevm = {
      config = {
        microvm = {
          hypervisor = "qemu";
          graphics.enable = true;
          qemu.extraArgs = [
            "-audiodev"
            "pipewire,id=auddev0"
            "-device"
            "intel-hda"
            "-device"
            "hda-output,audiodev=auddev0"
          ];
          # NEW! User type network
          interfaces = [
            {
              type = "user";
              id = "vm-netvm";
              mac = "02:00:00:01:01:01";
            }
          ];
          shares = [
            {
              source = "/nix/store";
              mountPoint = "/nix/.ro-store";
              tag = "ro-store";
            }
          ];
          writableStoreOverlay = "/nix/.rw-store";
          volumes = [
            {
              image = "root-overlay.img";
              mountPoint = "/";
              size = 8192;
            }
          ];
        };

        # TODO: Add custom config below, just treated as new machine in NixOS :)
      };
    };
  };
```

## 完整示例？
你可以参考我的 [NixOS](https://github.com/pokon548/dotfiles/blob/main/nixos/modules/workprofile.nix) 配置！这也是我目前在用的配置 :)

[^1]: Android 的工作空间本质就是一个无头用户
[^2]: 事实上，我还曾经考虑过将笔记本的显卡直通进虚拟机内，以获得最佳性能。但由于 NVIDIA 始终无法识别到我手动嵌入固件内的 vBIOS，导致 Muxless 显卡无法正常驱动，故放弃传统虚拟机方案
[^3]: 如果你还是 Xorg 用户，可以考虑使用 xterm 来以其它用户身份执行应用。参见：[https://unix.stackexchange.com/questions/108784/running-gui-application-as-another-non-root-user](https://unix.stackexchange.com/questions/108784/running-gui-application-as-another-non-root-user)
[^4]: https://astro.github.io/microvm.nix/shares.html
[^5]: https://github.com/astro/microvm.nix/issues/83
[^6]: 不特别指定文件系统的情况下，MicroVM 默认会将持久层格式化为 Ext4。如果你不喜欢，可以手动指定一个其它的文件系统：[https://astro.github.io/microvm.nix/microvm-options.html#microvmvolumesfstype](https://astro.github.io/microvm.nix/microvm-options.html#microvmvolumesfstype)
[^7]: 简单来说，开启这个选项：services.pipewire.systemWide = true
[^8]: 如果你遇到了播放设备不对的问题，请尝试重启你的电脑