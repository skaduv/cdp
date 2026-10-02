# Claude Desktop Patch

用于 Windows Claude Desktop 的本地 PowerShell 补丁脚本。生成可注册的开发版应用目录，无需重新打包或发布 MSIX。

脚本在独立的开发版副本中按代码结构修改 ASAR 和前端 JavaScript，实现模型、工具与遥测相关调整，并同步更新 ASAR 完整性哈希及 EXE 内嵌哈希，修改前备份原始文件以支持恢复。

## 功能

- **模型名称**：Gateway/Mantle 接受非空且不含控制字符的模型 ID，保留原始 ID。
- **模型发现**：默认启用自动发现，移除发现结果中的厂商名称过滤，并与手填模型清单合并。
- **简体中文**：新增原生界面、前端及动态文案语言资源，可在语言设置中选择。
- **Computer Use / Browser Use**：开启相关本地特性，保留组织策略、系统权限和服务可用性检查。
- **工具搜索**：默认启用 `toolSearchEnabled`，扩展到 1p 和 3p 会话。
- **WebFetch**：默认启用 `skipWebFetchPreflight`，跳过向 Anthropic 查询域名的预检，保留网络白名单和工具权限。
- **遥测**：默认启用 `disableEssentialTelemetry` 和 `disableNonessentialTelemetry`，在 1p 模式中强制关闭对应遥测，并设置 Code 子进程禁用遥测、错误报告的环境变量。
- **备份与恢复**：修改前自动备份，恢复前后校验 SHA256，并移除新增语言文件。

自动发现、工具搜索和 WebFetch 预检设置允许显式 `false` 关闭；1p 遥测使用强制覆盖。

## 环境要求

- Windows，x64 Claude Desktop。
- Python 3.10 或更高版本。
- Node.js，可通过 `PATH` 找到或使用 `-NodePath` 指定。
- 激活修改版时，需要开启 Windows 开发者模式。


## 快速开始

### 1. 开启开发者模式

按 **Win + I** 打开 Windows 设置，搜索“开发者模式”，开启并确认提示。

- 新版 Windows 11：**系统 → 高级 → 开发者选项**。
- 较旧 Windows 11：**隐私和安全性 → 开发者选项**。
- Windows 10：**更新和安全 → 开发者选项**。

无需开启设备门户或设备发现。参见 [Microsoft 开发者模式文档](https://learn.microsoft.com/en-us/windows/advanced-settings/developer-mode)。

### 2. 检查兼容性

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "\路径\Patch-ClaudeDesktop.ps1" -CheckOnly
```

在线获取语言资源并检查应用结构及补丁结果，不修改程序资源或应用注册。

### 3. 应用并激活

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "\路径\Patch-ClaudeDesktop.ps1" -Activate
```

脚本准备修改副本并完成校验后，询问是否替换当前用户的 Claude 应用注册。输入 `y` 后继续。

不加 `-Activate` 只准备副本。默认目录为 `\路径\ClaudeDesktop-Patched`；如检测到先前的旧命名目录，脚本会继续使用该目录及其原始备份。

激活后，在 **Settings → Language** 选择“简体中文”。重启 Claude 并新建会话，使会话设置生效。

### 4. 更新已有副本

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "\路径\Patch-ClaudeDesktop.ps1" -PatchOnly -Activate
```

自定义过安装目录时，添加相同的 `-InstallDir`。

## 备份与恢复

资源备份位于安装目录的 `patch-backup`。重复应用补丁保留首次原始基线，不会用修改后的文件替换原始备份。

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "\路径\Patch-ClaudeDesktop.ps1" -Restore
```

`-RestoreOriginal` 是 `-Restore` 的同义参数。恢复无需 Node.js 或语言包，仍需要 Python。

恢复会还原脚本修改的 EXE、ASAR 和前端文件，并删除新增语言文件；不会回退账号数据或恢复官方签名安装的应用注册方式。回到官方安装方式需要重新安装官方包。

激活前另行保存原应用布局与用户数据快照，位于 `registration-backup`。快照可能包含登录信息，应仅在本机保管。

## 参数

| 参数 | 说明 |
| --- | --- |
| `-CheckOnly` | 检查兼容性与补丁结果 |
| `-Activate` | 注册修改版并启动 |
| `-PatchOnly` | 使用已有副本，不重新复制安装布局 |
| `-Restore` | 恢复原始程序资源 |
| `-InstallDir <路径>` | 指定修改副本目录 |
| `-MsixPath <路径>` | 从原版 MSIX 提取应用 |
| `-LanguageDir <路径>` | 可选：指定含上游 desktop/frontend/statsig-zh-CN.json 的 resources 目录；默认在线下载 |
| `-PythonPath <路径>` | 指定 Python 可执行文件 |
| `-NodePath <路径>` | 指定 Node.js 可执行文件 |
| `-NoLaunch` | 激活后不启动应用 |
| `-Yes` | 跳过应用注册替换确认 |


## 已知限制

- 修改 EXE 会使原 Anthropic 数字签名失效；脚本会重新计算并保留 ASAR 完整性校验。
- 客户端仍使用 Anthropic Messages 协议；任意模型名称通过校验，不代表网关或模型支持全部工具协议。
- 工具搜索、Computer Use 和 Browser Use 的实际可用性取决于后端能力、组织策略及系统授权。
- 简体中文来自上游语言资源；新版新增且上游尚未翻译的条目保留英文。
- 遥测修改未经过完整联网抓包验收，独立 OTLP 导出不在覆盖范围内，不能保证所有遥测流量为零。

## 免责声明

本项目仅供学习与技术研究，使用者应遵守适用法律及相关服务条款，并自行承担使用、修改或分发本项目的风险与责任。
在适用法律允许的最大范围内，本项目不作任何明示或默示保证，作者及贡献者不对使用本项目产生的损失或责任承担任何责任。
