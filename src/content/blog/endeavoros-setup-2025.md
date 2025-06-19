---
author: Liya Du
pubDatetime: 2025-06-17T23:12:00Z
title: My EndeavorOS setup in 2025
postSlug: my-endeavor-setup-2025
featured: true
draft: false
tags:
  - linux
ogImage: "/assets/20240310/manjaro-setup-2024.png"
description: The Linux Desktop is more usable than ever in 2025.
---

After a fail attempt to upgrade my Hackintosh to latest Sonoma, I decided to give Manjaro a try. And see my self how long I can stick with that as a daily system. The one single fact I choose Manjaro over other distros is the [AUR](https://aur.archlinux.org/packages/yay) which makes finding and installing software like a breeze.

> EndeavorOS is now my new fav of Arch Linux distro.

## Prepare installation USB stick

Grab the latest EndeavorOS ISO from [here](https://endeavouros.com/). I am using the Mercury Neo with Linux 6.13.7 which was released on early March 2025.

In Windows, use [Rufus](https://rufus.ie/) to create a bootable USB stick.

In macOS, just use `dd` to create a bootable USB stick.

```bash
diskutil list # find the disk number, let's say it's /dev/disk4

diskutil unmountDisk /dev/disk4

sudo dd if=/path/to/endeavor.iso of=/dev/disk4 bs=4M status=progress && sync
```

And that's it, you can now boot from the USB stick.

During installation, we want to use `btrfs` partition which is more friendly for snapshot.

## Install essential packages

First thing first, every Arch Linux user must have `yay`.

```bash
sudo pacman -Sy yay base-devel
```

By default, when makepkg builds an AUR package, compression is enabled. After building, the package needs to be decompressed again for installation. This is unnecessary (and slow for large packages) if it's only for personal use on one's own machine. To set up package building without compression:

```bash
sudo sed -i "s/PKGEXT='.pkg.tar.xz'/PKGEXT='.pkg.tar'/g" /etc/makepkg.conf
```

Install the essential packages

```bash
yay -S zsh vim

```

Switch to `zsh` and install `oh-my-zsh`:

```bash
chsh -s /bin/zsh

sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

```

## My daily applications

Install my daily applications, your mileage may vary.

```bash
yay -S google-chrome cursor-bin warp-terminal-bin 1password notion-app-electron figma-linux feishu-bin zoom timeshift spotify git-extras

```


## Fcitx5 with RIME

```bash
sudo pacman -S fcitx5 fcitx5-qt fcitx5-gtk fcitx5-chinese-addons fcitx5-configtool fcitx5-rime

```

Add following into `/etc/environment`:

```txt
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDK_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

You can double check if the environment variables are set correctly by running `export|grep -E "IM|XMODIFIERS" `.

Add Rime as input method in `fcitx5-configtool`.

<Image src="/assets/20240310/add-rime.png" format="png" alt="Add Rime in Fcitx5 configuration tool" />

Clone the repo to configure Rime to use WuSong Pinyin (雾凇拼音):

```bash
git clone https://github.com/iDvel/rime-ice.git --depth=1

cp -r rime-ice/*  ~/.local/share/fcitx5/rime
```

And use some nicer theme:

```bash
yay -S fcitx5-skin-fluentdark-git
```

And set the theme in `Input Method > Configure addons > Classic User Interface > Theme`.

There is an [extensive reading](https://manateelazycat.github.io/2023/09/11/fcitx-best-config/) about further customizations of Fcitx5 with RIME

## Development environment

Install `nvm` and Node.js LTS:

```bash
yay -S nvm

nvm install --lts
```

Install `pnpm` and `yarn`:

```bash
yay -S yarn-pnpm-corepack

corepack prepare --activate yarn@1 pnpm@latest
```

## Small tweaks

- If you are using HiDPI, **Zoom** might not scale properly. You can fix it by the section below.
- Uncheck `Settings > Power Management > Energy Savings > Dim screen` to prevent screen dimming when you are back from idle.
- I am using a couple of Logitech devices, [Solaar](https://github.com/pwr-Solaar/Solaar) is a must-have device manager.

### Zoom HiDPI fix

Create a Zoom desktop shortcut with below content

```txt
[Desktop Entry]
Name=Zoom
Comment=Zoom Video Communications
Exec=env QT_AUTO_SCREEN_SCALE_FACTOR=1 /usr/bin/zoom %u
Icon=/usr/share/pixmaps/application-x-zoom.png
Terminal=false
Type=Application
StartupNotify=true
MimeType=x-scheme-handler/zoommtg;x-scheme-handler/zoomus;application/x-zoom;
Categories=Network;AudioVideo;
```

The key is `env QT_AUTO_SCREEN_SCALE_FACTOR=1` which lets Qt auto-detect screen DPI scaling. Or the fallback `QT_SCALE_FACTOR=2` might work as well.

## Know issues

- My set up is two monitors all with DP, there is a known issue that the [primary monitor will not wake up after sleep](https://forum.manjaro.org/t/primary-screen-not-working-after-wake-from-sleep-only-kde-5-26/125703/50?page=2). I have to unplug and plug it back to make it work.
