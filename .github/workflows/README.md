# GitHub Actions 工作流程说明

[![Build and Release](https://github.com/pentaho/pentaho-kettle/actions/workflows/release.yml/badge.svg)](https://github.com/pentaho/pentaho-kettle/actions/workflows/release.yml)
[![Build Test](https://github.com/pentaho/pentaho-kettle/actions/workflows/build-test.yml/badge.svg)](https://github.com/pentaho/pentaho-kettle/actions/workflows/build-test.yml)
[![Quick Build](https://github.com/pentaho/pentaho-kettle/actions/workflows/quick-build.yml/badge.svg)](https://github.com/pentaho/pentaho-kettle/actions/workflows/quick-build.yml)

本目录包含了Pentaho Kettle项目的GitHub Actions工作流程配置。

## 工作流程概览

### 1. `release.yml` - 构建和发布工作流程

**功能**: 执行完整的Maven构建并自动创建GitHub Release

**触发条件**:
- 手动触发 (workflow_dispatch)
- Git标签推送 (push tags: v*)

**主要步骤**:
1. 设置Java 11环境
2. 执行 `mvn clean package` 构建
3. 收集所有assembly生成的分发包
4. 创建GitHub Release并上传分发包

**生成的分发包**:
- `pdi-ce-*.zip` - Desktop Client Community Edition
- `pdi-static-*.zip` - 静态资源和配置
- `pdi-libs-*.zip` - 库依赖项
- `pdi-plugins-*.zip` - 插件模块
- `pdi-samples-*.zip` - 示例转换和作业

### 2. `build-test.yml` - 测试构建工作流程

**功能**: 执行构建测试，不创建Release

**触发条件**:
- 手动触发 (workflow_dispatch)
- Pull Request到主分支

**主要步骤**:
1. 设置Java 11环境
2. 执行Maven构建
3. 验证assembly输出
4. 上传构建产物（仅用于测试）

### 3. `quick-build.yml` - 快速构建工作流程

**功能**: 快速编译检查和核心模块测试

**触发条件**:
- 推送到主分支
- 手动触发 (workflow_dispatch)

**主要步骤**:
1. 设置Java 11环境
2. 快速编译检查
3. 运行核心模块单元测试
4. 提供快速反馈

## 使用说明

### 手动触发Release

1. 进入GitHub仓库的 **Actions** 页面
2. 选择 **"Build and Release Pentaho Kettle"** 工作流程
3. 点击 **"Run workflow"**
4. 填写以下参数：
   - **Release tag**: 发布标签 (例如: v11.0.0.0)
   - **Release name**: 发布名称
   - **Release description**: 发布描述
   - **Mark as pre-release**: 是否标记为预发布

### 通过Git标签触发Release

```bash
# 创建并推送标签
git tag v11.0.0.0
git push origin v11.0.0.0
```

### 测试构建

1. 进入GitHub仓库的 **Actions** 页面
2. 选择 **"Build Test (No Release)"** 工作流程
3. 点击 **"Run workflow"**
4. 选择构建配置：
   - **Maven build profile**: default 或 osgi
   - **Skip tests**: 是否跳过测试

## 构建环境要求

- **Java版本**: OpenJDK 11 (Temurin发行版)
- **Maven**: 使用GitHub Actions预装版本
- **内存配置**: `-Xmx4g -Xms1g`
- **超时时间**: 
  - Release构建: 120分钟
  - 测试构建: 90分钟

## 构建优化

工作流程包含以下优化措施：

1. **Maven依赖缓存**: 使用GitHub Actions缓存加速构建
2. **跳过非必要步骤**: 
   - 跳过JavaDoc生成
   - 跳过源码打包
   - 跳过代码质量检查（构建时）
3. **并行构建**: Maven使用多线程构建
4. **失败处理**: 构建失败时自动上传日志

## 故障排除

### 构建失败

1. 检查 **Actions** 页面的构建日志
2. 如果是依赖问题，可能需要清理Maven缓存
3. 查看上传的构建失败日志artifact

### 分发包缺失

1. 确认所有assembly模块都成功构建
2. 检查 `assemblies/*/target/` 目录下是否有zip文件
3. 验证Maven assembly插件配置

### Release创建失败

1. 确认GitHub token权限
2. 检查标签名称格式
3. 验证分发包文件是否存在

## 自定义配置

### 修改Java版本

在工作流程文件中修改 `JAVA_VERSION` 环境变量：

```yaml
env:
  JAVA_VERSION: '17'  # 改为Java 17
```

### 添加额外的构建步骤

在相应的工作流程文件中的 `steps` 部分添加新步骤。

### 修改Maven参数

在 `Run Maven build` 步骤中修改Maven命令参数。

## 安全注意事项

1. 工作流程使用 `GITHUB_TOKEN` 进行认证
2. 不要在工作流程中硬编码敏感信息
3. 定期检查依赖项的安全漏洞
4. 限制工作流程的执行权限

## 监控和维护

1. 定期检查工作流程执行状态
2. 监控构建时间趋势
3. 及时更新依赖项版本
4. 清理过期的构建artifact
