# OPS-GXL 操作管理器

## 概述

OPS-GXL 是一个基于 Galaxy Operator 框架的操作管理器，提供统一的模块化操作接口，支持系统级和模块级的生命周期管理。该操作器遵循 Sys-Operator 规范，实现了下载、安装、启动、停止、重启、更新、卸载、状态检查和诊断等标准化操作流程。

## 版本信息

当前版本：0.2.0

## 系统架构

### 核心组件

OPS-GXL 由以下核心组件构成：

- **mod_ops**: 模块操作管理器，提供单个模块的完整生命周期管理
- **sys_ops**: 系统操作管理器，基于 mod_list.yml 配置文件批量管理系统模块
- **ops_utls**: 操作工具集，提供通用的辅助功能，如文件下载和解压

### 目录结构

```
ops-gxl/
├── _gal/                    # Galaxy 配置目录
│   ├── adm.gxl             # 管理配置文件
│   └── project.toml        # 项目配置文件
├── mods/                   # 模块目录
│   ├── mod_ops.gxl         # 模块操作管理器
│   ├── ops_utls.gxl        # 操作工具集
│   └── sys_ops.gxl         # 系统操作管理器
├── .gitignore              # Git 忽略文件
├── readme.md               # 项目说明文档
└── version.txt             # 版本信息文件
```

## 功能特性

### 1. 模块生命周期管理

OPS-GXL 提供完整的模块生命周期管理功能：

- **download**: 下载模块资源
- **install**: 安装模块到目标环境
- **start**: 启动模块服务
- **stop**: 停止模块服务
- **restart**: 重启模块服务
- **update**: 更新模块版本
- **uninstall**: 卸载模块
- **status**: 检查模块运行状态
- **diagnose**: 诊断模块问题

### 2. 系统级批量操作

基于 `mod_list.yml` 配置文件，支持对多个模块进行批量操作：

- 自动遍历配置文件中的模块列表
- 支持模块启用/禁用控制
- 统一的操作接口和错误处理
- 并行化操作执行

### 3. 工具集功能

提供通用的辅助功能：

- **download_tar**: 支持从缓存或原始地址下载并解压 tar 文件
- 自动创建下载目录
- 支持缓存机制优化下载性能

## 配置说明

### 依赖配置

在 `_gal/adm.gxl` 中配置外部依赖：

```gxl
extern mod cfm { git = "https://github.com/galaxy-operators/cfm-gxl.git", channel = "${GXL_CHANNEL:main}" }
```

### 环境变量

OPS-GXL 使用以下环境变量：

- `ENV_MODEL`: 目标环境模型（如：arm-mac14-host）
- `ENV_MODULE_ENV`: 模块执行环境
- `ENV_SYS`: 系统配置目录
- `GXL_CHANNEL`: Galaxy 通道配置

## 使用方法

### 1. 单模块操作

使用 `mod_ops` 模块进行单个模块的操作：

```bash
# 下载模块
gx.run(local: "./mod/${ENV_MODEL}", env: "${ENV_MODULE_ENV}", flow: "download")

# 安装模块
gx.run(local: "./mod/${ENV_MODEL}", env: "${ENV_MODULE_ENV}", flow: "install")

# 启动模块
gx.run(local: "./mod/${ENV_MODEL}", env: "${ENV_MODULE_ENV}", flow: "start")
```

### 2. 系统批量操作

使用 `sys_ops` 模块进行系统级批量操作：

```bash
# 批量下载所有启用模块
gx.run(local: "./${ENV_SYS}/mods/${CUR.NAME}/${ENV_SYS_MODEL}", env: "${ENV_MODULE_ENV}", flow: "download")

# 批量安装所有启用模块
gx.run(local: "./${ENV_SYS}/mods/${CUR.NAME}/${ENV_SYS_MODEL}", env: "${ENV_MODULE_ENV}", flow: "install")
```

### 3. 工具集使用

使用 `ops_utls` 模块的辅助功能：

```bash
# 下载并解压 tar 文件
download_tar(artifacts, art_local)
```

## 配置文件规范

### mod_list.yml 格式

系统操作依赖于 `mod_list.yml` 配置文件，格式如下：

```yaml
- name: "module_name"
  enable: true
  # 其他模块配置...
```

### 模块配置

每个模块需要支持以下标准操作流程：

- download
- install
- start
- stop
- restart
- update
- uninstall
- status
- diagnose

## 最佳实践

### 1. 模块开发

- 遵循 Sys-Operator 命名规范
- 实现所有标准操作接口
- 提供详细的状态信息和错误处理
- 支持环境变量配置

### 2. 系统配置

- 使用 `mod_list.yml` 统一管理模块
- 合理设置模块启用状态
- 配置适当的环境变量
- 实现依赖检查机制

### 3. 操作执行

- 先执行 download 操作获取资源
- 按顺序执行 install、start 操作
- 定期执行 status 和 diagnose 进行健康检查
- 使用 update 进行版本升级

## 故障排除

### 常见问题

1. **模块下载失败**
   - 检查网络连接
   - 验证 `ENV_MODEL` 环境变量
   - 确认模块路径正确

2. **操作执行失败**
   - 检查 `mod_list.yml` 配置
   - 验证模块启用状态
   - 查看详细错误日志

3. **环境变量问题**
   - 确认所有必需环境变量已设置
   - 检查变量值格式正确
   - 验证路径存在性

### 调试方法

1. 使用 `diagnose` 流程进行问题诊断
2. 检查 `status` 流程的输出信息
3. 查看 `gx.echo` 输出的调试信息
4. 验证配置文件语法正确性

## 依赖关系

- **cfm-gxl**: 配置管理模块
- **Galaxy Operator 框架**: 基础运行环境
- **Sys-Operator 规范**: 系统操作标准

## 贡献指南

1. 遵循 Sys-Operator 命名规范
2. 实现完整的模块生命周期接口
3. 提供详细的文档和示例
4. 确保代码质量和测试覆盖

## 许可证

请参考项目根目录的许可证文件。

## 联系方式

如有问题或建议，请通过以下方式联系：

- 项目仓库：[Galaxy Operators](https://github.com/galaxy-operators)
- 问题反馈：GitHub Issues

---

*本文档遵循 Sys-Operator 规范编写，版本 0.2.0*