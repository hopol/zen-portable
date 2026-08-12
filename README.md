# 🌟 Zen Browser Portable — 绿色便携版

> **上游项目**：[Zen Browser](https://github.com/zen-browser/desktop) — 基于 Mozilla Firefox 引擎的多工作区生产力浏览器
> **许可证**：[MPL-2.0](LICENSE)（Mozilla Public License 2.0）

本仓库跟踪上游 Zen Browser 的每个发布，通过 GitHub Actions CI **下载上游预构建的 Release 二进制包**，重新封装为 **不污染系统、解压即用** 的绿色便携版本。

---

## ✅ 绿色便携版

| 特性 | 状态 | 说明 |
|------|------|------|
| 🚫 不污染系统 | ✔️ | 不写注册表、不创建系统服务、不修改 AppData |
| 📁 数据自包含 | ✔️ | Profile、主题、书签、扩展缓存全部在程序同级目录 |
| ▶️ 解压即用 | ✔️ | 无需安装，便携启动器一键运行 |
| 🔄 便携迁移 | ✔️ | 整目录拷贝即可完成迁移 |
| 🌐 全平台 | ✔️ | Windows / macOS / Linux |

---

## 🔧 便携版工作原理

### 绿色便携版 vs 标准版

| 标准版 | 绿色便携版 |
|--------|-----------|
| 通过安装包安装到系统目录 | 解压到任意目录（U盘/移动硬盘/任意位置） |
| 配置在 `%APPDATA%`/`~/Library`/`~/.mozilla` | **全部在 exe 同级目录的 `zen-portable-data/` 文件夹中** |
| 写注册表、写系统服务 | **不写任何注册表、不创建系统服务** |
| 卸载后残留配置文件 | **整目录删除即卸载，无残留** |

### 便携模式实现原理

Mozilla 系浏览器（包括 Zen Browser）通过 **Profile 系统** 管理用户数据。便携版通过**启动器脚本**强制重定向路径：

| 启动方式 | 运行模式 | 数据存放位置 |
|----------|----------|-------------|
| 通过启动器（`.bat` / `.sh` / `.command`） | **便携模式** | exe 同级 `zen-portable-data/` |
| 直接运行 `zen.exe` / `zen` | **标准模式** | 系统默认 Profile 目录 |

> **启动器脚本**通过显式设置 `MOZILLA_PROFILE` 环境变量并传递 `--profile` 命令行参数，强制 Zen Browser 使用便携数据目录。`.portable` 文件仅作为便携版标记，浏览器本身不会读取该文件。

### 路径重定向的实现

便携封装由 GitHub Actions CI 完成，具体步骤：

1. 从 `https://github.com/zen-browser/desktop/releases` 下载上游预构建的 Release 二进制包（Windows 安装包 / macOS DMG / Linux tarball）
2. 在 staging 目录中添加：
   - `.portable` 标记文件
   - 平台对应的便携启动器脚本（`.bat` / `.sh` / `.command`）
   - `README.txt` 使用说明
3. 重新打包为便携版压缩包
4. 生成 SHA256 校验文件并上传

---

## 🚀 快速开始

### Windows

```powershell
# 解压 zen-browser-portable-windows-amd64-<tag>.zip
# 双击 zen-portable.bat 运行
```

解压后目录结构：
```
zen.exe                ← 主程序
zen-portable.bat       ← 便携启动器（双击运行）
zen-portable.cmd       ← 便携启动器（备用）
.portable              ← 便携模式标记文件（不要删除）
zen-portable-data/     ← 首次运行后自动创建的便携数据目录
README.txt             ← 使用说明
```

### macOS

```bash
# 解压后双击 zen-portable.command 运行
# ⚠️ 不要直接双击 Zen.app，否则会以标准模式运行（数据写入系统目录）
```

### Linux

```bash
tar xzf zen-browser-portable-linux-amd64-<tag>.tar.gz
./zen-portable.sh
```

---

## 🛠️ 项目结构

```
zen-portable/
├── .github/
│   └── workflows/
│       └── portable-release.yml   ← GitHub Actions 工作流（三平台并行）
├── docs/
│   └── 绿色便携版技术原理.md       ← 便携技术原理深度解析
├── .gitignore
├── LICENSE                        ← MPL-2.0 开源协议
└── README.md                      ← 项目主文档
```

---

## 🔄 CI 构建流程

### 触发方式

| 触发方式 | 条件 | 行为 |
|----------|------|------|
| 定时触发 | 每周一 08:00 UTC | 自动拉取上游最新 Release |
| 手动触发 | Actions 页面手动运行 | 可指定上游版本号（tag） |

### 构建流水线

```
触发构建
    ↓
[prepare] 解析上游 tag
    ↓
┌─ build-macos ──────────────────────────────────────┐
│  1. GitHub API 获取 macOS DMG 下载链接               │
│  2. curl 下载 zen.macos-universal.dmg               │
│  3. hdiutil 挂载 DMG，提取 Zen.app                  │
│  4. 添加 .portable + zen-portable.command + README  │
│  5. 打包为 zen-browser-portable-macos-<tag>.zip    │
│  6. 生成 SHA256 校验文件                            │
│  7. softprops/action-gh-release 发布                │
└────────────────────────────────────────────────────┘
┌─ build-windows ────────────────────────────────────┐
│  矩阵：x64 / arm64（fail-fast: false）              │
│  1. 下载 zen.installer.exe（或 -arm64）             │
│  2. 7-Zip 解压 NSIS 安装包                         │
│  3. 复制程序文件到 staging                          │
│  4. 添加 .portable + zen-portable.bat + README      │
│  5. 打包为 zen-browser-portable-windows-<arch>.zip │
│  6. 生成 SHA256 校验文件                            │
│  7. 发布                                            │
└────────────────────────────────────────────────────┘
┌─ build-linux ──────────────────────────────────────┐
│  矩阵：amd64 / arm64                                │
│  1. 下载 zen.linux-x86_64.tar.xz（或 aarch64）     │
│  2. tar xJf 解压                                    │
│  3. 添加 .portable + zen-portable.sh + README       │
│  4. 打包为 zen-browser-portable-linux-<arch>.tar.gz │
│  5. 生成 SHA256 校验文件                            │
│  6. 发布                                            │
└────────────────────────────────────────────────────┘
```

### 构建产物

| 文件 | 平台 | 架构 |
|------|------|------|
| `zen-browser-portable-macos-<tag>.zip` | macOS | Universal（Intel + Apple Silicon） |
| `zen-browser-portable-windows-amd64-<tag>.zip` | Windows | x86_64 |
| `zen-browser-portable-windows-arm64-<tag>.zip` | Windows | ARM64 |
| `zen-browser-portable-linux-amd64-<tag>.tar.gz` | Linux | x86_64 |
| `zen-browser-portable-linux-arm64-<tag>.tar.gz` | Linux | ARM64 |
| `SHA256SUMS` | — | — |

---

## 📦 从标准版迁移

从标准版 Zen Browser 迁移到绿色便携版：

| 平台 | 标准版 Profile 位置 | 便携版目标位置 |
|------|--------------------|---------------|
| Windows | `%APPDATA%/Mozilla/Zen/Profiles/` | exe 同级 `zen-portable-data/` |
| macOS | `~/Library/Application Support/Zen/Profiles/` | exe 同级 `zen-portable-data/` |
| Linux | `~/.mozilla/zen/` | exe 同级 `zen-portable-data/` |

将整个标准版 Profile 目录拷贝到便携版 `zen-portable-data/` 即可。

---

## 🚫 已知限制

1. **上游发布驱动**：便携版只能基于上游已发布的版本生成，无法提供上游尚未发布的特性。
2. **构建包大小**：便携版压缩包大小与上游安装包相当（Windows ~90 MB，Linux ~90 MB，macOS ~200 MB）。
3. **硬件加速**：部分 Linux 发行版需额外安装 GPU 驱动和库以获得最佳性能。
4. **macOS 安全**：macOS 首次运行 `zen-portable.command` 可能需前往「系统设置 → 隐私与安全性」中点击「仍要打开」。

---

## 📄 许可证

本项目采用 [MPL-2.0](LICENSE)（Mozilla Public License 2.0）开源协议。

---

*本文档由仓库代码与配置分析生成，如有出入请以 [GitHub 仓库](https://github.com/zen-browser/desktop) 为准。*
