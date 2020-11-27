# Brunch框架说明文件（简体中文翻译）

## 概述

首先，感谢swtpm的维护者Project Croissant，以及他们的Chromebrew框架。本项目在创建时大量采用了他们的辛劳成果。

Brunch框架的目的是通过ChromeOS官方的恢复镜像，创建一个通用的X86_64镜像。其使用了1GB的ROOTC分区（包含一个定制的内核，一个initramfs，swtpm二进制，用户空间补丁和配置文件）和一个特定的EFI分区用于引导。

**警告：Brunch并非ChromeOS默认支持的工作方式，某些情况下，Brunch下的ChromeOS的脚本运行可能会出现问题，并有可能会意外删除你的数据（甚至非ChromeOS分区的数据）。同时，ChromeOS恢复镜像中包含了可能与其他设备非常相似的固件更新，可能会导致这个相似的设备接受并错误地刷入。若你选择安装Brunch，则你同意承担上述所有风险，并且本人不对你机器工作异常、数据丢失等后果进行负责。因此，我们建议仅在没有任何敏感数据的设备上安装Brunch框架，并且建议将非敏感的数据也备份到云端。**

**提醒：本Repo仅用于对README.md文件进行汉化，本人并不会对此Repo内的其他任何文件进行后续开发和修改。若要了解和下载Project Brunch的最新版本以及提交issue等，请移步原作者sebanc页面！**

## 硬件支持和新增的功能

硬件支持高度依赖通用Linux内核中的硬件兼容性。因此，只有支持Linux的硬件才能够正常工作，并且针对你设备所使用的特定的内核命令应该能够通过GRUB引导程序（参阅“修改GRUB引导程序”一节）。

基础硬件兼容性：
- x86_64电脑并支持UEFI引导；
- 英特尔硬件（CPU和GPU），从第1代“Nehalem”架构开始（即第一代Core i系列处理器，参见https://en.wikipedia.org/wiki/Intel_Core ） ；
- AMD Stoney Ridge（第7代APU，A4/A6/10-9000系列，参见 https://en.wikipedia.org/wiki/List_of_AMD_accelerated_processing_units ），仅能使用“grunt”恢复镜像（更老的处理器和最新的锐龙处理器暂不支持）；
- Nvidia独立显卡也不受支持。

针对BIOS/MBR引导的特别步骤：参照下述步骤进行操作，但在解压Brunch后，请将在本branch（master）中的“mbr_support.tar.gz”也解压进同一目录。

特定硬件支持：
- 传感器：采用了一个实验性的补丁来允许英特尔重力加速度传感器和光线传感器工作；
- Microsoft Surface设备：独立的内核补丁已包含在内。

附加功能：
- nano文本编辑器
- qemu（支持spice）

## ChromeOS恢复镜像

存在2种ChromeOS的恢复镜像，并且它们使用了不同的设备配置机制：
- 已编译镜像：为诸如eve（Google Pixelboot）和nocturne（Google Pixel Slate）这样的单一设备配置。
- 未编译镜像：通过CrosConfig工具，可配置用于多个设备的镜像。

与使用已编译镜像（的配置和Android程序访问）的Croissant框架相反，Brunch在使用了未编译镜像的情况下，理应也能让其两者正常运行，并提供更好的硬件兼容性。

目前来说：
- “rammus”镜像推荐在第4代酷睿及更新的英特尔处理器上使用。
- “samus”镜像推荐在第3代酷睿及更老的英特尔处理器上使用。
- “grunt”镜像仅在你拥有AMD硬件时使用。

如果你不确定使用什么恢复镜像，WesBosch的“brunch-toolkit”项目拥有一个兼容性检查，能检测你应使用的镜像：
https://github.com/WesBosch/brunch-toolkit

ChromeOS恢复镜像可以从https://cros-updates-serving.appspot.com/ 或者 https://cros.tech/ 下载

# 安装方式

你可以将ChromeOS安装在U盘/SD卡（最少16GB）中，或者将镜像安装在你的硬盘里，用于双系统引导（需要至少14GB可用空间）。

## 在Linux下安装ChromeOS（最简单）

### 需求

- root权限。
- `pv`，`tar`和`cgpt`程序包/二进制。

### 在U盘/SD卡/硬盘上全盘安装ChromeOS（单系统引导）

1. 下载ChromeOS恢复镜像并解压。
2. 下载与ChromeOS版本对应的Brunch框架（参见Release页面）。
3. 打开终端，进入Brunch压缩包所在目录。
4. 解压：
```
tar zxvf brunch_< 版本号 >.tar.gz
```
5. 查看你U盘/SD卡/硬盘的名称，例如/dev/sdX（不含“X”后的数字。必须十分小心，因为安装程序会删除该设备的所有文件）
6. 将ChromeOS安装在你的U盘/SD卡/硬盘上（请删除所有“<>”注释内容）：
```
sudo bash chromeos-install.sh -src < ChromOS恢复镜像所在位置 > -dst < 你U盘/SD卡/硬盘的名称，例如/dev/sdX >
```
7. 重启电脑，并引导至U盘/SD卡/硬盘（参见电脑制造商的说明）。
8. （仅针对开启了Secure Boot安全引导的用户）若出现蓝屏并提示 "Verfification failed: (15) Access Denied"，则你需要将安全引导的密钥导入。选择“OK->Enroll key from disk->EFI-SYSTEM->brunch.der->Continue”，然后再次重启并引导。

此时GRUB菜单应该会出现，选择ChromeOS，几分钟后（期间Brunch框架正在为首次启动自行编译），你应该就能看到ChromeOS的欢迎界面，并可以开始使用了。

### 在硬盘上双系统启动

ChromeOS的分区结构非常特别，从而使得它很难实现双系统引导。其中一个解决方案就是绕过这个，并将ChromeOS放置在一个磁盘镜像中，并从镜像引导。

确保你拥有一个ext4（推荐）或NTFS分区，可用空间至少大于14GB，并且没有任何加密。如果没有分区，则手动创建一个（参考网络教程）。

1. 下载ChromeOS恢复镜像并解压。
2. 下载与ChromeOS版本对应的Brunch框架（参见Release页面）。
3. 打开终端，进入Brunch压缩包所在目录。
4. 解压：
```
tar zxvf brunch_< 版本号 >.tar.gz
```
5. 将未加密的ext4或NTFS分区挂载至需要创建磁盘镜像的目录：
```
mkdir -p ~/tmpmount
sudo mount < 需要放置镜像的ext4或NTFS分区的名称（包含sdX后的数字） > ~/tmpmount
```
6. 创建ChromeOS磁盘镜像：
```
sudo bash chromeos-install.sh -src < path to the ChromeOS recovery image > -dst ~/tmpmount/chromeos.img -s < size you want to give to your chromeos install in GB (system partitions will take around 10GB, the rest will be for your data) >
```
7. 在Linux中创建一个GRUB的配置文件，用于Brunch引导：
- 在脚本处理结束后，复制在终端中出现的grub配置信息（在两行星号之间的文字）
- 执行 `sudo cp /etc/grub.d/40_custom /etc/grub.d/99_brunch`
- 接下来执行 `sudo nano /etc/grub.d/99_brunch`在文件末尾粘贴grub配置信息，并保存退出nano（Ctrl+X）
- 最后，执行 `sudo update-grub`
8. 卸载目标分区
```
sudo umount ~/tmpmount
```
9. （仅针对开启了安全引导的用户）下载本branch (master)的安全引导密钥“brunch.der”并通过以下命令合并：
```
sudo mokutil --import brunch.der
```
10. 重启电脑，进入已经修改好的GRUB引导程序。

此时，GRUB菜单就会出现，选择“ChromeOS (boot from disk image)”，几分钟后（期间Brunch框架正在为首次启动自行编译），你应该就能看到ChromeOS的欢迎界面，并可以开始使用了。

## 在Windows下安装ChromeOS

### 需求

- 管理员权限

### 在U盘/SD卡上安装ChromeOS

1. 下载ChromeOS恢复镜像并解压。
2. 下载与ChromeOS版本对应的Brunch框架（参见Release页面）。
3. 在Microsoft商店中安装Ubuntu WSL（参见网络教程）。
4. 运行Ubuntu WSL并安装pv、tar和cgpt依赖：
```
sudo apt update && sudo apt install pv tar cgpt
```
5. 使用 `cd` 命令进入Brunch压缩包所在目录：
```
cd /mnt/c/Users/< username >/Downloads/
```
6. 解压：
```
sudo tar zxvf brunch_< version >.tar.gz
```
7. 确保你的硬盘上有至少14GB的可用空间。
8. 创建ChromeOS镜像：
```
sudo bash chromeos-install.sh -src < ChromOS恢复镜像所在位置 > -dst chromeos.img
```
9. 使用“Rufus”（https://rufus.ie/ ）来将chromeos.img写入你的U盘/SD卡。
10. 重启电脑，并引导至U盘/SD卡/硬盘（参见电脑制造商的说明）。
11. （仅针对开启了Secure Boot安全引导的用户）若出现蓝屏并提示 "Verfification failed: (15) Access Denied"，则你需要将安全引导的密钥导入。选择“OK->Enroll key from disk->EFI-SYSTEM->brunch.der->Continue”，然后再次重启并引导。
12. 此时GRUB菜单应该会出现，选择ChromeOS，几分钟后（期间Brunch框架正在为首次启动自行编译），你应该就能看到ChromeOS的欢迎界面，并可以开始使用了。
至此阶段，你的U盘/SD卡会被错误地识别为仅有14GB空间，无论实际大小是多少。请参照以下步骤修复：
13. 在ChromeOS的桌面，按下Ctrl+Alt+F2打开一个shell进程。
14. 以 `root` 身份登录。
15. 执行以下命令：
```
sudo resize-data
```
16. 当提示需要重启时，重启你的设备，并再次引导进U盘/SD卡，此时你就能正常使用了。

### 在硬盘上双系统启动（新版教程，灵活性较低）

1. 确保你拥有一个NTFS分区，可用空间至少14GB，并且没有BitLocker加密。如果没有分区，可以创建一个（参见网络教程）。
2. 创建一个运行ChromeOS的U盘/SD卡并引导进入。
3. 打开ChromeOS Shell（Ctrl+Alt+T并输入 `shell`）。
4. 将未加密的NTFS分区挂载至需要创建磁盘镜像的目录：
```
mkdir -p ~/tmpmount
sudo mount < the destination partition (ext4 or ntfs) which will contain the disk image > ~/tmpmount
```
5. Create the ChromeOS disk image:
```
sudo bash chromeos-install -dst ~/tmpmount/chromeos.img -s < size you want to give to your chromeos install in GB (system partitions will take around 10GB, the rest will be for your data) >
```
6. Copy the GRUB configuration which is displayed in the terminal (select it and CTRL+SHIFT+C).
7. Run `sudo nano ~/tmpmount/chromeos.grub.txt` and paste the config there (CTRL°SHIFT+V to paste and then CTRL-X to exit)
8. Unmout the destination partition
```
sudo umount ~/tmpmount
```
9. Reboot to Windows, Install grub 2 win (https://sourceforge.net/projects/grub2win/) and launch the application.
10. Click on `Manage Boot Menu` button, then `Add A New Entry`.
11. Select `submenu` in the 'Type' section and input "Chrome OS" as title.
12. Now, click `Edit Custom Code` this will open a text file. Open the chromeos.grub.txt file we saved in step 7 and copy the grub configuration in grub2win.
#### Then remove the "rmmod tpm" line.
13. Click `Ok` and `apply` (It won't save your entry unless you click `ok` and `apply`)
14. Important: Disable "Fast startup" in Windows (refer to online resources).
15. Reboot.
16. The GRUB-2 win menu should appear, select "ChromeOS". Brunch will be rebuilt on first boot so, be patient. You should be greeted by ChromeOS startup screen once the process completes.
You can now start using ChromeOS from your HDD.

## 在ChromeOS下安装ChromeOS

1. Boot your ChromeOS USB flash drive / SD card.
2. Open the ChromeOS shell (CTRL+ALT+T and enter `shell` at the invite)
3. Identify your HDD device name e.g. /dev/sdX (Be careful here as the installer will erase all data on the target drive)
4. Install ChromeOS to HDD:
```
sudo chromeos-install -dst < your HDD device. e.g. /dev/sdX >
```
5. Shutdown your computer and remove your ChromeOS USB flash drive / SD card.

Note: Even if you boot from GRUB on your HDD, if you have a ChromeOS USB flash drive / SD card inserted, the initramfs will boot from it in priority.

The GRUB menu should appear, select ChromeOS and after a few minutes (the Brunch framework is building itself on the first boot), you should be greeted by ChromeOS startup screen. You can now start using ChromeOS.

# 可选步骤

## 框架选项

某些选项的特定功能可以通过内核命令行来激活，但这些功能也有可能存在风险，或不适用于所有用户：
- "enable_updates"：允许ChromeOS系统更新（请自行承担风险：ChromeOS会被升级，但Brunch框架/内核并不会。这可能会让你的ChromeOS工作不稳定甚至无法引导）；
- "android_init_fix"：代替用的初始化，用于支持某些设备无法正确启动其中一种Android容器时进行切换；
- "mount_internal_drives"：允许在ChromeOS下自动挂载硬盘分区（Android媒体服务将会检测这些设备，在其结束前将会造成高CPU占用。此过程依照数据的实际情况，有可能会花费数小时不等），若分区存在卷标，则会被使用；
- "broadcom_wl"：若你的设备存在博通无线网络设备，则启用；
- "iwlwifi_backport"：若内核无法原生支持你的英特尔无线网络设备，则启用；
- "rtl8188eu"：若你的设备使用了rtl8188eu无线网卡，则启用；
- "rtl8723bu": 若你的设备使用了rtl8723bu无线网卡，则启用；
- "rtl8723de": 若你的设备使用了rtl8723de无线网卡，则启用；
- "rtl8821ce": 若你的设备使用了rtl8821ce无线网卡，则启用；
- "rtbth"：若你的设备使用了RT3290/RT3298LE蓝牙设备，则启用；
- "acpi_power_button"：若长按电源按钮不显示电源菜单，则尝试使用这个选项；
- "alt_touchpad_config"：若触摸板存在问题，则尝试使用；
- "alt_touchpad_config2"：另一个触摸板问题的解决方案；
- "disable_intel_hda"：某些Chromebook需要屏蔽snd_hda_intel模块，本选项将会再次进行此操作；
- "internal_mic_fix"：用于强制启用某些设备的内置麦克风；
- "asus_c302"：华硕C302设备专用的固件和补丁；
- "baytrail_chromebook"：英特尔Baytrail架构处理器Chromebook专用的音频修复；
- "sysfs_tablet_mode"：允许从sysfs控制平板模式(使用`echo 1 | sudo tee /sys/bus/platform/devices/tablet_mode_switch.0/tablet_mode`命令来激活，或者使用0来关闭)；
- "force_tablet_mode"：与上一条相同，但在开机时默认启用平板模式；
- "suspend_s3"：关闭“挂起到空闲（S0ix）”状态，并使用S3代替；
- "advanced_als"：ChromeOS默认的自动亮度控制非常的基础 (https://chromium.googlesource.com/chromiumos/platform2/+/master/power_manager/docs/screen_brightness.md )，此选项将开启更多自动亮度控制等级（参考自Google Pixel Slate）。

在内核命令行上添加"options=选项1,选项2,..."（没有空格）来激活它们。
请将上述指令添加在cros_debug之后（含空格），loop.max.....之前。最后一个选项没有逗号，并与loop.max存在一个空格。

例如："cros_debug options=enable_updates,advanced_als loop.max....."将会开启这两个选项。

## Kernel command line parameters

Those are not options, just add them on the kernel command line after "cros_debug" and before "options=...." if any:
- "enforce_hyperthreading=1": improve performance by disabling a ChromeOS security feature and forcing hyperthreading everywhere (even in crositini).
- "i915.enable_fbc=0 i915.enable_psr=0": if you want to use crouton (needed with kernel 5.4).
- "psmouse.synaptics_intertouch=1": enables gestures with more than 2 fingers on some touchpad models.
- "console=": No text will be displayed on boot (it will not make boot faster).

like this: 
![](https://user-images.githubusercontent.com/69226625/97113026-9fec2880-170d-11eb-930f-972f0b38af4f.png)
## Identify the installed Brunch framework version

1. Open the ChromeOS shell (CTRL+ALT+T and enter `shell` at the invite)
2. Display the Brunch version:
```
cat /etc/brunch_version
```

## Update both ChromeOS and the Brunch framework

It is currently recommended to only update ChromeOS when the matching version of the Brunch framework has been released.

1. Download the new ChromeOS recovery image version and extract it.
2. Download the Brunch release corresponding to the ChromeOS recovery version (from the GitHub release section).
3. Open the ChromeOS shell (CTRL+ALT+T and enter `shell` at the invite)
4. Update both ChromeOS and Brunch:
```
sudo chromeos-update -r < path to the ChromeOS recovery image > -f < path to the Brunch release archive >
```
5. Restart ChromeOS

## Update only the Brunch framework

If you chose to use the "enable_updates" option and have updated to a new ChromeOS release, you might want to update the brunch framework to match your current ChromeOS version.

1. Download the Brunch release corresponding to your ChromeOS version (from the GitHub release section).
2. Open the ChromeOS shell (CTRL+ALT+T and enter `shell` at the invite)
3. Update Brunch:
```
sudo chromeos-update -f < path to the Brunch release archive >
```
4. Restart ChromeOS

*WARNING*: Do not update brunch with unfinished OTA update as it may make chromeOS unbootable.

## Modify the GRUB bootloader

### From Windows

1. Install notepad++ (https://notepad-plus-plus.org/)
2. Look for the EFI partition in the Explorer and browse to the efi/boot folder.
3. Edit the grub.cfg file with notepad++ (warning: editing this file with standard Notepad or Wordpad will render the file unusable and prevent GRUB from booting due to formatting issues)
4. Add your specific kernel parameters at the end of the Linux line arguments.

### From Linux

1. Create a directory to mount the EFI partition:
```
mkdir /tmp/efi_part
```
2. Mount the partition 12 of your device to your EFI partition:
```
sudo mount /dev/< partition 12 of ChromeOS device > /tmp/efi_part
```
3. Edit the file /tmp/efi_part/efi/boot/grub.cfg with your favorite editor (launched as root).
4. Unmount the partition:
```
sudo umount /tmp/efi_part
```

### From ChromeOS

1. Run `sudo edit-grub-config`.
2. Now you can modify your grub entry.
3. Save by Ctrl+o (Press enter to confirm).
4. Exit by Ctrl+x.

You can visit wiki https://github.com/sebanc/brunch/wiki


# FAQ

1) The instructions are difficult to follow as I am not familiar with Linux commands.

I cannot not go much deeper into details here for now but I will try to clarify the install process once I see the main pain points. Nevertheless, ChromeOS is based on Linux and it would probably be interesting for you to read online resources on Linux basics before attempting this.

2) My computer will not boot the created USB flash drive / SD card whereas it normally can (and I have correctly followed the instructions).

Some devices (notably Surface Go) will not boot a valid USB flash drive / SD card with secure boot on even if the shim binary is signed. For those devices, you will need to disable secure boot in your bios settings and use the legacy EFI bootloader by adding the "-l" parameter when running the chromeos-install.sh script.

3) The first boot and the ones after a framework change or an update are incredibly long.

Unfortunately, the Brunch framework has to rebuild itself by copying the original rootfs, modules and firmware files after each significant change. The time this process takes depends mostly on your USB flash drive / SD card write speed. You may try with one that has better write speed or use the dual boot method to install it on your HDD.

4) ChromeOS reboots randomly.

This can in theory be due to a lot of things. However, the most likely reason is that your USB flash drive / SD card is too slow. You may try with one that has better write speed or use the dual boot method to install it on your HDD.

5) Some apps do not appear on the playstore (Netflix...)

In order to have access to the ChromeOS shell, ChromeOS is started in developer mode by default. If you have a stable enough system, you can remove "cros_debug" from the GRUB kernel command line (see "Modify the GRUB bootloader" section) and then do a Powerwash (ChromeOS mechanism which will wipe all your data partition) to disable developer mode.

6) Some apps on the Playstore show as incompatible with my device.

Some Playstore apps are not compatible with genuine Chromebooks so it is probably normal.

