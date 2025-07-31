# Pentaho Kettle 构建故障排除指南

## 🚨 常见构建问题及解决方案

### 1. 依赖解析失败

#### 问题症状
```
[ERROR] Could not resolve dependencies for project org.pentaho.di.plugins:pdi-core-plugins-impl:jar:11.0.0.0-SNAPSHOT
[ERROR] Could not find artifact org.pentaho.reporting.libraries:libfonts:jar:11.0.0.0-SNAPSHOT
```

#### 解决方案
1. **检查依赖坐标**：
   ```bash
   # 验证正确的artifact坐标
   mvn dependency:tree -Dverbose | grep -i "libfonts\|libloader"
   ```

2. **清理本地仓库**：
   ```bash
   # 删除有问题的依赖缓存
   rm -rf ~/.m2/repository/org/pentaho/reporting/
   mvn dependency:resolve -U
   ```

3. **检查网络连接**：
   ```bash
   # 测试Maven仓库连接
   curl -I https://repo.orl.eng.hitachivantara.com/artifactory/pnt-mvn/
   ```

### 2. 编译错误

#### 问题症状
```
[ERROR] package org.pentaho.reporting.libraries.fonts does not exist
[ERROR] package org.pentaho.reporting.libraries.resourceloader does not exist
```

#### 解决方案
1. **检查POM依赖**：
   确保`plugins/core/impl/pom.xml`包含正确的依赖项：
   ```xml
   <dependency>
     <groupId>org.pentaho.reporting.library</groupId>
     <artifactId>libfonts</artifactId>
     <version>${pentaho-reporting.version}</version>
   </dependency>
   ```

2. **重新构建依赖模块**：
   ```bash
   mvn clean install -pl plugins/core/impl -am -DskipTests
   ```

### 3. 版本属性未定义

#### 问题症状
```
[ERROR] 'dependencies.dependency.version' for org.ehcache:ehcache:jar:jakarta is missing
```

#### 解决方案
1. **检查根POM属性**：
   确保`pom.xml`包含版本定义：
   ```xml
   <properties>
     <ehcache.version>3.10.8</ehcache.version>
   </properties>
   ```

2. **验证有效POM**：
   ```bash
   mvn help:effective-pom | grep -i ehcache
   ```

### 4. 内存不足错误

#### 问题症状
```
[ERROR] Java heap space
[ERROR] GC overhead limit exceeded
```

#### 解决方案
1. **增加Maven内存**：
   ```bash
   export MAVEN_OPTS="-Xmx4g -Xms1g -XX:MaxPermSize=512m"
   mvn clean package
   ```

2. **分模块构建**：
   ```bash
   # 分别构建各个模块
   mvn clean install -pl core -am
   mvn clean install -pl engine -am
   mvn clean install -pl plugins/core/impl -am
   ```

### 5. 网络超时问题

#### 问题症状
```
[ERROR] Transfer failed for https://repo.orl.eng.hitachivantara.com/...
[ERROR] Connect timed out
```

#### 解决方案
1. **配置重试机制**：
   ```bash
   mvn clean package \
     -Dmaven.wagon.http.retryHandler.count=3 \
     -Dmaven.wagon.httpconnectionManager.ttlSeconds=120
   ```

2. **使用离线模式**（如果依赖已缓存）：
   ```bash
   mvn clean package -o
   ```

## 🔧 诊断工具和命令

### 1. 依赖分析
```bash
# 查看依赖树
mvn dependency:tree -Dverbose

# 分析依赖冲突
mvn dependency:analyze

# 解析所有依赖
mvn dependency:resolve -U

# 查看有效POM
mvn help:effective-pom
```

### 2. 构建诊断
```bash
# 详细构建日志
mvn clean package -X

# 跳过测试的快速构建
mvn clean package -DskipTests -T 1C

# 仅编译不打包
mvn clean compile

# 验证项目结构
mvn validate
```

### 3. 缓存管理
```bash
# 清理本地仓库
mvn dependency:purge-local-repository

# 强制更新快照
mvn clean package -U

# 清理构建目录
mvn clean
```

## 🚀 性能优化建议

### 1. 并行构建
```bash
# 使用多线程构建
mvn clean package -T 1C

# 指定线程数
mvn clean package -T 4
```

### 2. 跳过非必要步骤
```bash
mvn clean package \
  -DskipTests \
  -Dmaven.javadoc.skip=true \
  -Dmaven.source.skip=true \
  -Dcheckstyle.skip=true \
  -Dpmd.skip=true \
  -Dspotbugs.skip=true
```

### 3. 增量构建
```bash
# 仅构建变更的模块
mvn clean package -pl plugins/core/impl -am

# 从失败的模块继续构建
mvn clean package -rf :pdi-core-plugins-impl
```

## 📋 环境检查清单

### 构建前检查
- [ ] Java版本：JDK 11或更高
- [ ] Maven版本：3.6.0或更高
- [ ] 网络连接：能访问Maven仓库
- [ ] 磁盘空间：至少5GB可用空间
- [ ] 内存：至少8GB RAM

### 依赖检查
- [ ] 根POM版本属性完整
- [ ] dependencyManagement配置正确
- [ ] 模块间依赖关系清晰
- [ ] 排除冲突依赖

### 构建环境
- [ ] MAVEN_OPTS设置合理
- [ ] 本地仓库路径正确
- [ ] 代理设置（如需要）
- [ ] 镜像仓库配置

## 🆘 获取帮助

### 1. 查看构建日志
```bash
# 生成详细日志文件
mvn clean package -X > build.log 2>&1

# 搜索错误信息
grep -i error build.log
grep -i "could not" build.log
```

### 2. 社区资源
- [Pentaho Community](https://community.hitachivantara.com/s/topic/0TO1J000000MFHPWA4/pentaho)
- [Maven官方文档](https://maven.apache.org/guides/)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/pentaho)

### 3. 报告问题
提交问题时请包含：
- 完整的错误信息
- Maven版本和Java版本
- 操作系统信息
- 构建命令和参数
- 相关的POM文件片段

## 📝 最佳实践

1. **定期清理**：定期清理本地Maven仓库
2. **版本管理**：使用版本属性统一管理依赖版本
3. **增量构建**：开发时使用增量构建提高效率
4. **日志保存**：保存构建日志便于问题排查
5. **环境隔离**：使用Docker或虚拟机隔离构建环境
