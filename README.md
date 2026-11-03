# VeilCue

**本地字幕工作台** —— 视频 → 字幕 → 翻译 → 烧录，全流程在你自己机器上跑。

Windows x64 · 免安装 · 不写注册表 · 不碰 C 盘

---

> ⚠️ **开发测试中 —— 软件本体尚未发布**
>
> 这个仓库目前只对外提供 **GPU 加速包**；基础包（`VeilCue-1.0.0-win64.zip`）
> 要等测试收尾后再上传，源码也还没放出来。
> 想试用的请稍等，或者先看下面的说明了解它能做什么。

---

## 它做什么

| 环节 | 支持什么 |
|---|---|
| **取字幕** | 内封字幕轨 · **硬字幕 OCR**（自动定位字幕条，也可手动框选）· **语音识别**（本地模型或云端 ASR） |
| **翻译** | 官方免费通道 · **本地模型**（llama.cpp，可挂 GPU 加速包）· 自定义 OpenAI 兼容接口 |
| **整理** | 字幕编辑器（逐条改、低置信度标黄、双语对照）· **说话人分离**（多人对谈自动标 `[说话人 1]`） |
| **输出** | SRT / 烧录进视频（NVENC 或 x264）· 批量转写与翻译 |

另外带一个 **浏览器扩展**：在 B 站 / YouTube 播放器上叠 AI 译文，翻译仍然在桌面端做。

---

## 下载与安装

> 基础包**待发布**。Releases 里现在的 5 个 zip 是 GPU 加速包，要等基础包出来才能用。

装好基础包之后（流程如下，预计下个版本可用）：

去 [Releases](https://github.com/NekoTumble/VeilCue/releases) 下 `VeilCue-1.0.0-win64.zip`（约 22 MB）。

```
解压 → 双击 veilcue.exe
```

三点务必注意：

1. **整个文件夹一起用** —— 目录里 18 个 `.dll` 缺一个就打不开（缺了会在启动前弹"找不到 xxx.dll"，连日志都没有）。
2. **别放在需要管理员权限的位置**（例如 `C:\Program Files`）—— 程序把 `config.toml`、`models\`、`logs\` 写在自己目录里，写不进去会起不来或设置存不住。
3. **解压时用"全部解压"**，别在压缩包里直接双击。

首次启动会自动生成 `config.toml`；API Key、模型都在界面的「设置」和「接口与通道」里填。

---

## 模型

模型不随包（体积太大）。两种拿法：

- 界面「模型与设备」页**点下载**，会自动落到 `models\`；
- 或者手动放到 `models\` 对应子目录。

首次跑某个环节要加载模型，会慢几秒；之后常驻内存。

---

## GPU 加速（可选，NVIDIA）

默认用 CPU，开箱即用。想要本地翻译/识别快很多，装加速包：

1. 界面里会显示**本机算力**对应哪一档（`sm_75` / `80` / `86` / `89` / `120a`）；
2. 从 [Releases](https://github.com/NekoTumble/VeilCue/releases) 下对应的 `ggml-cuda-<架构>.zip`；
3. 解压，让路径变成 `<软件目录>\accel\<架构>\ggml-cuda.dll` + `accel.json`；
4. 重启软件。

**CUDA 12.x 运行库和显卡驱动要自己装** —— 我们不随包带（那有 783 MB）。

---

## 浏览器扩展

`apps/extension/` 是一份纯 JS 的 Chromium 扩展（Chrome / Edge，111+），零构建链。

1. `chrome://extensions` → 打开"开发者模式" → "加载已解压的扩展程序" → 选 `apps/extension` 目录；
2. 桌面端「接口与通道」→「本机接口」卡片里复制令牌，粘到扩展的设置页；
3. 打开 B 站 / YouTube 视频，播放器上会出现一颗按钮。

---

## 从源码构建

只支持 Windows。需要 Rust 工具链和 VS 生成工具。

```powershell
# 静态形状（CUDA 编进 exe，只认一个架构，开发/验收用）
powershell -ExecutionPolicy Bypass -File ./scripts/build-desktop-r2t2.ps1

# 交付形状（动态后端：exe 只 15 MB，CUDA 走 accel\<arch>\ 加速包）
powershell -ExecutionPolicy Bypass -File ./scripts/build-desktop-r2t2.ps1 -Release -Dynamic
```

⚠️ **别裸跑 `cargo build -p veilcue`**：不带 `--features r2t2` 会编出没有识别后端的版本，
而且 Tauri 把 `apps/desktop/ui/**` 内嵌进 exe、cargo 不把前端当构建输入 ——
只改前端时也要走上面的脚本，否则交出去的还是旧界面。

---

## 隐私

- 取字幕（OCR / 语音识别）、本地翻译、烧录 **全部在本机完成**，不联网。
- 只有当你把翻译通道切成**云端 / 自定义 API**时，字幕文本才会发到你配的那个地址。
- `config.toml` 里有你的 API Key，**别外传**（它已在 `.gitignore` 里）。

---

## 已知边界

- **水印 / 台标**：与字幕处在同一条水平带的文字会被一起认进来（属素材问题，可用「手动框选」把区域收到字幕那一条）。
- **只支持 Windows x64**。
- 语音识别与本地翻译的速度取决于你的 CPU / 显卡；CPU 跑长视频会比较慢。

---

## 反馈

有问题请开 [Issue](https://github.com/NekoTumble/VeilCue/issues)，附上 `logs\` 目录里当天的日志。
