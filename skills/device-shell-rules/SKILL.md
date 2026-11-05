---
name: device-shell-rules
description: 本机 DevCodex 环境强制规则。Termux 沙盒已损坏（ld 缺 libreadline.so.8，直接报 CANNOT LINK EXECUTABLE），所以严禁使用 linux_exec、python_run、node_run、pipeline_run、sandbox_pkg_install；所有 shell 操作一律用 shell_exec（Shizuku ADB，uid 2000）。任何需要 shell、脚本、文本处理、JSON 校验、联网下载的任务，第一时间就走 shell_exec。此外 Android 的 sed/awk/cut 处理超长行（>4万字节）会静默截断，禁止用来解析大 JSON；手机无 root 无法读写 /data/user/0 私有目录；应用工作区 /storage/emulated/0/Android/data/ai.devcodex.agent/files/workspace 可直接读写。
---

# 本机环境规则（DevCodex on Android）

这份技能的唯一目的：让 AI 在**每个新对话的第一轮**就知道该用哪个执行通道，不用等用户提醒。

## 硬事实（已实测）

| 通道 | 状态 | 证据 |
|---|---|---|
| `shell_exec`（Shizuku ADB） | ✅ 可用 | uid=2000(shell)，adb 组权限，可 pm/am/ls/cat/curl |
| `linux_exec` | ❌ 坏 | 缺 `libreadline.so.8` |
| `python_run` | ❌ 坏 | `CANNOT LINK EXECUTABLE ... bash` |
| `node_run` | ❌ 坏 | 同上，共用同一 bash |
| `pipeline_run` | ❌ 坏 | 同上 |
| `sandbox_pkg_install` | ❌ 坏 | 装完也起不来 |

**结论：一切命令走 `shell_exec`。这是默认动作，不是备选方案。**

## 禁止用 sed/awk/cut 处理大文件

Android 自带的 `sed`/`awk` 对**超长行（约 4 万字节以上）会静默截断**，不报错，结果全错。

已踩过的坑：主题 JSON 的 `custom_css` 字段是一整行 7 万字节，用 `sed -n '33p'` 读出来被截断，导致后续切割全错、生成坏文件。

替代做法：

- 用 `head -c` / `tail -c` 做**字节级**切片
- 用 `grep -bo` 取字节偏移定位
- 用 `od -An -tx1` 检查控制字符
- 校验 JSON：先剥字符串只留结构骨架，再查 `,,` `,]` `,}` `:,` 等非法模式

## 文件系统边界

| 路径 | 可访问 | 说明 |
|---|---|---|
| `/storage/emulated/0/Android/data/ai.devcodex.agent/files/workspace` | ✅ 读写 | 工作区，fs_* 工具默认位置 |
| `/storage/emulated/0/` 其他目录 | ✅ 读写 | 如 /sdcard/酒馆美化 |
| `/data/user/0/ai.devcodex.agent` | ❌ | 应用私有目录，无 root 进不去 |
| `/data/local/tmp` | ✅ 读写 | 临时文件、解包 APK |

写入用户文件时保留原属主和权限：用 `cat src > dst` 重定向，不要 `cp`（会覆盖权限）。

## 联网

`curl` 可用（`/system/bin/curl`）。抓 GitHub 用 `codeload.github.com/<owner>/<repo>/tar.gz/refs/heads/main`。

## 交付前必须自检

1. 改动只落在目标文件，原文件备份或未动
2. JSON/配置改动要过结构校验，不能只凭"看起来对"
3. 长行文件禁止用行工具断言内容
4. 明确说出哪些没验证

## 什么时候不用这份规则

沙盒若被修复（`libreadline.so.8` 补齐、`python_run` 能跑通），可恢复用 Python 处理 JSON。修好后请用户更新本技能描述。
