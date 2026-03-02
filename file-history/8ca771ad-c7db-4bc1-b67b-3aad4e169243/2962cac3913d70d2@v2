# 工作流程配置

## 标准操作流程

### 打包完成后
```yaml
trigger: package_created
action: open_folder
command: start .
description: 完成打包后立即打开安装包文件夹
```

### 依赖安装
```yaml
trigger: install_dependencies
action: use_mirror
mirror: https://pypi.tuna.tsinghua.edu.cn/simple
description: 使用清华镜像源加速下载
```

### 配置更新
```yaml
trigger: config_update
action: update_both
targets:
  - global: ~/.claude/
  - local: ./
description: 同时更新全局和本地配置
```

## 自动化规则

1. **打包流程**
   - 创建安装包
   - 验证文件完整性
   - 打开文件夹
   - 显示使用说明

2. **配置管理**
   - 全局配置: 跨项目通用设置
   - 本地配置: 项目特定设置
   - 保持同步: 关键设置同时更新

3. **用户交互**
   - 不询问确认
   - 直接执行操作
   - 显示执行结果
