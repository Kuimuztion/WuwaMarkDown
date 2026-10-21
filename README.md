# Fedora/ArchLinux 游玩鸣潮方案

本指南基于 Fedora 44（KDE Plasma 6 / Wayland）实测整理，同时标注了 Arch Linux 及其衍生发行版的对应操作。
原生运行，不需要官方启动器，不修改游戏内任何 .pak 或二进制文件，尽量规避腾讯 ACE 反作弊风险。

## 目录

- 一、核心理念与避坑必读
- 二、准备工作与环境依赖
- 三、安装特化兼容层 DWProton
- 四、下载与管理游戏客户端（ww-manager）
- 五、在 Lutris 中添加与配置游戏
- 六、关键问题修复
- 七、Wayland 性能与画质调优
- 八、日常维护与更新
- 九、关于风险评估

## 一、核心理念与避坑必读

### 1. 不要把官方启动器放到 Wine/Lutris 里运行下载。

官方启动器内嵌 WebView2 组件，且常调用驱动级服务，在 Wine 下容易卡 0%、白屏死锁或解压校验报错。

### 2. 不要把游戏放在 Windows 的 NTFS 分区运行。

Linux 挂载 NTFS 存在权限映射和 Socket 通信阻塞问题，容易导致无法联网或引擎闪退。请安装在 Linux 原生文件系统（ext4 / btrfs）上。

### 3. 兼容层首选 DWProton。

常规 GE-Proton 播放剧情 CG 时，常因 Media Foundation 解码失败而跳过或丢失视频。DWProton 是二游特化兼容层，内置专有解码补丁。

### 4. 安全底线。

不修改游戏内任何 .pak 或二进制文件，避免反作弊封号风险。

## 二、准备工作与环境依赖

Fedora 44 的安装命令如下（在终端执行）：

```bash
sudo dnf install lutris wine pipx glibc-langpack-zh mesa-vulkan-drivers
```

说明：Fedora 使用 glibc 语言包模块化管理，安装 glibc-langpack-zh 即可提供完整的 zh_CN.UTF-8 支持，无需手动生成 locale。

ProtonUp-Qt 在 Fedora 官方源中没有收录。为避免 Flatpak 沙盒的路径权限问题，推荐直接使用官方 AppImage，与 RPM 安装的 Lutris 配合良好：

```bash
mkdir -p ~/.local/bin
curl -sL https://api.github.com/repos/DavidoTek/ProtonUp-Qt/releases/latest | grep "browser_download_url.*x86_64\.AppImage\"" | cut -d : -f 2,3 | tr -d '\" ' | xargs -I {} curl -L -o ~/.local/bin/protonup-qt {}
chmod +x ~/.local/bin/protonup-qt
```

完成后在终端输入 protonup-qt 即可启动，也可以自行创建桌面快捷方式。

Arch Linux 及衍生发行版的对应操作：

```bash
sudo pacman -S --needed wine-staging lutris python-pipx
paru -S protonup-qt-bin（或 yay -S protonup-qt-bin）
```

## 三、安装特化兼容层 DWProton

DWProton（Dawn Winery Proton）专为二次元抽卡游戏定制，内置专有视频解码与着色器补丁。

### 1. 打开 ProtonUp-Qt。

2. 在底部 Install for 下拉菜单中选择 Lutris（对应路径为 ~/.local/share/lutris/runners/wine）。
3. 开启高级模式（关键）：点击右下角的 About，勾选 Enable Advanced Mode，然后点击 Close。未开启高级模式时 DWProton 会被隐藏。
4. 点击左侧的 Add version：Compatibility tool 选择 dwproton；Version 选择列表最顶部的最新稳定版（如 dwproton-11.0-14）；点击 Install 等待下载解压完成。
5. 完成后彻底关闭并重新启动一次 Lutris，以刷新识别兼容层。

## 四、下载与管理游戏客户端（ww-manager）

使用开源 CLI 工具 wutheringwaves-cli-manager（https://github.com/timetetng/wutheringwaves-cli-manager），支持原生多线程高速下载，不依赖官方启动器。

### 1. 安装与环境配置：

```bash
pipx install ww-manager
pipx ensurepath
source ~/.bashrc
```

若使用 zsh，请将 ~/.bashrc 替换为 ~/.zshrc。

### 2. 下载完整客户端。建议统一存放在家目录的 ~/Games 下：

```bash
mkdir -p ~/Games/WutheringWaves-CN
ww -p ~/Games/WutheringWaves-CN download cn
```

B 站服将末尾的 download cn 改为 download bilibili。

官服与 B 站服的资源约 99% 一致，无需重复占用 80GB 空间：
切换至 B 站服：在游戏根目录执行 ww checkout bilibili（首次会自动拉取几兆 SDK 差异包并缓存）
切回官服：ww checkout cn

## 五、在 Lutris 中添加与配置游戏

### 1. 点击 Lutris 左上角的 + 号，选择 Add locally installed game。

### 2. Game info 标签页：

Name 填写 鸣潮（或自定义名称）
Runner 选择 Wine (Runs Windows games)

### 3. Game options 标签页：

Executable 指向 Win64 核心程序，切勿选择 launcher.exe：
/home/你的用户名/Games/WutheringWaves-CN/Client/Binaries/Win64/Client-Win64-Shipping.exe

Arguments 启动参数：
-krqlv=hd -ResX=1920 -ResY=1080 -fullscreen

Wine prefix 独立的虚拟 C 盘目录：
/home/你的用户名/Games/Prefixes/wuthering-waves

### 4. Runner options 标签页：

Wine version 选择 dwproton-11.0-14-x86_64（或你安装的对应版本）
保持 Enable DXVK 和 Enable Fsync 开启。

### 5. System options 标签页，在 Environment variables 中点击 Add，依次添加两组变量：

第一组（中文环境）：Key 为 LANG，Value 为 zh_CN.UTF-8
第二组（无 ReBAR 老平台必备）：Key 为 VKD3D_CONFIG，Value 为 no_upload_hvv,nodxr

### 6. 点击右上角 Save 保存。

## 六、关键问题修复

### 1. 协议正文乱码与中文字号忽大忽小

现象：用户协议变成拉丁字符乱码（如 åœ¨...），登录界面中文大小不一。
原因：系统缺少中文编码支持，Wine 会降级回退到西欧编码；同时 Proton 自带的中文替身字体基线高度不一致。

解决步骤：

确认已安装中文语言包（Fedora）：
sudo dnf install glibc-langpack-zh

Arch 系统请生成中文 Locale：
sudo sed -i 's/^#zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/' /etc/locale.gen
sudo locale-gen

首次在 Lutris 启动一次游戏并退出（以生成 Wine 前缀），然后将微软雅黑写入前缀：

mkdir -p ~/Games/Prefixes/wuthering-waves/drive_c/windows/Fonts/
cd ~/Games/Prefixes/wuthering-waves/drive_c/windows/Fonts/
curl -L -O https://raw.githubusercontent.com/fernvenue/microsoft-yahei/master/msyh.ttc
curl -L -O https://raw.githubusercontent.com/fernvenue/microsoft-yahei/master/msyhbd.ttc
curl -L -O https://raw.githubusercontent.com/fernvenue/microsoft-yahei/master/msyhl.ttc
ln -sf msyh.ttc msyh.ttf
ln -sf msyhbd.ttc msyhbd.ttf

### 2. 启动参数改成 uhd 后虚幻引擎 Fatal Error 崩溃

现象：启动参数传入 -krqlv=uhd 后必弹 Fatal error! 退出。
原因：-krqlv 是画质档位标记。标准渠道和 ww-manager 下载的是高清包（hd）。没有超高清资源包时强制传入 uhd，引擎索引材质会断言崩溃。
解决办法：启动参数保持 -krqlv=hd，在游戏设置里把画质拉高即可，渲染精度不受影响。

### 3. 登录界面多账号下拉列表文字空白

现象：多账号下拉菜单中，除第一项外其他项只有图标，文字看不见。
原因：Wine 在离屏自绘组合框（OwnerDraw ComboBox）上对透明背景文字的渲染缺陷，数据本身是完整的。
解决办法：展开下拉列表后，按键盘上的 下方向键 或 上方向键，光标所在项会高亮并完整显示账号文本；也可以直接鼠标点击对应行切换。

## 七、Wayland 性能与画质调优

针对 Fedora 44 默认的 KDE Plasma 6（Wayland 会话）：

### 1. 关闭游戏内垂直同步。

原理：KWin-Wayland 合成器本身已提供无撕裂的帧呈现。游戏内再开垂直同步会造成双重 VSync 阻塞，导致输入延迟、画面粘滞和帧节奏波动。
做法：进入游戏【设置 -> 图像】，将垂直同步设为 OFF。

### 2. 解决无 ReBAR 老平台的显存总线塞车。

原理：老平台不支持 Resizable BAR（SAM），PCIe 可映射的 Host-Visible VRAM 只有 256MB，DX12（VKD3D-Proton）容易因显存换页排队而卡顿。
做法：确保 Lutris 环境变量中已加入 VKD3D_CONFIG=no_upload_hvv,nodxr。该参数禁用了慢速的小窗口上传通道，并关闭光追模拟开销，能明显提升帧率稳定性。

### 3. 游戏内画质推荐（设置 -> 图像设置）：

显示模式：1920x1080 全屏
帧率上限：60 FPS
渲染精度：最高（1.0），千万不要调低，低于 1.0 会明显模糊
FSR / 超分辨率：关闭
动态模糊：关闭
体积雾 / 阴影质量：建议中或高

## 八、日常维护与更新

### 1. 游戏更新与校验：

```bash
ww -p ~/Games/WutheringWaves-CN sync
```

若输出校验全部通过，说明已是最新版。大版本停服更新前，可在官方开启预下载通道后执行：

```bash
ww -p ~/Games/WutheringWaves-CN predownload
```

Arch 用户可直接在游戏目录执行 ww sync 与 ww predownload。

### 2. DWProton 升级：大版本更新或反作弊策略调整时，打开 protonup-qt，在 Lutris 分类下点击 Add version 下载最新 dwproton，然后在 Lutris 的 Runner options 中切换即可，无需重启系统。

### 3. 系统更新频率：游戏与兼容层与系统相互独立，平时无需频繁升级系统。建议每月或空闲时运行一次系统更新。

Fedora：sudo dnf upgrade
Arch：yay -Syu（避免局部升级 Partial Upgrade）

## 九、关于风险评估

本指南是我在 Fedora 44 和 Arch Linux 环境下使用 Wine/Proton 兼容层运行《鸣潮》的个人实践总结。
该方案不代表官方的支持或认可，兼容性可能随游戏客户端、反作弊组件及 Wine 运行时的更新而变化。
游戏能正常启动，不代表不存在账号风险。请在游玩前自行评估相关风险，并以官方公告和客服答复为准。
