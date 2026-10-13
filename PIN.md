# abing7k 的 cfsm-agent 安装脚本（固定版本）

这个仓库是 [huilang-me/cfsm-agent](https://github.com/huilang-me/cfsm-agent) 的 fork，用来托管自用的安装脚本。

分支说明：

- `main`：跟随上游，只用于同步，不在这里做改动。
- `my`：自用分支。跟上游相比只有一处改动 —— `install.sh` / `install.ps1` 默认安装**固定版本**，当前为 `v1.0.19`。

## 为什么固定版本

面板（Worker）是自建修改版。上游 agent 随时可能发新版，官方安装脚本默认拉 `latest`，意味着新装机器、重装机器会直接换成一个我们没验证过的 agent，可能出现上报格式不匹配。固定版本后，脚本拉到的永远是我们验证过的那一版。

当前线上各机器的 agent 版本：`v1.0.18` / `v1.0.19`，两个都和自建 Worker 验证过。

## 安装命令

Linux / OpenWrt / FreeBSD / macOS：

```bash
curl -fsSL 'https://raw.githubusercontent.com/abing7k/cfsm-agent/my/install.sh' | sh -s -- install \
  -id=SERVER_ID -secret=SECRET -url=https://monitor.0378999.xyz/update \
  -collect_interval=0 -interval=60 -connection_mode=auto -ping_mode=tcp \
  -reset_day=1 -auto_update=0 \
  -ct='gd-ct-dualstack.ip.zstaticcdn.com' \
  -cu='gd-cu-dualstack.ip.zstaticcdn.com' \
  -cm='gd-cm-dualstack.ip.zstaticcdn.com'
```

Windows（管理员 PowerShell）：

```powershell
$script = "$env:TEMP\install-cf-probe.ps1"
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/abing7k/cfsm-agent/my/install.ps1" -OutFile $script -UseBasicParsing
PowerShell -ExecutionPolicy Bypass -File $script install -id=SERVER_ID -secret=SECRET -url=https://monitor.0378999.xyz/update
```

临时指定其它版本（不改脚本，只影响这一次安装）：

```bash
curl -fsSL 'https://raw.githubusercontent.com/abing7k/cfsm-agent/my/install.sh' | sh -s -- install --install-version=v1.0.18 -id=... -secret=... -url=...
```

## 升级 agent 的流程

1. 先在一台测试机上装新版本：命令里临时加 `--install-version=vX.Y.Z`，确认面板能正常出数据；
2. 确认没问题后，改 `install.sh` 里的 `AGENT_VERSION` 和 `install.ps1` 里的 `$InstallVersion`，提交到 `my` 分支；
3. 之后所有新装/重装机器都会用这个版本。

## 注意事项

- **保持 `-auto_update=0`**。开着的话 agent 会自己去上游 GitHub release 拉新版，绕开这里的固定版本。自动更新只由本机配置控制，面板不会强制打开。
- 二进制仍然从上游 release 下载：`https://github.com/huilang-me/cfsm-agent/releases/download/<固定tag>/cf-probe-<os>-<arch>`。tag 一旦发布就不会变，所以内容是可复现的。如果以后需要连二进制也搬到自己的仓库，可以再做镜像。
- 同步上游时只影响 `main` 分支，`my` 分支上的改动不会被覆盖。
