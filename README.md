# Arch Linux 鸣潮 (Wuthering Waves) 完整游玩与避坑实战指南

> 本指南基于实际踩坑经验整理，涵盖客户端下载、特化兼容层（DWProton）安装、Lutris 配置、中文字体排版与乱码修复、画质/性能调优以及日常更新与维护。
> 适用于 Arch Linux 及基于 Arch 的发行版，原生运行，不触碰/篡改任何游戏内部资源，百分之百规避腾讯 ACE 反作弊风险。

---

## 目录
- [一、核心理念与避坑必读](#一核心理念与避坑必读)
- [二、准备工作与环境依赖](#二准备工作与环境依赖)
- [三、安装特化兼容层：DWProton](#三安装特化兼容层dwproton)
- [四、下载与管理游戏客户端（ww-manager）](#四下载与管理游戏客户端ww-manager)
- [五、在 Lutris 中添加与配置游戏](#五在-lutris-中添加与配置游戏)
- [六、关键问题修复（必看避坑）](#六关键问题修复必看避坑)
  - [1. 协议正文乱码（åœ¨...）与中文字号忽大忽小](#1-协议正文乱码åœ¨与中文字号忽大忽小)
  - [2. 强制改成 uhd 导致虚幻引擎 Fatal Error 崩溃](#2-强制改成-uhd-导致虚幻引擎-fatal-error-崩溃)
  - [3. 登录界面多账号下拉列表文字空白](#3-登录界面多账号下拉列表文字空白)
- [七、性能与画质调优指南](#七性能与画质调优指南)
- [八、日常维护与更新工作流](#八日常维护与更新工作流)

---

## 一、核心理念与避坑必读

1. **绝对不要直接把官方启动器放到 Wine/Lutris 里运行下载**：
   官方启动器内嵌 WebView2 组件且常伴随驱动级服务调用，在 Wine 下容易卡 0%、白屏死锁或解压报错。
2. **切勿把游戏放在 Windows 的 NTFS 分区运行**：
   Linux 挂载 NTFS 存在权限映射、套接字（Socket）通信阻塞问题，极易引发游戏无法联网或崩溃。**务必安装在 Linux 原生文件系统（如 ext4 / btrfs）**。
3. **兼容层选择**：
   常规 GE-Proton 播放鸣潮剧情视频时常因 Media Foundation 解码失败导致“丢失/跳过 CG”；建议使用二游特化兼容层 **DWProton**。
4. **安全底线**：
   不修改游戏内部任何 `.pak` 或二进制文件，杜绝腾讯 ACE 反作弊封号风险。

---

## 二、准备工作与环境依赖

确保系统中已安装基础图形与包管理工具：

```bash
# 1. 安装基础工具与 pipx
sudo pacman -S --needed wine-staging lutris python-pipx

# 2. 安装兼容层管理工具 ProtonUp-Qt（通过 AUR）
paru -S protonup-qt-bin
# 或使用 yay:
# yay -S protonup-qt-bin
```

---

## 三、安装特化兼容层：DWProton

DWProton（Dawn Winery Proton）专为二次元抽卡游戏定制，内置专有视频解码与着色器优化补丁。

1. 打开 **ProtonUp-Qt**。
2. 在底部的 **Install for** 下拉菜单中选择 **Lutris**（对应路径为 `~/.local/share/lutris/runners/wine`）。
3. **开启高级模式（关键）**：
   * 点击界面下方的 **【About】** 按钮；
   * 勾选 **【Enable Advanced Mode】（启用高级模式）**，然后点击 **Close**。*(因为 DWProton 属于特化工具，未开高级模式时会被默认隐藏)*
4. 点击主界面左下角的 **【Add version】**：
   * **Compatibility tool**：选择 **`dwproton`**；
   * **Version**：选择列表中最顶部的最新版本（如 `dwproton-11.0-14`）；
   * 点击 **【Install】** 等待自动下载解压完成。
5. 安装完成后，**彻底关闭并重新打开一次 Lutris** 以便其刷新识别。

---

## 四、下载与管理游戏客户端（ww-manager）

使用社区开源的高性能 CLI 管理工具 [wutheringwaves-cli-manager](https://github.com/timetetng/wutheringwaves-cli-manager)，原生多线程高速下载，免除启动器烦恼。

### 1. 安装与环境变量配置
```bash
# 通过 pipx 一键安装
pipx install ww-manager

# 将 pipx 的二进制路径加入系统环境变量
pipx ensurepath
source ~/.bashrc
```

### 2. 下载完整客户端
创建专门的游戏存放目录并执行下载（以官服为例）：

```bash
# 创建官服目录
mkdir -p ~/Games/WutheringWaves-CN

# 开始下载官服（工具会自动下载 80+ GB 完整客户端并校验解压）
ww -p ~/Games/WutheringWaves-CN download cn
```
*(如果是 B 站服，可将 `download cn` 改为 `download bilibili`)*

> [!TIP]
> **关于官服与 B 站服一秒切换（省去额外 80GB 空间）：**
> 鸣潮两服 99% 的资产完全一致，工具自带秒级换服功能：
> * 切换至 B 站服：`ww checkout bilibili`（首次会自动拉取几兆专有 SDK 差异文件并缓存）；
> * 切回官服：`ww checkout cn`。

---

## 五、在 Lutris 中添加与配置游戏

1. 点击 Lutris 窗口左上角的 **`+`** -> **Add locally installed game**。
2. **【Game info】标签页**：
   * **Name**：输入 `鸣潮`；
   * **Runner**：选择 `Wine (Runs Windows games)`；
   * 其余项全部留空。
3. **【Game options】标签页**：
   * **Executable**：选择客户端主程序（**注意：千万不要选 launcher.exe**）：
     ```text
     /home/你的用户名/Games/WutheringWaves-CN/Client/Binaries/Win64/Client-Win64-Shipping.exe
     ```
   * **Arguments（启动参数）**：填入绕过启动器与渲染参数：
     ```text
     -krqlv=hd -ResX=1920 -ResY=1080 -fullscreen
     ```
   * **Wine prefix**：指定一个前缀目录（Wine 会自动在此生成虚拟 C 盘）：
     ```text
     /home/你的用户名/Games/Prefixes/wuthering-waves
     ```
4. **【Runner options】标签页**：
   * **Wine version**：选择安装好的 **`dwproton-11.0-14-x86_64`**；
   * 确保 **Enable DXVK** 与 **Enable Fsync** 保持打开。
5. **【System options】标签页**：
   * 滚动到下方的 **Environment variables**，点击 **Add** 添加环境变量：
     * **Key**: `LANG`
     * **Value**: `zh_CN.UTF-8`
6. 点击右上角 **【Save】** 保存。

---

## 六、关键问题修复（必看避坑）

### 1. 协议正文乱码（`åœ¨...`）与中文字号忽大忽小
* **现象**：协议正文是一堆带有拉丁字母的乱码；登录界面中文有的字大、有的字小。
* **原因**：系统缺少 `zh_CN.UTF-8` 导致 Wine 降级使用西欧编码；同时 Proton 自带的中文字体为开源替身，字号基线不一。
* **解决办法**：
  1. 生成中文 Locale：
     ```bash
     sudo sed -i 's/^#zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/' /etc/locale.gen
     sudo locale-gen
     ```
  2. 补齐微软原版微软雅黑字体（在终端执行，下载至前缀）：
     ```bash
     cd ~/Games/Prefixes/wuthering-waves/drive_c/windows/Fonts/
     curl -L -O https://raw.githubusercontent.com/fernvenue/microsoft-yahei/master/msyh.ttc
     curl -L -O https://raw.githubusercontent.com/fernvenue/microsoft-yahei/master/msyhbd.ttc
     curl -L -O https://raw.githubusercontent.com/fernvenue/microsoft-yahei/master/msyhl.ttc
     ln -sf msyh.ttc msyh.ttf
     ln -sf msyhbd.ttc msyhbd.ttf
     ```

### 2. 强制改成 `uhd` 导致虚幻引擎 Fatal Error 崩溃
* **现象**：把启动参数改为 `-krqlv=uhd` 后启动游戏必弹 `Fatal error!` 崩溃退出。
* **原因**：`-krqlv` 是库洛画质档位参数。`ww-manager` 与常规渠道下载的是标准高清包（`hd`）。本地没有下载超高清资源包时，强行传 `uhd` 会导致引擎索引材质失败抛出断言崩溃。
* **解决办法**：
  * **启动参数必须保持 `-krqlv=hd`**。
  * 游戏内部的高清渲染与特效在游戏内的【图像设置】拉满即可，丝毫不影响画质。

### 3. 登录界面多账号下拉列表文字空白
* **现象**：点击手机号下拉菜单，第一项显示正常，第 2/3/4 项只有手机图标和减号，文字隐形。
* **原因**：这是 Wine 在离屏自绘列表（OwnerDraw ComboBox）上的经典渲染已知缺陷：未选中项的透明背景文字被 Wine 当成透明丢弃，但底层数据完好无损。
* **解决办法**：
  * 展开下拉菜单后，**直接按键盘上的【下方向键 `↓`】或【上方向键 `↑`】**，光标移动到哪一项，哪一项就会被瞬间激活并完整显现出手机号！
  * 也可以直接鼠标“盲点”对应行进行账号切换。

---

## 七、性能与画质调优指南

针对 Linux + Wayland / 中端显卡避免微卡顿（Stuttering）的最佳实践：

### 1. 关闭游戏内“垂直同步”（极度关键，解决画面发粘与微卡顿）
* **原理**：Wayland（如 KDE Plasma / GNOME Wayland）合成器底层已强制带有无撕裂同步。如果在游戏里再开启垂直同步，会造成**双重 VSync 阻塞**，导致严重的输入粘滞和帧节奏抖动。
* **做法**：进入游戏【设置 -> 图像】，**将【垂直同步】彻底关闭（OFF）**。

### 2. 解决无 ReBAR 老平台的显存总线塞车（如老 Xeon / RX 500 系列）
如果你的硬件平台不支持 Resizable BAR (SAM)，PCIe 可见显存窗口仅有 256MB，跑 DX12 会引起总线排队卡顿。
* 在 Lutris -> **System options** -> **Environment variables** 添加：
  * **Key**: `VKD3D_CONFIG`
  * **Value**: `no_upload_hvv,nodxr`

### 3. 游戏内画质参数推荐
在游戏内【设置 -> 图像设置】：
* **显示模式**：`1920x1080 全屏`
* **渲染精度**：`最高 (1.0)`（千万不要开低，开低会导致严重发虚发糊）
* **FSR / 超分辨率**：`关闭`（1080P 下开启 FSR 涂抹感严重）
* **动态模糊**：`关闭`
* **体积雾 / 阴影质量**：中端显卡建议设为 `中` 或 `高`（极高档算力消耗极大）

---

## 八、日常维护与更新工作流

### 1. 游戏版本更新（解耦系统，随时更新）
* **日常检查与完整校验**：
  ```bash
  ww sync
  ```
  *(若显示 `所有文件校验通过，无需下载`，说明即是最新版)*
* **大版本停服维护前（预下载）**：
  在官方公告开启预下载后执行：
  ```bash
  ww predownload
  ```

### 2. DWProton 升级
* 当二游大版本更新带来新的反作弊机制变动时，打开 **ProtonUp-Qt**，在 Lutris 分类下点 `Add version` 下载最新版 `dwproton`；
* 在 Lutris 的 Runner options 切换到新版本即可，**不需要重启系统**。

### 3. 系统维护频率建议
* 游戏与兼容层独立于系统，平时无需天天滚动更新；
* 建议以**一个月为周期**（或空闲周末）集中执行一次全盘升级：
  ```bash
  yay -Syu
  ```
* 避免单挑某个软件做局部升级（Partial Upgrade），以保持 Arch 系统长期健壮稳定。

---

## 九、关于风险评估

本指南记录了我在 Arch Linux 环境下使用 Wine/Proton 兼容层运行《鸣潮》的个人实践。本指南不代表官方对该运行方案的支持或认可，兼容性可能随游戏客户端、反作弊组件及兼容层的更新而发生变化。

游戏能够正常启动或运行，并不代表该方案已获得官方认可，也不代表不存在账号风险。请读者在使用前自行评估相关风险，并以官方公告及客服答复为准。
