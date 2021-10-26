# Brunch框架说明文件（简体中文翻译）

## 概述

首先，感谢swtpm的维护者Project Croissant，以及他们的Chromebrew框架。本项目在创建时大量采用了他们的辛劳成果。

Brunch框架的目的是通过ChromeOS官方的恢复镜像，创建一个通用的X86_64镜像。其使用了1GB的ROOTC分区（包含一个定制的内核，一个initramfs，swtpm二进制，用户空间补丁和配置文件）和一个特定的EFI分区用于引导。

**警告：Brunch并非ChromeOS默认支持的工作方式，某些情况下，Brunch下的ChromeOS的脚本运行可能会出现问题，并有可能会意外删除你的数据（甚至非ChromeOS分区的数据）。同时，ChromeOS恢复镜像中包含了可能与其他设备非常相似的固件更新，可能会导致这个相似的设备接受并错误地刷入。若你选择安装Brunch，则你同意承担上述所有风险，并且本人不对你机器工作异常、数据丢失等后果进行负责。因此，我们建议仅在没有任何敏感数据的设备上安装Brunch框架，并且建议将非敏感的数据也备份到云端。**

**译者声明：本Repo仅用于对README.md文件进行汉化，并基于个人实践操作经验，对其中部分内容进行适当添加、修改。本人并不会对此Repo内的其他任何文件进行后续开发和修改。此外，本说明文件由于诸多原因，无法时刻与原Repo同步更新汉化，若要了解和下载Project Brunch的最新版本以及提交issue等，请移步原作者sebanc/brunch页面！**

**再次强调，请勿在本Repo内下载Brunch框架！本Repo仅对README.md进行汉化，不会对其他文件进行同步和后续开发！**

## 硬件支持和新增的功能

硬件支持高度依赖通用Linux内核中的硬件兼容性。因此，只有支持Linux的硬件才能够正常工作，并且针对你设备所使用的特定的内核命令应该能够通过Grub引导程序（参阅“修改Grub引导程序”一节）。

基础硬件兼容性：
- x86_64电脑并支持UEFI或MBR引导（使用MBR引导安装将会附带一些限制，详见“MBR/BIOS设备相关限制”一节）；
- 英特尔酷睿i系列处理器（CPU和GPU），从第1代“Nehalem”架构开始（即第一代Core i系列处理器，参见https://en.wikipedia.org/wiki/Intel_Core ）；
- AMD 锐龙处理器（CPU和GPU），使用“zork”镜像；
- AMD Stoney Ridge APU（即第7代A12/A10/A9/A6/A4/FX/E2-9000系列APU，参见https://en.wikipedia.org/wiki/List_of_AMD_accelerated_processing_units ）；
- 仅拥有Nvidia独立显卡的设备不受支持，即便是同时拥有核显和Nvidia独显的设备，独立显卡也无法使用（含独显的Surface Book用户请在UEFI Settings-Devices中关闭“DGPU”选项，否则无法引导）。

提醒：
- 英特尔第1代核显不兼容高于r81的ChromeOS版本（后续更新中也许仍有可能改变）。

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
### 英特尔
* [“rammus”镜像适合在第1-第9代酷睿处理器上使用。](https://cros.tech/device/rammus)
* [“volteer”镜像适合在第10和第11代酷睿处理器上使用。](https://cros.tech/device/volteer)
  * [第11代（和某些第10代）酷睿处理器需要使用5.10内核](https://github.com/bbs3223474/Brunch-CN-Readme#%E4%BF%AE%E6%94%B9%E5%86%85%E6%A0%B8%E7%89%88%E6%9C%AC) 
### AMD
* [“grunt”镜像推荐在AMD Stoney Ridge和Bristol Ridge（即第7代APU）上使用。](https://cros.tech/device/grunt)
* [“zork”镜像推荐在AMD锐龙处理器上使用。](https://cros.tech/device/zork)
  * [锐龙4000系列处理器需要使用5.10内核](https://github.com/bbs3223474/Brunch-CN-Readme#%E4%BF%AE%E6%94%B9%E5%86%85%E6%A0%B8%E7%89%88%E6%9C%AC) 

ChromeOS恢复镜像可以从上述连接，或是https://cros-updates-serving.appspot.com/ 或者 https://cros.tech/ 下载

# Brunch工具包

如果你不想使用下述的一系列Linux命令，由WesBosch制作的“Brunch工具包”可以让Brunch的安装和升级更为简单：
https://github.com/WesBosch/brunch-toolkit

其主要功能有（Brunch、Linux和WSL环境下可用）：
- 检查CPU与Brunch框架的兼容性。
- 根据硬件推荐可用镜像名称。
- 将Brunch安装到硬盘或分区中。

在Brunch框架下可用的功能：
- 更新ChromeOS和/或Brunch。
- 修改ChromeOS启动动画。

# 安装方式

你可以将ChromeOS安装在U盘/SD卡（最少16GB）中，或者将镜像安装在你的硬盘里，用于双系统引导（需要至少14GB可用空间）。

## 在Linux下安装ChromeOS（最简单）

### 需求

- root权限。
- `pv`，`tar`和`cgpt`程序包/二进制。

### 在U盘/SD卡/硬盘上全盘安装ChromeOS（全盘安装/单系统引导）

1. 下载ChromeOS恢复镜像并解压。
2. 从[Release页面](https://github.com/sebanc/brunch/releases)下载Brunch。通常，我们推荐使用[最新版本](https://github.com/sebanc/brunch/releases/latest)。
3. 打开终端，进入Brunch压缩包所在目录。
4. 解压：
```
tar zxvf brunch_< 版本号 >.tar.gz
```
5. 查看你U盘/SD卡/硬盘的名称，例如/dev/sdX（不含“X”后的数字。必须十分小心，因为安装程序会删除该设备的所有文件）。你也可以使用Gparted、Gnome disks或直接执行lsblk -e7命令来查看你想要安装到的硬盘名称。
6. 将ChromeOS安装在你的U盘/SD卡/硬盘上（请删除所有“<>”注释内容）：
```
sudo bash chromeos-install.sh -src < ChromOS恢复镜像所在位置 > -dst < 你U盘/SD卡/硬盘的名称，例如/dev/sdX >
```
7. 重启电脑，并引导至U盘/SD卡/硬盘（参见电脑制造商的说明）。
8. （仅针对开启了Secure Boot安全引导的用户）若出现蓝屏并提示 "Verfification failed: (15) Access Denied"，则你需要将安全引导的密钥导入。选择“OK->Enroll key from disk->EFI-SYSTEM->brunch.der->Continue”，然后再次重启并引导。
9. (推荐) 当Grub引导菜单出现时，选择“ChromeOS (settings)”进入，并配置brunch（参见本说明文档的“Brunch设置”一节）。配置完成后重启设备，重新引导进入U盘/SD卡。
10. 当Grub引导菜单出现时，选择“ChromeOS”，几分钟之后（Brunch框架在首次引导时需要自我编译）你就应该能看到ChromeOS的启动界面了。

### 在硬盘上与Linux双系统启动

ChromeOS的分区结构非常特别，从而使得它很难实现双系统引导。其中一个解决方案就是绕过这个，并将ChromeOS放置在一个磁盘镜像中，并从镜像引导。

确保你拥有一个ext4（推荐）或NTFS分区，可用空间至少大于14GB，并且没有任何加密。如果没有分区，则手动创建一个（参考网络教程）。

1. 下载ChromeOS恢复镜像并解压。
2. 下载与ChromeOS版本对应的Brunch框架（参见Release页面）。
3. 打开终端，进入Brunch压缩包所在目录。
4. 解压：
```
tar zxvf brunch_< 版本号 >.tar.gz
```
5. 使用 `lsblk -e7` 来检测你想用来安装Brunch的分区名称。（如果你还没有创建分区，则需要先创建后进行此步骤）
6. 将未加密的ext4或NTFS分区挂载至需要创建磁盘镜像的目录：
```
mkdir -p ~/tmpmount
sudo mount < 需要放置镜像的ext4或NTFS分区的名称（包含sdX后的数字） > ~/tmpmount
```
7. 创建ChromeOS磁盘镜像：
```
sudo bash chromeos-install.sh -src < ChromOS恢复镜像所在位置 > -dst ~/tmpmount/chromeos.img -s < 以GB为单位，输入你希望分配给ChromeOS的容量（系统分区大约会占用10GB，其余的将作为数据分区）>
```
8. 在Linux中创建一个Grub的配置文件，用于Brunch引导：
- 在脚本处理结束后，复制在终端中出现的grub配置信息（在两行星号之间的文字）
- 执行 `sudo cp /etc/grub.d/40_custom /etc/grub.d/99_brunch`
- 接下来执行 `sudo nano /etc/grub.d/99_brunch`在文件末尾粘贴Grub配置信息，并保存退出nano（Ctrl+X）
- 最后，执行 `sudo update-grub`
9. 卸载目标分区
```
sudo umount ~/tmpmount
```
10. （仅针对开启了安全引导的用户）下载本branch (master)的安全引导密钥“brunch.der”并通过以下命令合并：
```
sudo mokutil --import brunch.der
```
11. 重启电脑。
12. (推荐) 当Grub引导菜单出现时，选择“ChromeOS (settings)”进入，并配置brunch（参见本说明文档的“Brunch设置”一节）。配置完成后重启设备，重新引导。
13. 当Grub引导菜单出现时，选择“ChromeOS”，几分钟之后（Brunch框架在首次引导时需要自我编译）你就应该能看到ChromeOS的启动界面了。

## 在Windows下安装ChromeOS

### 需求

- 管理员权限

### 在U盘/SD卡上安装ChromeOS

1. 下载ChromeOS恢复镜像并解压。
2. 下载与ChromeOS版本对应的Brunch框架（参见Release页面）。
3. 打开Windows的WSL功能，并在Microsoft商店中安装Ubuntu WSL（参见网络教程）。
4. 运行Ubuntu WSL并安装pv、tar和cgpt依赖：
```
sudo apt update && sudo apt install pv tar cgpt
```
5. 使用 `cd` 命令进入Brunch压缩包所在目录：
```
cd /mnt/c/Users/< 用户名 >/Downloads/
```
6. 解压：
```
sudo tar zxvf brunch_< 版本号 >.tar.gz
```
7. 确保你的硬盘上有至少14GB的可用空间。
8. 创建ChromeOS磁盘镜像：
```
sudo bash chromeos-install.sh -src < ChromOS恢复镜像所在位置 > -dst chromeos.img
```
9. 使用“Rufus”（https://rufus.ie/ ）来将chromeos.img写入你的U盘/SD卡。
10. 重启电脑，并引导至U盘/SD卡/硬盘（参见电脑制造商的说明）。
11. （仅针对开启了Secure Boot安全引导的用户）若出现蓝屏并提示 "Verfification failed: (15) Access Denied"，则你需要将安全引导的密钥导入。选择“OK->Enroll key from disk->EFI-SYSTEM->brunch.der->Continue”，然后再次重启并引导。
12. (推荐) 当Grub引导菜单出现时，选择“ChromeOS (settings)”进入，并配置brunch（参见本说明文档的“Brunch设置”一节）。配置完成后重启设备，重新引导进入U盘/SD卡。
13. 当Grub引导菜单出现时，选择“ChromeOS”，几分钟之后（Brunch框架在首次引导时需要自我编译）你就应该能看到ChromeOS的启动界面了。
至此阶段，你的U盘/SD卡会被错误地识别为仅有14GB空间，无论实际大小是多少。请参照以下步骤修复：
14. 在ChromeOS的桌面，按下Ctrl+Alt+F2打开一个shell进程。
15. 以 `root` 身份登录。
16. 执行以下命令：
```
resize-data
```
17. 当提示需要重启时，重启你的设备，并再次引导进U盘/SD卡，此时你就能正常使用了。

### 在硬盘上与Windows双系统启动（新版教程，灵活性较低）

**提示**：如果你觉得以下内容太难理解，[这里](https://github.com/sebanc/brunch/wiki/Detailed-installation-instructions-from-Windows)有一份更为详细且易懂的教程可供参考。

1. 确保你拥有一个NTFS分区，可用空间至少14GB，并且没有BitLocker加密。如果没有分区，可以手动创建一个（参见网络教程）。
2. 创建一个运行Brunch的U盘/SD卡（见前文）并引导进入。
3. 打开ChromeOS Shell（Ctrl+Alt+T并输入 `shell`）。
4. 使用 `lsblk -e7` 来检测你想用来安装Brunch的分区名称。（如果你还没有创建分区，则需要先创建后进行此步骤）
5. 将未加密的NTFS分区挂载至需要创建磁盘镜像的目录：
```
mkdir -p ~/tmpmount
sudo mount < 需要生成镜像的目标分区名称 > ~/tmpmount
```
6. 创建ChromeOS磁盘镜像：
```
sudo bash chromeos-install -dst ~/tmpmount/chromeos.img -s < 以GB为单位，输入你希望分配给ChromeOS的容量（系统分区大约会占用10GB，其余的将作为数据分区）>
```
7. 复制在终端中显示的Grub配置信息（选中并按下Ctrl+Shift+C）。
8. 执行 `sudo nano ~/tmpmount/chromeos.grub.txt` 命令并粘贴 (Ctrl+Shift+V粘贴，Ctrl+X退出)。
9. 卸载目标分区：
```
sudo umount ~/tmpmount
```
10. 重启至Windows，安装Grub2win (https://sourceforge.net/projects/grub2win/) 并打开程序。
11. 点击 `Manage Boot Menu` 按钮，选择 `Add A New Entry` 。
12. 在“Type”菜单下选择 `submenu` ，并输入“Chrome OS”作为标题。
13. 点击 `Edit Custom Code` 来打开一个文本编辑框。打开第7步创建的chromeos.grub.txt，并将其中的内容复制到grub2win的配置文本中。
#### 删除“rmmod tpm”一行
14. 保存文件，点击 `Ok` 和 `apply` （点击这两个按钮之前，所有配置都不会被保存）。
15. 重要：关闭Windows的快速启动（参见网络教程）。
16. 重启。
17. (推荐) 当Grub引导菜单出现时，选择“ChromeOS (settings)”进入，并配置brunch（参见本说明文档的“Brunch设置”一节）。配置完成后重启设备，重新引导。
18. 当Grub引导菜单出现时，选择“ChromeOS”，几分钟之后（Brunch框架在首次引导时需要自我编译）你就应该能看到ChromeOS的启动界面了。

### 在硬盘上与Windows双系统启动（旧版教程，灵活性较高）

1. 下载ChromeOS恢复镜像并解压。
2. 下载与ChromeOS版本对应的Brunch框架（参见Release页面）。
3. 开启Windows的WSL功能，并在Microsoft商店中安装Ubuntu WSL（参见网络教程）。
4. 运行Ubuntu WSL并安装pv、tar和cgpt依赖：
```
sudo apt update && sudo apt install pv tar cgpt
```
5. 使用 `cd` 命令进入Brunch压缩包所在目录（请根据实际目录修改以下内容）：
```
cd /mnt/c/Users/< 用户名 >/Downloads/
```
6. 解压：
```
sudo tar zxvf brunch_< 版本号 >.tar.gz
```
7. 确保你的硬盘存在一个NTFS分区，可用空间至少14GB，并且没有BitLocker加密。如果没有分区，可以手动创建一个（参见网络教程）。亦可以使用Windows所安装在的分区。
8. 创建ChromeOS磁盘镜像：
```
sudo bash chromeos-install.sh -src < ChromOS恢复镜像所在位置 > -dst chromeos.img < 这将会在Brunch框架所在的目录中直接创建名为chromeos.img的镜像，且不影响分区的其他数据 > -s < 以GB为单位，输入你希望分配给ChromeOS的容量（系统分区大约会占用10GB，其余的将作为数据分区）>
```
9. 将Brunch目录下生成的chromeos.img拷贝至你所需要的位置（可以是任意一个NTFS分区的根目录，也可以放在子目录下）。
10. 下载任意一个Linux发行版（如Linux Mint、Ubuntu等，建议使用带图形界面的版本），将其iso写入U盘/SD卡。
11. 重启电脑，并引导至U盘/SD卡内，进入Linux发行版的Live预览（无需安装）。
12. 使用磁盘分区工具（如cgpt等）查看ChromeOS镜像所在目标分区的名称（例如/dev/sda1、/dev/nvme0n1p3等），并记下。
13. 拔下U盘/SD卡并重启电脑，回到Windows下。
14. 安装[Grub2win](https://sourceforge.net/projects/grub2win/)并打开程序。
15. 点击 `Manage Boot Menu` 按钮，选择 `Add A New Entry` 。
16. 在“Type”菜单下选择 `submenu` （亦可以使用isoboot或custom code，没有明显区别），并输入“Chrome OS”作为标题。
17. 点击 `Edit Custom Code` 来打开一个文本编辑框。参照下列内容，对必要信息进行修改，并输入到文本框内。
```
img_part=/dev/sda1 < 或/dev/sdb1、/dev/nvme0n1p3等，参照第12步获得的信息修改 >
img_path=/ChromeOS/chromeos.img < 如果镜像所在位置和文件名不同，请按照你的实际情况修改。若镜像在子目录内，子目录的大小写必须与实际相符！若在根目录内，则保留开头的“/”，然后输入镜像名称 >
search --no-floppy --set=root --file $img_path
loopback loop $img_path
linux (loop,7)/kernel boot=local noresume noswap loglevel=7 disablevmx=off \
	cros_secure cros_debug console=vt.global_cursor_default=0 brunch_bootsplash=default options=enable_updates,advanced_als loop.max_part=16 img_part=$img_part img_path=$img_path
initrd (loop,7)/lib/firmware/amd-ucode.img (loop,7)/lib/firmware/intel-ucode.img (loop,7)/initramfs.img

###警告：请不要复制本行及以下所有行。在粘贴时请删除上面所有的“<>”注释！另外，除了“img_part”、“img_path”两项和“cros_debug”之后、“loop.max”之前的内容，其余内容不需要改动，不建议尝试手动修改。一旦有空格、逗号等不符合格式要求，则GRUB2Win引导界面会出现错误信息。###
###Grub2Win不需要“rmmod tpm”一行，因此本例中没有提供。###
###若你已经熟悉Linux环境下你的硬盘分区的名称，则可以忽略第10到第12步。分区名称必须严格采用Linux环境下的格式，在Windows的“磁盘管理”和“diskpart”工具中的内容基本没有参考价值。###

```
18. 保存文件，点击 `Ok` 和 `apply` （点击这两个按钮之前，所有配置都不会被保存）。
19. 重要：关闭Windows的快速启动（参见网络教程）。
20. 重启。
21. 此时GRUB2Win引导菜单将会出现，选择Chrome OS进入，并耐心等待。
现在你可以正常使用ChromeOS了。

## 在U盘/SD卡中的Brunch下安装ChromeOS

1. 引导进你U盘/SD卡的ChromeOS。
2. 打开ChromeOS Shell (Ctrl+Alt+T并输入 `shell` )。
3. 查看你硬盘的名称，例如/dev/sdX（不含“X”后的数字。必须十分小心，因为安装程序会删除该设备的所有文件）。也可以使用 `lsblk -e7` 来检测你想用来安装的分区名称。
4. 将ChromeOS安装到硬盘上：
```
sudo chromeos-install -dst < 目标硬盘的名称，例如/dev/sdX >
```
5. 关闭电脑，并取出U盘/SD卡。（注意：即便你从硬盘上的GRUB引导，如果你安装了Brunch的U盘/SD卡处于插入状态，initramfs也会优先从其上面引导。）

此时Grub菜单应该会出现，选择ChromeOS，几分钟后（期间Brunch框架正在为首次启动自行编译），你应该就能看到ChromeOS的欢迎界面，并可以开始使用了。

## MBR/BIOS设备相关限制

Brunch也可以在BIOS/MBR设备上安装，但需要多进行几步操作：
- 如果你已经安装了Windows，并且希望实现ChromeOS双引导，你可以参照[此处](https://github.com/sebanc/brunch/wiki/Detailed-installation-instructions-from-Windows)。
- 否则，master分支中的mbr_support.tar.gz文件将只能用于创建运行Brunch的U盘，或是通过以下方式进行单系统安装：

安装步骤：
1) 在“基础硬件兼容性”一节中确认你的CPU/GPU是否受到支持。
2) 确认`pv`，`tar`，`cgpt`和`sgdisk`软件包/二进制已安装。
3) 解压Brunch安装包到你需要的目录下。
4) 解压mbr_support.tar.gz到同一目录（若需要覆盖文件，则覆盖）。
5) 下载ChromeOS恢复镜像并解压。
6) 从“在U盘/SD卡/硬盘上全盘安装ChromeOS（全盘安装/单系统引导）”一节的第5步开始安装。

# Brunch设置

Brunch配置菜单可以通过Grub的“ChromeOS (settings)”引导项进入，或在ChromeOS下的crosh shell里通过 `sudo edit-brunch-config` 命令进入。
提示：要访问crosh shell，可按下Ctrl+Alt+T，并输入 `shell` 。（译注：与Windows双引导的安装方式不支持上述方式配置，需要在Windows下打开Grub2Win才可修改）

## 框架选项

某些选项的特定功能可以通过内核命令行来激活，也可通过访问Brunch配置菜单直接启用，但这些功能也有可能存在风险，或不适用于所有用户：
- "enable_updates"：允许ChromeOS系统更新（请自行承担风险：ChromeOS会被升级，但Brunch框架/内核并不会。这可能会让你的ChromeOS工作不稳定甚至无法引导）；
- "pwa"：使用它来开启Brunch PWA（然后在这里下载安装：https://sebanc.github.io/brunch-pwa/ ）；
- "android_init_fix"：代替用的初始化，用于支持某些设备无法正确启动其中一种Android容器时进行切换；
- "mount_internal_drives"：允许在ChromeOS下自动挂载内置硬盘的所有分区（Android媒体服务将会检测这些设备，在其结束前将会造成高CPU占用。此过程依照数据的实际情况，有可能会花费数小时不等），若分区存在卷标，则会被使用；
- "broadcom_wl"：若你的设备存在博通无线网卡，则启用；
- "iwlwifi_backport"：若内核无法原生支持你的英特尔无线网卡，则启用；
- "rtl8188eu"：若你的设备使用了rtl8188eu无线网卡，则启用；
- "rtl8188fu": 若你的设备使用了rtl8188fu无线网卡，则启用；
- "rtl8192eu": 若你的设备使用了rtl8192eu无线网卡，则启用；
- "rtl8723bu"：若你的设备使用了rtl8723bu无线网卡，则启用；
- "rtl8723de"：若你的设备使用了rtl8723de无线网卡，则启用；
- "rtl8723du"：若你的设备使用了rtl8723du无线网卡，则启用；
- "rtl8812au"：若你的设备使用了rtl8812au无线网卡，则启用；
- "rtl8814au"：若你的设备使用了rtl8814au无线网卡，则启用；
- "rtl8821ce"：若你的设备使用了rtl8821ce无线网卡，则启用；
- "rtl88x2bu"：若你的设备使用了rtl88x2bu无线网卡，则启用；
- "rtl8821cu"：若你的设备使用了rtl8821cu无线网卡，则启用；
- "rtbth"：若你的设备使用了RT3290/RT3298LE蓝牙设备，则启用；
- "ipts"：启用K5.4/5.10内核下的Surface设备的触摸屏驱动（感谢Linux-surface团队，尤其是StollD）；
- "goodix"：改善Goodix（汇顶科技）触摸屏支持；
- "invert_camera_order"：如果你的前后置相机切换顺序是相反的，则启用；
- "no_camera_config"：如果你的相机完全不工作，你可以尝试使用该选项以关闭相机配置文件；
- "oled_display"：若你的设备使用的是OLED显示屏，则启用（必须使用K5.10内核）；
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
- "advanced_als"：ChromeOS默认的自动亮度控制非常的基础 (https://chromium.googlesource.com/chromiumos/platform2/+/master/power_manager/docs/screen_brightness.md )，此选项将开启更多自动亮度控制等级（参考自Pixel Slate）。

在内核命令行上添加"options=选项1,选项2,..."（没有空格）来激活它们。
请将上述指令添加在cros_debug之后（含空格），loop.max.....之前。最后一个选项没有逗号，并与loop.max存在一个空格。

例如："cros_debug options=enable_updates,advanced_als loop.max....."将会开启这两个选项。

## 修改内核版本

通过配置菜单或手动修改Grub引导文件，你可以在多个内核间进行切换：
- kernel 5.4：理论上最稳定的默认内核。
- kernel 5.10：最新内核，若使用了第10代或更新的酷睿、以及AMD锐龙4000系列处理器时需要启用。
- kernel 4.19：上一代Brunch内核。
- kernel chromebook-5.4：用于Chromebook设备的最佳内核。
- kernel chromebook-4.4：用于兼容部分老款Chromebook的内核。
- kernel macbook：包含不同MacBook设备专用补丁的5.10内核。

译注：上述操作方式仅适用于从Grub引导菜单进入ChromeOS (settings)引导项。如果你需要从Grub配置文件里直接修改，请使用以下步骤：
1. 打开Grub2Win或在ChromeOS命令行里输入 `sudo edit-brunch-config` （见前文）进行编辑。
2. 找到配置文件的“/kernel”部分，将斜杠后的文字替换为你所需要的内核，所有空格均需要使用横杠代替（例如：“kernel-5.10”、“kernel-macbook”等）。
3. 保存并退出编辑，重启电脑。

警告：修改内核可能会导致你无法登陆ChromeOS账号，此时只能通过Powerwash格式化机器来修复（在登录界面按下Ctrl+Alt+Shift+R）。因此，在切换内核之前，请先确保你已经备份了所有数据。

## 内核命令行参数

以下内容与上述选项不同，如果符合以下情况，请将对应内容添加至Grub配置文件的“cros_debug”之后、“options=...”之前（使用“ChromeOS (Settings)”进入配置菜单的无需手动寻找添加位置，直接添加即可）。
最常见的内核命令行参数如下：
- "enforce_hyperthreading=1"：通过关闭ChromeOS的安全功能来强行开启超线程以提升性能（甚至在crositini下也是如此，会使所有CPU核心保持在最高频率，即便是空闲状态，电量消耗将严重增加）。
- "i915.enable_fbc=0 i915.enable_psr=0"：如果你使用了crouton(5.4内核需要)。
- "psmouse.elantech_smbus=1"：某些elantech的触摸板需要应用此修复。
- "psmouse.synaptics_intertouch=1"：在某些触摸板上开启2指以上的手势操作。

就像这样（仅针对手动编辑Grub配置文件）：
![](https://user-images.githubusercontent.com/69226625/97113026-9fec2880-170d-11eb-930f-972f0b38af4f.png)

## 引导图像

你可以通过切换此Repo的分支至你的Brunch版本，并进入“bootsplashes”文件夹，来预览到不同的引导图像。

# 更新Brunch框架/ChromeOS

## 检查已安装的Brunch框架版本

1. 打开ChromeOS Shell (Ctrl+Alt+T并输入 `shell` )。
2. 显示Brunch版本：
```
cat /etc/brunch_version
```

## 同时升级ChromeOS和Brunch框架

目前，我们只推荐在对应版本的Brunch框架推出后才升级ChromeOS。

1. 下载新版的ChromeOS恢复镜像并解压。
2. 下载与ChromeOS版本对应的Brunch框架（参见Release页面）。
3. 打开ChromeOS Shell (Ctrl+Alt+T并输入 `shell` )。
4. 同时升级ChromeOS和Brunch框架：
```
sudo chromeos-update -r < ChromeOS所在位置 > -f < Brunch框架压缩包所在位置 >
```
5. 重启ChromeOS。

提示：BiteDasher制作了一个脚本文件，使你可以通过一个命令完成Brunch和ChromeOS的升级：https://github.com/BiteDasher/brcr-update

## 仅升级Brunch框架

如果你使用了“enable_updates”选项并已经更新到新版ChromeOS，那么你可能需要更新Brunch框架来对应目前的ChromeOS版本。

1. 下载与ChromeOS版本对应的Brunch框架（参见Release页面）。
2. 打开ChromeOS Shell (Ctrl+Alt+T并输入 `shell` )。
3. 升级Brunch：
```
sudo chromeos-update -f < Brunch框架压缩包所在位置 >
```
4. 重启ChromeOS。


# 常见问题

1) 我不熟悉Linux命令行，这些说明太难了。

我没法说得太过具体，但一旦发现有什么明显的痛点，我也会尽可能说得更清楚。然而，ChromeOS本身就基于Linux，如果你事先学习了一点基础的Linux知识，这个项目可能对你来说才会更有意思。

2) 我已经按照步骤操作了，但我的电脑没有像以往那样正常引导进U盘/SD卡。

某些设备（比如Surface Go）在安全引导开启时，完全不会引导一个合法的U盘/SD卡，哪怕二进制已经签名了。对于这些设备，你需要在BIOS中手动关闭安全引导，并且在使用chromeos-install.sh安装时，在尾部加上“-l”参数来使用传统EFI引导程序。

3) 首次启动、框架变更或升级之后的引导时间太久了。

不幸的是，在每次重大改动之后，Brunch框架都需要通过拷贝原始rootfs、模块和固件文件来进行自我重建。其所需的时间根据你的U盘/SD卡速度的不同而不同。你可以试试使用更快的设备，或安装在你的内置硬盘里。

4) ChromeOS不定时重启。

这理论上有很多种可能性，当然，最大的可能是你的U盘/SD卡太慢了。你可以试试使用更快的设备，或安装在你的内置硬盘里。

5) 某些应用在商店里找不到（比如Netflix...）

为了能够访问ChromeOS Shell，ChromeOS默认就以开发者模式启动。如果你的系统足够稳定，你可以在GRUB的内核命令行中移除“cros_debug”（参见“修改GRUB引导程序”一节）并进行Powerwash（ChromeOS的一种机制，它会格式化ChromeOS的所有的数据分区）来关闭开发者模式。

6) Play商店上的某些应用与我的设备不兼容。

Play商店上的某些应用与通用的Chromebook设备不兼容本来就不是什么稀罕事。（译者注：ChromeOS往往运行在x86_64平台上，而Play商店上大多数应用都是基于ARM架构处理器编写的，因此经常会发生无法使用的情况）

7) Surface电脑有XX功能无法使用。

通常来说，由于Surface设备采用了定制硬件和专有驱动，导致其兼容性往往不如普通的设备。目前来说，你可能会遇到（包括但不限）触摸屏断触、误触、位移；摄像头无法使用；独立显卡无法使用；外设（如Surface Dial、外接显示屏等）工作不正常之类的问题。对于某些问题，可能可以通过强制使用K5.10内核来应付，但长远来看，还是必须要等待Brunch框架的后续更新。

8) 在ChromeOS跑码时，提示“ChromeOS was not found on any device”。

在确保Grub、ChromeOS镜像本身的安装和配置都完全正确的前提下，出现此行提示则证明你的CPU与当前的ChromeOS镜像版本（下载时的设备代号）不兼容。请尝试换用其他设备代号的镜像，或等待可能存在的后续支持。
