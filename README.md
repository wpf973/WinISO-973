# WinISO-973

Windows 10 LTSC 2021 IoT x64 镜像：**原始母盘存档 + 云端自动集成当月更新 + 双格式发布**。

## 两个 Release，别搞混

| Release tag | 用途 | 说明 |
|---|---|---|
| `Win10-2021-LTSC-IoT-x64` | **原始母盘素材库（勿删）** | 未打补丁的官方 ISO，仅改名后分卷存档；工作流每月从这里取母盘 |
| `Win10-2021-LTSC-IoT-x64-<年月>` | **每月成品** | Actions 集成当月更新后生成，**同一个 Release 内含 WIM / ESD 两种版本** |

## 选哪个版本下载

每月成品 Release 里同时提供两组附件，**按需下载其中一种即可，不要混用分卷**：

| 版本 | 文件名特征 | 特点 |
|---|---|---|
| **WIM（未压缩）** | `...-WIM.7z.001` … + `SHA256SUMS-WIM.txt` | 内含 `install.wim`；体积较大、先发布，适合后期二次修改、挂载提取 |
| **ESD（压缩）** | `...-ESD.7z.001` … + `SHA256SUMS-ESD.txt` | 内含 `install.esd`；体积约小 25%，由同一份 WIM 转换而来，适合分发、刻盘、部署 |

WIM 版先发布，随后把同一份 `install.wim` 转成 `install.esd` 并重建可启动 ISO，ESD 版自动追加到同一个 Release。**只集成一次补丁**，两个版本内容一致，区别仅在安装映像的压缩格式。

## 下载与还原

1. 打开最新的成品 Release，选定一个版本，把该版本的**全部** `.7z.001` / `.002` / `.003`… 和对应的 `SHA256SUMS-WIM.txt` 或 `SHA256SUMS-ESD.txt` 下载到**同一目录**
2. 校验：`Get-FileHash *.7z.* -Algorithm SHA256` 与清单比对
3. 安装 [7-Zip](https://www.7-zip.org/)，右键 `.001` → 解压，得到完整 ISO（命令行：`7z x *-WIM.7z.001`）

分卷必须齐全且不要改名；每个分卷 1950M，低于 GitHub 单附件 2 GiB 上限。

## 云端自动打补丁与发布

工作流：[`.github/workflows/patch-and-release.yml`](.github/workflows/patch-and-release.yml)

- **定时**：每月 15 日 03:17 UTC（北京 11:17），晚于微软补丁日
- **手动**：Actions → 「云端自动打补丁并发布」→ Run workflow，可填 `build_version`、`source_tag`
- **串行、只集成一次**：单个 job，`wim2esd=0` 集成一次生成 WIM 版 → 打包并**先发布 WIM** → 就地把 `install.wim` 用 `wimlib export --compress=LZMS --solid` 转成 `install.esd`（失败回退 DISM `/Compress:Recovery`）→ 用上游自带 `oscdimg` 重建可启动 ISO → 打包并追加发布 ESD 版到同一个 Release
- **流程**：下载母盘 3 个分卷 → 克隆 `adavak/Win_ISO_Patching_Scripts` → 复制 `config/W10UI.ini` 并临时设 `wim2esd=0` → `Start.cmd` 集成 → 7z 解出 ISO 内容目录 → WIM 分卷 + SHA256 + 上传 → WIM 转 ESD → oscdimg 重建 ISO → ESD 分卷 + SHA256 + 上传
- **配置**：仓库里的 `config/W10UI.ini` 严格照抄上游，仅三项定制：`nosuggtip=1`、`norestorage=1`、`nogamebar=1`；`AutoStart=1` 来自上游。`wim2esd=0` 只在本次运行时临时设置，**不改仓库配置文件**；ESD 版不靠改配置重跑集成，而是直接转换 WIM 结果
- **容错**：即使 ESD 转换或重建失败，WIM 版此时已经发布，客户不受影响
- **CI 适配**：上游脚本末尾有 `choice` 等待按键退出，无人值守会卡死；工作流在启动时用字节级替换处理这两处，不改集成逻辑
- **要求**：仓库 Actions 权限为 `Read and write`；`windows-2022` 实测有 D: 147 GB，磁盘充足

> 公开仓库 60 天无任何提交会自动停用定时任务，届时去 Actions 页点 **Enable workflow** 恢复；定时只在默认分支 `main` 生效。公开仓库标准 runner 免费且无分钟上限，但单次 job 上限 6 小时。

## 说明

本仓库仅用于镜像分发，母盘来自获得相应授权的官方渠道。公开分发 Windows 安装镜像受微软许可约束，请确认使用范围。
