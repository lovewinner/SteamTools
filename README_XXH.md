# Watt Toolkit (Steam++) - Linux Headless 版本

精简版：仅保留 Linux 版本，移除 UI 界面，默认开启网络加速功能并启用所有加速站点。

## 快速开始

### 编译

```bash
dotnet build src/BD.WTTS.Client.Avalonia.App -c Release
```

### 运行无UI代理服务

```bash
dotnet run --project src/BD.WTTS.Client.Avalonia.App -- -clt proxy-headless
```

## 命令行参数

| 命令 | 说明 |
|------|------|
| `-clt proxy-headless` | 无UI代理模式，自动启用所有加速站点（推荐） |
| `-clt proxy -on` | 开启代理（需主进程已运行） |
| `-clt proxy -off` | 关闭代理 |
| `-clt shutdown` | 安全结束正在运行的程序 |
| `-clt show -cert` | 显示根证书信息 |
| `-clt linux -ceri <路径>` | 安装证书（需root权限） |
| `-clt linux -cerd <路径>` | 删除证书（需root权限） |

## 证书管理

Linux 上安装/删除证书需要 root 权限：

```bash
# 安装证书
sudo dotnet BD.WTTS.Client.Avalonia.App.dll -clt linux -ceri <AppDataDirectory>

# 删除证书
sudo dotnet BD.WTTS.Client.Avalonia.App.dll -clt linux -cerd <AppDataDirectory>
```

## 架构说明

```
主进程 (BD.WTTS.Client.Avalonia.App)
├── 加速插件 (BD.WTTS.Client.Plugins.Accelerator)
│   ├── 代理服务管理
│   ├── 加速站点配置（从服务端动态获取）
│   └── 脚本管理
└── 反向代理子进程 (BD.WTTS.Client.Plugins.Accelerator.ReverseProxy)
    ├── YARP 反向代理引擎
    ├── 证书管理
    └── DNS 解析
```

- 反向代理子进程通过 IPC 与主进程通信，**无需手动启动**
- 加速站点数据从服务端动态获取，首次加载时默认全选所有站点
- 代理服务默认在程序启动时自动开启

## 配置说明

### 默认配置

- **代理端口**: 26561
- **DNS**: 223.5.5.5
- **代理模式**: Hosts
- **自动启动代理**: 是
- **DNS over HTTPS**: 启用
- **启用所有加速站点**: 是（首次加载时）

### 配置文件位置

Linux 遵循 XDG 规范：

- **数据目录**: `$XDG_DATA_HOME/Steam++`
- **缓存目录**: `$XDG_CACHE_HOME/Steam++`

## 依赖项

- .NET 11.0 Runtime
- ASP.NET Core Runtime 11.0（反向代理子进程需要）
