# 项目分析指南

## 通用分析步骤

### 1. 识别技术栈

按优先级检查以下文件：

| 标志文件 | 技术栈 |
|----------|--------|
| `pom.xml` | Java + Maven |
| `build.gradle` / `build.gradle.kts` | Java/Kotlin + Gradle |
| `package.json` | Node.js/TypeScript |
| `requirements.txt` / `pyproject.toml` / `setup.py` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `Gemfile` | Ruby |
| `composer.json` | PHP |
| `*.sln` / `*.csproj` | C#/.NET |

```
Glob "**/pom.xml"
Glob "**/package.json"
Glob "**/requirements.txt"
Glob "**/go.mod"
Glob "**/Cargo.toml"
```

读取找到的构建文件，提取：
- 语言版本
- 框架和版本（Spring Boot、Express、Django 等）
- 关键依赖列表

### 2. 分析目录结构

```bash
# 获取顶层目录
ls -la {project-root}

# 获取 src 目录结构（限制深度）
find {project-root}/src -type d -maxdepth 4 2>/dev/null | head -50
```

### 3. 识别核心文件

按文件角色分类查找：

#### API 层
```
Glob "**/controller/**/*.java"
Glob "**/controllers/**/*.{ts,js}"
Glob "**/routes/**/*.{ts,js,py}"
Glob "**/views/**/*.py"
Glob "**/*Provider.java"
Glob "**/*Handler.{go,java}"
```

#### 业务层
```
Glob "**/service/**/*.java"
Glob "**/services/**/*.{ts,js,py}"
Glob "**/usecases/**/*"
Glob "**/domain/**/*"
```

#### 数据层
```
Glob "**/entity/**/*.java"
Glob "**/model/**/*.{java,ts,py}"
Glob "**/models/**/*.{ts,js,py}"
Glob "**/repository/**/*.java"
Glob "**/mapper/**/*.{java,xml}"
Glob "**/dao/**/*.java"
Glob "**/schema/**/*"
```

#### 配置
```
Glob "**/config/**/*"
Glob "**/configuration/**/*"
Glob "**/*.yml"
Glob "**/*.yaml"
Glob "**/*.toml"
Glob "**/.env.example"
```

### 4. 分析 API

对每个 API 入口文件执行 Read，提取：
- 路由/端点路径
- HTTP 方法
- 请求参数（路径参数、查询参数、请求体）
- 响应结构
- 认证要求
- 调用的 Service 方法

### 5. 分析数据模型

读取 Entity/Model 文件，提取：
- 表名/集合名
- 字段名、类型、约束
- 关联关系
- 索引定义

对 Java 项目，同时检查 Mapper XML 确认字段映射。

**数据来源优先级**: Entity 类 > Mapper XML > DDL 文件

### 6. 提取枚举和常量

```
Glob "**/enum/**/*.java"
Glob "**/enums/**/*.java"
Glob "**/constants/**/*.java"
Glob "**/types/**/*.ts"
Grep "export enum" --glob "*.ts"
Grep "class.*Enum" --glob "*.py"
```

### 7. 分析构建和开发流程

读取以下文件获取开发信息：
- `Makefile` / `Taskfile.yml`
- `docker-compose.yml`
- `.github/workflows/*.yml`
- `scripts/` 目录
- `README.md`（开发相关章节）

---

## 技术栈特定模式

### Java + Spring Boot

```
# 启动类
Grep "@SpringBootApplication" --glob "*.java"

# REST 控制器
Grep "@RestController" --glob "*.java"
Grep "@RequestMapping" --glob "*.java"

# 服务
Grep "@Service" --glob "*.java"

# 数据访问
Grep "@Repository" --glob "*.java"
Grep "@Entity" --glob "*.java"
Grep "@Table" --glob "*.java"

# 配置
Glob "**/application*.yml"
Glob "**/application*.properties"
```

### Node.js / TypeScript

```
Glob "**/index.ts"                              # 入口
Glob "**/app.ts"
Grep "router\." --glob "*.ts"                   # 路由
Grep "app\.(get|post|put|delete)" --glob "*.ts"
Grep "Schema\(" --glob "*.ts"                   # ORM
Grep "@Entity" --glob "*.ts"
```

### Python (Django/FastAPI/Flask)

```
Glob "**/urls.py"                               # Django 路由
Glob "**/views.py"                              # Django 视图
Glob "**/models.py"                             # 模型
Grep "@app\.(get|post|put|delete)" --glob "*.py" # FastAPI
Grep "@app\.route" --glob "*.py"                # Flask
```

### Go

```
Glob "**/main.go"                               # 入口
Grep "func.*Handler" --glob "*.go"              # 路由
Grep "type.*struct" --glob "*.go"               # 模型
```
