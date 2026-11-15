# WinISO-973
自动打包win10-2021-LTSC-IOT.iso

## 下载镜像

最新版本：[Win10-2021-LTSC-IoT-x64](https://github.com/wpf973/WinISO-973/releases/tag/Win10-2021-LTSC-IoT-x64)

把 `.7z.001` / `.002` / `.003` 和 `SHA256SUMS.txt` 全部下载到同一目录，用 7-Zip 打开 `.001` 解压，得到 `win10-2021-LTSC-IOT.iso`。分卷必须齐全，不要改名。

校验：`Get-FileHash *.7z.* -Algorithm SHA256` 与 `SHA256SUMS.txt` 比对。
