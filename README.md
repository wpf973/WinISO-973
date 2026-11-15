# WinISO-973

Windows 10 LTSC 2021 IoT x64 镜像：**原始母盘存档 + 云端自动集成当月更新**。

## 两个 Release，别搞混

| Release tag | 用途 | 说明 |
|---|---|---|
| `Win10-2021-LTSC-IoT-x64` | **原始母盘素材库（勿删）** | 未打补丁的官方 ISO，仅改名后分卷存档；工作流每月从这里取母盘 |
| `Win10-2021-LTSC-IoT-x64-<年月>` | **每月成品** | Actions 自动集成当月更新后生成并发布的新 ISO |

## 云端自动打补丁与发布

工作流：[`.github/workflows/patch-and-release.yml`](.github/workflows/patch-and-release.yml)

- **定时**：每月 15 日 03:17 UTC（北京 11:17），晚于微软补丁日
- **手动**：Actions → 「云端自动打补丁并发布」→ Run workflow，可填 `build_version`、`source_tag`、`wim2esd`
- **流程**：下载母盘 3 个分卷 → 克隆 `adavak/Win_ISO_Patching_Scripts` → 写入 `config/W10UI.ini` → `Start.cmd` 集成当月更新 → 分卷打包 → 自动创建新 Release
- **关键配置**：`config/W10UI.ini` 严格照抄上游 `adavak/Win_ISO_Patching_Scripts` 的 `W10UI.ini`，仅三项定制：`nosuggtip=1`、`norestorage=1`、`nogamebar=1`；`AutoStart=1` 来自上游，保证无人值守。工作流只做原样复制，不改写任何配置
- **要求**：仓库 Actions 权限须为 `Read and write`；标准 runner 只有 14 GB 磁盘，若报磁盘不足需改用带大容量 D: 盘的镜像、larger runner 或自托管 runner

> 公开仓库 60 天无任何提交会自动停用定时任务，届时去 Actions 页点 **Enable workflow** 恢复；定时只在默认分支 `main` 生效。

## 下载与还原

打开最新的成品 Release，下载**全部** `.7z.001` / `.002` / `.003` 和 `SHA256SUMS.txt` 到同一目录。

1. 校验：`Get-FileHash *.7z.* -Algorithm SHA256` 与清单比对
2. 安装 [7-Zip](https://www.7-zip.org/)，右键 `.001` → 解压，得到完整 ISO（命令行：`7z x *.7z.001`）

分卷必须齐全且不要改名；每个分卷 1950M，低于 GitHub 单附件 2 GiB 上限。

## 说明

本仓库仅用于镜像分发，母盘来自获得相应授权的官方渠道。公开分发 Windows 安装镜像受微软许可约束，请确认使用范围。
